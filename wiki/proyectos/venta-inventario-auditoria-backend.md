---
titulo: Venta e Inventario — Auditoría de arquitectura del backend
tipo: proyecto
tags: [inventario, nestjs, sequelize, auditoria, solid, clean-code, seguridad, observabilidad, deuda-tecnica]
fecha_creacion: 2026-07-08
fecha_actualizacion: 2026-07-09
estado: activo
---

# Venta e Inventario — Auditoría de arquitectura del backend

Auditoría de solo lectura del backend NestJS. Documenta deuda técnica, violaciones SOLID, clean code, cobertura Swagger, estándar de auditoría de columnas, seguridad y observabilidad. **No cambia código** — es un entregable de diagnóstico para revisar y corregir por fases.

Hub: [[venta-inventario]] · Relacionadas: [[venta-inventario-rbac]] · [[venta-inventario-multitenancy]] · [[venta-inventario-testing]]

## Dónde vive

- Repo: submódulo `backend/` (repo `inventario-multisede-backend`).
- Resumen priorizado: `backend/PLAN-MEJORAS-BACKEND.md`.
- Detalle punto por punto: `backend/docs/auditoria/` — índice maestro en `docs/auditoria/README.md` + una **ficha por hallazgo** (`fase-N/FN-XX-*.md`).
- Cada ficha: qué está mal (snippet real) · por qué · en qué me baso (principio/estándar) · documentación oficial + enlaces · cómo se corregiría · checklist de revisión.

## Método de trabajo (preferencia de Christian)

- Detallar **por partes, fase por fase** — no todo de golpe. Christian revisa punto por punto y marca estado en el índice (📄 listo / ✅ aprobado / ❌ descartado / 🔧 corregido).
- **Un commit por fase** en el submódulo backend (rama `main`). No se hizo push; el puntero del submódulo en el repo padre quedó sin commitear.
- Regla: limpieza (Fases 1–5) no cambia lógica; Fases 6–7 sí → requieren aprobación explícita antes de ejecutar.

## Fases (8, total 64 fichas)

| Fase | Tema | Fichas | Naturaleza |
|------|------|--------|-----------|
| 1 | Config / tooling | 9 | limpieza |
| 2 | SOLID (god services, storage, guardia) | 8 | refactor forma |
| 3 | Duplicación / DRY | 6 | refactor forma |
| 4 | Clean code | 10 | limpieza |
| 5 | Swagger | 6 | doc |
| 6 | Estándar de auditoría de columnas | 4 | ⚠️ cambia datos |
| 7 | Seguridad | 13 | 🔴 cambia lógica |
| 8 | Observabilidad | 8 | falta infra |

Estado: las 8 fases **detalladas y commiteadas**. Ejecución de correcciones: en curso por cubetas.

## Clasificación por decisión requerida (cubetas)

Antes de ejecutar, los 64 hallazgos se clasificaron por cuánta deliberación necesitan:
- **🟢 Cubeta A — cero consulta** (práctica correcta única, tipo "encriptar contraseña"): ~12 puntos. Ejecutar directo.
- **🟡 Cubeta B — dirección fija**, solo un dato menor o mecánico: ~14 puntos (p.ej. CORS necesita la lista de orígenes; auditoría de columnas F6-A/B/C).
- **🔴 Cubeta C — requiere decisión** de producto/arquitectura: el resto (god services, atributos globales, endpoint fantasma, Sentry/métricas con destino, etc.).

## Estado de ejecución — Cubeta A (rama `limpieza/cubeta-a-obvios`)

**Aplicado y commiteado** (rama `limpieza/cubeta-a-obvios` en el submódulo backend, build `tsc` verificado, **sin push**):
- 🔒 **F7-07** — `iniciarSesion` filtra `eliminado:false` (usuario borrado ya no loguea).
- 🔒 **F7-04**-parte — `usuarios.buscarPorId` excluye `contrasenaHash` (fin de la fuga del hash).
- 🔒 **F1-04** — secreto JWT sin fallback `'cambia_estoseguro'`; el arranque **falla** si falta `JWT_SECRETO`.
- **F1-01 + F2-02** — `main.ts`: quitado volcado de config BD a logs, `enableShutdownHooks()` + `.catch()`, `console.log`→`Logger`.
- **F1-06 + F1-09** — eliminada dep muerta `uuid`, agregado `engines: node 20.x`.
- **F4-03** — imports muertos (`Op`, `ApiPropertyOptional`).
- **F5-01** — `@ApiBearerAuth()` en 19 controladores protegidos.

**Diferido** (obvio en dirección pero ejecución no trivial):
- **F7-02** hashear refresh tokens → el `jti` es PK *y* token a la vez; hashear exige refactor del flujo de emisión/lookup, no es 1 línea.
- **F6-D** `usuarioCreacion=negocioId` → el getter que crea la config no tiene `usuarioId`; necesita un "usuario sistema".
- **F4-06 / F5-03 / F5-04** — dispersos/verbosos.

**⚠️ Al desplegar:** F7-07 y F7-04 cambian comportamiento — probar login multi-negocio antes de mergear.

**Preferencia de trabajo aprendida:** nada de `perl -i`/`sed -i` masivo sobre `src/`; ediciones con Edit tool o subagente que use Read+Edit + `npm run build`.

**Siguiente:** Cubeta B (scoping tenant F7-03/05/08, CORS F7-10 con orígenes reales, throttler+helmet F7-11, auditoría columnas F6-A/B/C).

## Hallazgos de alto impacto

**Seguridad (Fase 7) — los graves:**
- `refresh-tokens.controlador` sin `@Permisos()`: `GET` lista tokens de todos y el `jti` ES el refresh token → **account takeover**. Además el token se guarda **sin hashear** (`hash = jti`).
- **IDOR**: `negocio-permisos` toma `negocioId` del path (no del token); `usuarios.buscarPorId` usa `findByPk(id)` sin `negocioId` y devuelve `contrasenaHash`.
- **Escalada cross-tenant**: `vincularRol`/`vincularPermisos` no validan que el `usuarioId` sea del negocio.
- `jwt.estrategia.validate()` recalcula permisos en cada request **sin `eliminado:false`** y **sin filtro de plan/nivel** → anula el filtrado del login (causa: lógica de permisos efectivos triplicada).
- `iniciarSesion` sin `eliminado:false`; FKs (categoría/sede) validadas sin `negocioId`; catálogo de atributos global sin ownership; CORS `origin:true`+`credentials:true`; **sin rate limiting / helmet**.

**Estándar de columnas (Fase 6):** ~25 `create()` setean `usuarioActualizacion` en el INSERT (viola [[venta-inventario]] regla NULL-al-crear); varios UPDATE no setean auditoría; `negocio-pagina-config` guarda `usuarioCreacion = negocioId` (mete id de negocio en FK a usuarios).

**Observabilidad (Fase 8):** casi nula. Solo `Logger` de Nest en 3 archivos + `/health` inline. Falta: structured logging (JSON), request logging + latencia, correlation id, métricas Prometheus, error tracking (Sentry), tracing (OpenTelemetry), readiness/liveness, y **audit log de eventos de seguridad** (crítico dado Fase 7).

## Correcciones de documentación detectadas

- `backend/CLAUDE.md` dice "TypeORM" — es **incorrecto**, el proyecto usa **Sequelize** (ya reflejado en el hub).
- Título Swagger genérico "API Proyecto Base"; falta `@ApiBearerAuth()` en todos los controladores.

## No tocar

- Envelope de respuesta `{ exito, mensaje, datos }` (español) diverge del estándar global `{ success, ... }`, pero el frontend (`HttpBaseService`) depende de `datos` → **cambiarlo rompe el frontend**. Documentado, fuera de scope.

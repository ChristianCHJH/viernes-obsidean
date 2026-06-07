---
titulo: Venta e Inventario Multi-Sede (Adifnex)
tipo: proyecto
tags: [inventario, ventas, multi-sede, fullstack, angular, nestjs, android, kotlin, sequelize, adifnex]
fecha_creacion: 2026-05-06
fecha_actualizacion: 2026-05-15
estado: activo
---

# Venta e Inventario Multi-Sede (Adifnex)

Sistema multi-plataforma de gestión de inventario multi-sede. Backend NestJS, frontend Angular 18, app Android nativa. Marca: **Adifnex**.

## Stack

| Capa | Tecnología |
| ---- | ---------- |
| Backend | NestJS 10 + **Sequelize 6.37** + PostgreSQL |
| Frontend | Angular 18 + PrimeNG + Tailwind + RxJS |
| Mobile | Kotlin + Jetpack Compose + Retrofit |
| Autenticación | JWT (15m) + Refresh Tokens (30d) |
| Documentación API | Swagger |

> **Excepción al estándar**: este proyecto usa **Sequelize**, no TypeORM.

## Estado actual

- Backend: autenticación completa + inventario completo + multi-tenancy; **114 unit tests (Jest) — Fase 1 completa**
- Backend (detalle): módulo usuarios con tenant isolation; endpoint `PATCH /autenticacion/usuarios/:id/negocio`; `secciones-permiso/rol/:rolId` expone `rolPermisoId`; `secciones-permiso/usuario/:id` retorna `asignado` + `asignadoPorRol`
- Frontend: RBAC funcional, sidebar con filtro por permisos, vistas rol-acceso y usuario-acceso con acordeones; lista de usuarios con columna "Rol" (badges azules); indicador "Desde rol" (badge violeta + `pi-id-card`) en vista de permisos de usuario; checkbox deshabilitado si permiso viene de rol
- Android: pantallas Splash, Login, Dashboard operativas

## Sub-páginas

- [[venta-inventario-auth]] — JWT, módulos de auth, tablas usuarios/refresh_token, BIGINT string fix
- [[venta-inventario-multitenancy]] — Estrategia tenant, tabla negocio, tiers, negocio_id en tablas
- [[venta-inventario-inventario]] — Módulos, tablas, endpoints, flujos (movimiento, transferencia, carga inicial)
- [[venta-inventario-rbac]] — RBAC 3 capas, tabla permisos, roles_permisos, SQL seed, strings en JWT
- [[venta-inventario-android]] — App Adifnex, MVVM, pantallas, TokenAuthenticator
- [[venta-inventario-frontend]] — Angular, extraerDatos(), sidebar, class-validator gotchas
- [[venta-inventario-testing]] — Jest unit tests backend (114 tests), estrategia E2E, mocks Sequelize
- [[venta-inventario-catalogo]] — Catálogo público Next.js, plantillas, permisos Plan 2, migración 014
- [[venta-inventario-correo]] — Módulo correo Gmail SMTP, bienvenida, recuperación contraseña

## Regla: consultar schema antes de escribir migraciones SQL

Antes de escribir cualquier `INSERT`, `UPDATE` o `ALTER` sobre las tablas del proyecto, **verificar la estructura real en `database/setup-completo.sql` o `database/autenticacion.sql`**.

Los nombres de columnas difieren del estándar que se asume por defecto:

| Tabla               | Columna real | Error común                  |
| ------------------- | ------------ | ---------------------------- |
| `permisos`          | `permiso`    | ~~`nombre`~~                 |
| `permisos`          | `seccion_id` | ~~`seccion_permiso_id`~~     |
| `roles`             | `rol`        | ~~`nombre`~~                 |
| `permiso_secciones` | `nombre`     | ✓ correcto                   |

**Error cometido (2026-05-11)**: se escribió `INSERT INTO permisos (nombre, seccion_permiso_id, ...)` sin verificar — fallaron 2 queries consecutivas.

**Regla operativa**: antes de cualquier migración, leer las 3–5 líneas de CREATE TABLE del archivo de schema.

---

## Links relacionados

- [[autenticacion-prototipo]] — patrón JWT compartido
- [[tecnologias/stack-principal]] — stack estándar (este proyecto usa Sequelize como excepción)
- [[tecnologias/patron-autenticacion-jwt]] — patrón auth completo
- [[driver-acompanamiento]] — otro proyecto Android

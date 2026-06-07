---
titulo: Render — Plataforma de despliegue
tipo: tecnologia
tags: [render, deploy, hosting, produccion, paas, docker]
fecha_creacion: 2026-05-29
fecha_actualizacion: 2026-05-29
estado: activo
---

# Render

PaaS elegido para el backend de venta-inventario después de que Railway agotó el trial (~$4.54 créditos, 2026-05-24). Free tier disponible con limitaciones.

## Migración Railway → Render (2026-05-29)

El backend `inventario-multisede-backend` se migró de Railway a Render. Mismo Dockerfile — no requirió cambios de código.

## Configuración del Web Service

- **Language**: Docker (no Node) — el `.npmrc` con `legacy-peer-deps=true` se copia automático en el build Docker; con Node mode podría no respetarse
- **Branch**: `main`
- **Region**: Oregon (US West)
- **Root Directory**: vacío (repo es solo el backend)

## Variables de entorno requeridas

```env
DB_HOST=<render postgres internal host>
DB_NAME=postgres
DB_PASS=<password>
DB_PORT=5432
DB_SCHEMA=inventario-ventas
DB_SSL=true
DB_USER=<usuario>
JWT_SECRETO=<mínimo 64 chars>
JWT_EXPIRACION=15m
JWT_REFRESH_EXPIRACION=30d
BACKEND_URL=https://<nombre>.onrender.com
STORAGE_PROVIDER=cloudinary
CLOUDINARY_CLOUD_NAME=dovxgcpc5
CLOUDINARY_API_KEY=<de consola Cloudinary>
CLOUDINARY_API_SECRET=<de consola Cloudinary>
```

**NO agregar `PORT`** — Render lo inyecta automáticamente. El backend ya lee `process.env.PORT || 3000` en `main.ts`.

## Gotchas críticos

### Free tier — cold start

El servicio duerme después de inactividad. Primera request tarda **30–50 segundos** en despertar. Si un cliente (ej. Vercel SSR) llama durante el cold start, el fetch puede timeout.

**Síntoma**: catálogo Next.js muestra `PaginaNoDisponible` aunque la BD y el código estén correctos.

**Diagnóstico**: abrir el endpoint directo en el browser para despertar el servicio → luego recargar el catálogo.

**Solución producción**: upgrade a Starter ($7/mes) o health check externo cada 10 min (ej. UptimeRobot free).

### Dockerfile EXPOSE vs PORT de Render

El Dockerfile tiene `EXPOSE 3200` (hardcodeado). Esto es solo documentación — Render ignora el EXPOSE y asigna el puerto via variable `PORT`. El CMD `node dist/main.js` arranca NestJS que lee `process.env.PORT`. Sin conflicto.

### Variables de entorno — sin morado como en Railway

A diferencia de Railway (donde las variables pendientes aparecen en morado), Render aplica los cambios de variables al próximo deploy. No hay indicador visual de "pendiente". Siempre hacer un nuevo deploy después de cambiar variables críticas.

## Links relacionados

- [[proyectos/venta-inventario-catalogo]] — F8 deploy, configuración completa
- [[tecnologias/railway]] — plataforma anterior (trial agotado)

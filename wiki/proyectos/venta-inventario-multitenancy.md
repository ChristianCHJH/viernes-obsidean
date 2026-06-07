---
titulo: Venta e Inventario — Multi-tenancy
tipo: proyecto
tags: [inventario, multi-tenancy, negocio, tiers, saas]
fecha_creacion: 2026-05-07
fecha_actualizacion: 2026-05-07
estado: activo
---

# Venta e Inventario — Multi-tenancy

Sub-página de [[venta-inventario]].

## TenantInterceptor — comportamiento superadmin

El `TenantInterceptor` determina el `negocioId` activo por request:

| Condición | `req.negocioId` asignado |
| --------- | ------------------------ |
| Usuario normal (nivelPlan 1–3) | `negocioId` del JWT |
| Superadmin (nivelPlan 4) + header `X-Negocio-Context` presente | valor del header |
| Superadmin (nivelPlan 4) + **sin** header `X-Negocio-Context` | `0` (ningún tenant) |

**Decisión de diseño (2026-05-07):** cuando el superadmin no ha seleccionado un contexto explícito, el sistema asigna `negocioId = 0`. Esto hace que todas las consultas filtradas por tenant devuelvan listas vacías. El superadmin debe seleccionar activamente un negocio desde el selector de contexto del frontend.

Comportamiento anterior (reemplazado): si no había header, se usaba el `negocioId` del propio JWT del superadmin. Ese fallback fue eliminado porque inducía a confusión — el superadmin no "pertenece" a un negocio operativamente.

## Estrategia

**Shared schema, row-level isolation.** Mismas tablas para todos los tenants, separación por `negocio_id`.

## Tabla `negocio`

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| nombre | VARCHAR(200) | |
| ruc | VARCHAR(20) | UNIQUE, nullable |
| correo_contacto | VARCHAR(150) | |
| nivel_plan | SMALLINT | 1=Básico 2=Profesional 3=Empresarial 4=Developer |
| max_usuarios | SMALLINT | límite según plan contratado |
| max_sedes | SMALLINT | límite según plan contratado |
| fecha_vencimiento | TIMESTAMPTZ | NULL = sin expiración |
| + auditoría estándar | | |

## Tablas que tienen `negocio_id`

| Tabla | negocio_id | Notas |
|-------|------------|-------|
| `usuarios` | NOT NULL FK | un usuario pertenece a un solo negocio |
| `producto` | NOT NULL FK | catálogo por negocio |
| `sede` | NOT NULL FK | sucursales por negocio |
| `stock_sede` | NOT NULL FK | aislamiento directo + índice sin JOIN |
| `movimiento_inventario` | NOT NULL FK | auditoría directa por tenant |
| `roles` | NULLABLE FK | NULL = rol global de plataforma |

## SKU único por negocio

`idx_producto_sku` (global) eliminado → reemplazado por:

```sql
UNIQUE INDEX uq_producto_sku_negocio (negocio_id, sku) WHERE eliminado=false
```

Dos negocios distintos pueden tener el mismo SKU.

## `negocioId` y `nivelPlan` en JWT

Ambos campos van en el payload para evitar queries adicionales en cada request. El guard filtra por tenant usando `negocioId` y valida acceso a secciones usando `nivelPlan` directamente desde el token.

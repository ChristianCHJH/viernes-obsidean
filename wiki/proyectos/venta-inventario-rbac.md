---
titulo: Venta e Inventario — RBAC y Permisos
tipo: proyecto
tags: [inventario, rbac, permisos, roles, guard, sequelize, belongstomany, niveles, migraciones]
fecha_creacion: 2026-05-07
fecha_actualizacion: 2026-05-18
estado: activo
---

# Venta e Inventario — RBAC y Permisos

Sub-página de [[venta-inventario]].

## Modelo (3 capas)

```
Rol (administrador, operario, gerente)
  └─ roles_permisos
      └─ Permiso (INVENTARIO_PRODUCTOS_VER, INVENTARIO_STOCK_MOVER, ...)
          └─ permiso_secciones (inventario, usuarios, reportes, administracion)

usuarios_roles:    Usuario → Rol (permisos heredados)
usuarios_permisos: Usuario → Permiso (permisos directos, sobreescribe)
```

`PermisosGuardia` valida que el usuario tenga **al menos uno** de los permisos requeridos.

## Tabla `permisos`

> La columna es `permiso` (no `nombre`). La FK es `seccion_id` (no `seccion_permiso_id`).

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| permiso | VARCHAR | código técnico: `ADMINISTRACION_NEGOCIOS_VER` |
| descripcion | VARCHAR | texto legible en español |
| seccion_id | BIGINT FK | → `permiso_secciones` |
| + auditoría estándar | | |

## Tabla `roles_permisos`

> Nombre correcto: `roles_permisos` (no `rol_permisos`).

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| rol_id | BIGINT FK | → `roles` |
| permiso_id | BIGINT FK | → `permisos` |
| + auditoría estándar | | |

## SQL — seed de permiso nuevo

Patrón para insertar un permiso y asignarlo al rol administrador (id=1):

```sql
-- 1. Insertar permiso (seccion_id=12 → ADMINISTRACION)
INSERT INTO permisos (permiso, descripcion, seccion_id, estado, eliminado, usuario_creacion, usuario_actualizacion, fecha_creacion, fecha_actualizacion)
VALUES ('ADMINISTRACION_NEGOCIOS_VER', 'Ver y gestionar negocios del sistema', 12, true, false, 1, 1, NOW(), NOW());

-- 2. Asignar al rol administrador
INSERT INTO roles_permisos (rol_id, permiso_id, estado, eliminado, usuario_creacion, usuario_actualizacion, fecha_creacion, fecha_actualizacion)
VALUES (1, (SELECT id FROM permisos WHERE permiso = 'ADMINISTRACION_NEGOCIOS_VER'), true, false, 1, 1, NOW(), NOW());
```

> Después de insertar permisos el usuario debe re-loguearse. El JWT no se actualiza en caliente.

## Decisión: permisos como strings en JWT

Los permisos van en el JWT como `string[]`, no como IDs numéricos. Razones:

1. **Guards legibles**: `@Permisos('INVENTARIO_PRODUCTOS_VER')` — con IDs sería `@Permisos(47)`
2. **Frontend sin roundtrip extra**: `tienePermiso('INVENTARIO_PRODUCTOS_VER')` directo
3. **Los códigos son estables**: no cambian entre entornos; el ID numérico puede diferir entre staging y producción

## Frontend — diseño accordion

Los componentes `rol-acceso` y `usuario-acceso` usan acordeones por sección:

- Cada `SeccionPermiso` = acordeón colapsable con badge `X de Y activos`
- Checkboxes en grid dentro del acordeón
- Botones "Expandir todo" / "Colapsar todo"
- Secciones abiertas por defecto al cargar

**Label**: muestra `descripcion` (español) como texto principal y `permiso` (código técnico) en monospace debajo.

**No hay tabla de "permisos asignados"** separada — checkboxes `disabled+checked` = ya asignados; `unchecked` = disponibles; `checked` = pendientes de guardar.

### Fix CSS — texto largo en grid

```css
.permiso-item { min-width: 0; }
.permiso-nombre { word-break: break-word; overflow-wrap: break-word; min-width: 0; }
```

---

## Indicador "Desde rol" en vista de permisos de usuario

**Fecha:** 2026-05-09

### Endpoint

`GET /api/autenticacion/secciones-permiso/usuario/:id`

### Backend — `secciones-permiso.servicio.ts`

Se ejecutan 3 queries paralelas con `Promise.all`:

1. Secciones + permisos del sistema
2. Permisos directos del usuario (`usuarios_permisos`)
3. Roles del usuario (`usuarios_roles`)

Si el usuario tiene roles, se consultan los permisos de esos roles via `roles_permisos`.

La respuesta por permiso ahora incluye:

- `asignado: boolean` — si el usuario tiene el permiso asignado directamente
- `asignadoPorRol: boolean` — si el permiso viene via un rol asignado al usuario

### Frontend — interface y UI

Campo nuevo en `PermisoAsignableUsuario`:

```typescript
asignadoPorRol: boolean;
```

Reglas de la UI:

- Si `asignadoPorRol === true`: checkbox **deshabilitado** + badge visual "Desde rol" (color violeta/purple, ícono `pi pi-id-card`)
- Si `asignado === true` y `asignadoPorRol === false`: checkbox checked y habilitado (permiso directo)
- Checkbox deshabilitado impide que el usuario quite permisos que vienen de un rol

---

## Bug resuelto: Sequelize BelongsToMany y alias `->` con `raw: true`

**Fecha:** 2026-05-09  
**Contexto:** columna "Rol" en tabla de usuarios — `listar()` en `usuarios.servicio.ts`

### Síntoma

```text
ERROR: no existe la columna "usuariosRoles->rol"."nombre"
// después del primer fix:
ERROR: no existe la columna "roles"."nombre"
```

### Causa raíz

1. Sequelize genera aliases de tablas con `->` cuando se usa `HasMany` + nested include (`UsuarioRol → Rol`). El alias resultante es `"usuariosRoles->rol"`. En algunos contextos PostgreSQL interpreta `->` como operador JSON.
2. Al cambiar a `BelongsToMany`, Sequelize genera el alias `"roles->UsuarioRol"` para la tabla junction — sigue usando `->`.
3. El segundo error `roles.nombre` fue porque la entidad `Rol` tiene el campo `rol` (no `nombre`).

### Fix definitivo

**En `usuario.entidad.ts`:**

```typescript
// CORRECTO: BelongsToMany directo
@BelongsToMany(() => Rol, {
  through: () => UsuarioRol,
  foreignKey: 'usuarioId',
  otherKey: 'rolId'
})
roles: Rol[];

// INCORRECTO: HasMany a tabla junction + nested include
@HasMany(() => UsuarioRol)
usuariosRoles: UsuarioRol[];
```

**En `usuarios.servicio.ts` — el include correcto:**

```typescript
include: [{
  model: Rol,
  through: { attributes: [], where: { estado: true, eliminado: false } },
  where: { eliminado: false },
  required: false,
  attributes: ['id', 'rol']   // campo es "rol", NO "nombre"
}]
```

**Remover `raw: true` del `findAll`** — Sequelize associations no funcionan en raw mode.

---

## nivelMinimo en permisos (migración 007)

Campo agregado a la tabla `permisos`:

```sql
nivel_minimo SMALLINT NOT NULL DEFAULT 1  -- 1=Básico | 2=Profesional | 3=Empresarial | 4=Developer
```

Controla qué tier de plan debe tener un negocio para que ese permiso esté disponible. Se edita inline desde la UI (ver frontend).

---

## Estructura de secciones — dos contextos distintos

| Sección DB | Para qué pantalla | Permisos |
|---|---|---|
| `ADMINISTRACION_NEGOCIO` | Lista de negocios (super admin, `/negocios`) | `ADMINISTRACION_NEGOCIOS_VER/CREAR/EDITAR/CAMBIAR_ESTADO/ELIMINAR` |
| `NEGOCIOS` | Mi Negocio del tenant (`/mi-negocio`) | `NEGOCIOS_VER`, `NEGOCIOS_EDITAR` |

**Error común**: agregar los permisos CRUD de negocios a la sección `NEGOCIOS` — esa sección es solo para la vista del tenant de sus propios datos.

---

## Cadena de migraciones (008 → 009 → 010)

### 008 — permisos de negocios + eliminar PLAN_ACTIVO

- Soft-delete de sección `PLAN_ACTIVO` y sus permisos (`PLAN_ACTIVO_VER`, `PLAN_ACTIVO_EDITAR`)
- Seed de `ADMINISTRACION_NEGOCIOS_*` en sección `ADMINISTRACION_NEGOCIO`
- Seed de `NEGOCIOS_VER`, `NEGOCIOS_EDITAR` en sección `NEGOCIOS`

### 009 — consolidar SEGURIDAD_PERMISOS en SEGURIDAD_SECCIONES

- La pantalla "Secciones y Permisos" es una vista unificada — no tiene sentido tener dos secciones en la DB
- `SEGURIDAD_PERMISOS_CREAR/EDITAR/CAMBIAR_ESTADO` movidos a `SEGURIDAD_SECCIONES`
- Soft-delete: `SEGURIDAD_PERMISOS_VER`, `SEGURIDAD_PERMISOS_ELIMINAR`, `SEGURIDAD_SECCIONES_ELIMINAR`, sección `SEGURIDAD_PERMISOS`
- Seed de `SEGURIDAD_USUARIOS_VER_HISTORIAL` (faltaba desde 003)
- Assign de todos los permisos nuevos al rol `SUPERADMIN`

### 010 — limpiar permisos sin uso en UI

Soft-delete de 10 permisos activos que ningún `*appPermiso` ni route guard referencia:
`DASHBOARD_VER`, `INVENTARIO_PRODUCTOS_CAMBIAR_ESTADO`, `INVENTARIO_PRODUCTOS_EXPORTAR`, `INVENTARIO_SEDES_CAMBIAR_ESTADO`, `INVENTARIO_STOCK_EXPORTAR`, `INVENTARIO_MOVIMIENTOS_EXPORTAR`, `INVENTARIO_DISTRIBUCION_EXPORTAR`, `SEGURIDAD_USUARIOS_ELIMINAR`, `SEGURIDAD_ROLES_ELIMINAR`, `ADMINISTRACION_NEGOCIO_PERMISOS_EDITAR`.

Sección `DASHBOARD` queda vacía → soft-delete también.

**Regla**: cuando se agrega una migración con nuevos permisos, siempre incluir el INSERT en `roles_permisos` para `SUPERADMIN` en la misma migración. 003 asignó los permisos existentes en ese momento; los nuevos de cada migración posterior deben asignarse explícitamente.

---

## Diagnóstico de permisos — script

`database/diagnostico-permisos.sql` — compara permisos activos en DB vs. usados en el frontend.

Produce dos resultados:
1. Todos los permisos activos con `✓ SÍ / ✗ NO` si están en algún `*appPermiso` o route guard
2. Permisos referenciados en el frontend que NO existen en DB (rompen funcionalidad silenciosamente)

Patrón: CTE `permisos_frontend(permiso, origen)` con todos los permisos usados en código → LEFT JOIN contra `permisos` DB.

---

## Guard de ruta — mi-negocio

Cambio en `panel.component.ts` PERMISOS_VISTA:

```typescript
// Antes (PLAN_ACTIVO_VER fue eliminado en migración 008)
'mi-negocio': 'PLAN_ACTIVO_VER',

// Después
'mi-negocio': 'NEGOCIOS_VER',
```

Y en `mi-negocio-vista.component.html`, los botones de editar cambiaron de `PLAN_ACTIVO_EDITAR` → `NEGOCIOS_EDITAR`.

---

### Lecciones críticas

| Regla | Detalle |
| ----- | ------- |
| La entidad `Rol` usa `rol` como nombre del rol | Nunca asumir `nombre` — siempre verificar la entidad |
| `raw: true` rompe associations | Si se necesitan includes/associations, nunca usar `raw: true` |
| `BelongsToMany` > `HasMany` + nested include | Para relaciones M:N, usar siempre `BelongsToMany` directamente |

### Frontend — mapper de roles en `ElementoListaUsuario`

```typescript
// Interface
export interface ElementoListaUsuario {
  // ...otros campos...
  roles: string[];
}

// Mapper — extraer campo "rol" (no "nombre")
roles: Array.isArray(crudo['roles'])
  ? crudo['roles'].map((r: { rol: string }) => r.rol)
  : []
```

UI: badges pill azules en columna "Rol" de la tabla de usuarios.

---

## Regla crítica: `@NegocioActual()` vs `req.user.negocioId`

**Regla**: todo controller que sirva datos filtrados por negocio (tenant-scoped) DEBE usar el decorador `@NegocioActual()`, nunca `req.user.negocioId` directamente.

### Por qué

`TenantInterceptor` (`src/comun/interceptores/tenant.interceptor.ts`) setea `request.negocioId` con la siguiente lógica:

- Si el header `X-Negocio-Context` está presente **y** el usuario es SUPERADMIN (nivelPlan === 4) → usa el valor del header
- En cualquier otro caso → usa `negocioId` del JWT

`@NegocioActual()` lee `request.negocioId` (ya procesado por el interceptor). Un controller que usa `req.user.negocioId` bypasea el interceptor — el cambio de contexto del superadmin no tiene efecto.

### Bug documentado (2026-05-18)

`negocio.controlador.ts` usaba `req.user.negocioId` en 4 métodos:

- `obtenerMiNegocio`, `actualizarMiNegocio`, `obtenerPaginaConfig`, `actualizarPaginaConfig`

Al cambiar negocio desde el selector en el panel, "Mi Página" y "Mi Negocio" seguían devolviendo datos del negocio del JWT. Fix: reemplazar todos por `@NegocioActual() negocioId: number`.

### Checklist al crear un nuevo controller tenant-scoped

```typescript
// ✓ correcto — respeta TenantInterceptor
import { NegocioActual } from '../../comun/decoradores/negocio-actual.decorador';

@Get()
listar(@NegocioActual() negocioId: number) {
  return this.servicio.listar(negocioId);
}

// ✗ incorrecto — bypasea TenantInterceptor
@Get()
listar(@Request() req: { user: { negocioId: number } }) {
  return this.servicio.listar(req.user.negocioId);
}
```

---
titulo: Frontend — Permisos RBAC y UI
tipo: proyecto
tags: [inventario, angular, rbac, permisos, signals]
fecha_creacion: 2026-06-07
fecha_actualizacion: 2026-06-07
estado: activo
---

# Permisos RBAC y UI

Sub-página de [[venta-inventario-frontend]].

## Modal de reasignación de negocio

- Botón con icono `pi-building` visible por fila, solo para `esSuperAdmin()`.
- Abre un dialog con dropdown que carga negocios de forma lazy.
- Dropdown usa `appendTo="body"` — evita z-index issues (ver [[venta-inventario-frontend-gotchas]]).
- Llama `PATCH /autenticacion/usuarios/:id/negocio` con `{ negocioId }`.

## Permisos de rol — auto-save y seleccionar todo

**Flujo actual** (sin botón "Guardar" — todo es inmediato):

- **Checkbox individual** → `togglePermisoInmediato()`: si no asignado → POST batch; si asignado → DELETE con `rolPermisoId`
- **Botón de sección** → `seleccionarTodaSeccion()`: si hay sin asignar → batch assign; si todos asignados → `forkJoin` de DELETEs en paralelo
- Spinners individuales: `guardandoPermisoId` (por permiso) y `guardandoSeccionId` (por sección)

```typescript
interface PermisoAsignableRol {
  // ...campos previos...
  rolPermisoId?: number | string | null; // ID de `roles_permisos` para DELETE directo
}
```

## Vista unificada Secciones + Permisos

**Decisión**: dos páginas (`/sections` y `/permissions`) unificadas en `/dashboard/secciones-permisos`.
**ADR**: `ADR-secciones-permisos-unificado.md` en raíz del proyecto.

`GET /autenticacion/secciones-permiso` ya devuelve `permisos[]` anidados — cero cambios backend.

```typescript
export interface SeccionPermisoItem {
  id: number;
  permiso: string;
  descripcion?: string | null;
  estado: boolean;
}
// agregado a SeccionPermiso:
permisos?: SeccionPermisoItem[];
```

**Endpoints huérfanos**: `GET /autenticacion/permisos` y `GET /autenticacion/permisos/:id` — sin consumidor frontend. Reservados para limpieza.

**Componente activo**: `features/secciones/secciones-con-permisos-lista/` — tabla expandible con chevron, sub-tabla con `border-left: 4px solid var(--spa-primary)`, `ConfirmDialogComponent` Material, `*appPermiso` en todos los botones.

## Selector inline de nivelMinimo (2026-05-11)

Reemplaza badge estático por 4 pills clickeables (Bás./Prof./Emp./Dev.) que hacen PATCH inmediato.

```typescript
readonly nivelCargando = signal<Set<number>>(new Set());        // por permiso
readonly seccionNivelCargando = signal<Set<number>>(new Set()); // por sección
```

**Cambio individual**: agrega `permisoId` al Set → deshabilita pills → PATCH → `finalize` quita ID → `next` parchea signal in-place.

**Aplicar a toda sección**:
```typescript
forkJoin(permisos.map(p =>
  this.servicioPermisos.actualizarPermiso(p.id, { nivelMinimo: nivel, usuarioActualizacion: 1 })
))
```
Marca `seccionNivelCargando` Y todos los `permisoId` → `forkJoin` paralelo → `finalize` limpia ambos Sets → `next` parchea todos.

**CSS pills**:
```css
.nivel-btn.active.nivel-1 { background: #e0f2fe; color: #0369a1; border-color: #7dd3fc; }
.nivel-btn.active.nivel-2 { background: #d1fae5; color: #065f46; border-color: #6ee7b7; }
.nivel-btn.active.nivel-3 { background: #fef3c7; color: #92400e; border-color: #fcd34d; }
.nivel-btn.active.nivel-4 { background: #ede9fe; color: #5b21b6; border-color: #c4b5fd; }
```

> El `<thead>` de la sub-tabla está dentro del `@for (s of seccionesFiltradas())` → `@if (estaExpandida(s.id))`, por eso `s` está en scope en el header de la sub-tabla.

## Regla permanente: permisos en cada módulo nuevo

- **Backend**: bloque SQL de INSERT en `003-seed-permisos.sql` + `@Permisos()` en cada endpoint.
- **Frontend**: cada botón/acción protegida lleva `*appPermiso="'SECCION_ACCION'"` (mayúsculas).
- Ambas cosas van juntas — no se entrega frontend sin el SQL correspondiente.

## Regla: botones en versión mobile

Cada botón de acción agregado en tabla desktop (`hidden md:block`) **también debe estar** en tarjetas mobile (`block md:hidden`). Son dos bloques independientes.

**Error documentado (2026-05-11)**: botón "Ver historial" agregado solo en desktop — mobile quedó sin él.

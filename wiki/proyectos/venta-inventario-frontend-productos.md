---
titulo: Frontend — Productos, Marcas y Etiquetas
tipo: proyecto
tags: [inventario, angular, productos, marcas, etiquetas, bugs]
fecha_creacion: 2026-06-07
fecha_actualizacion: 2026-06-07
estado: activo
---

# Productos, Marcas y Etiquetas

Sub-página de [[venta-inventario-frontend]].

## Marcas y Etiquetas — páginas de gestión (2026-05-14)

Dos vistas CRUD bajo `features/inventario/`:

- `marcas-lista/` — tabla + cards mobile, dialog crear/editar, confirm eliminar. Columnas: ID, Nombre, Estado, Acciones.
- `etiquetas-lista/` — igual que marcas + color picker (12 colores predefinidos) + dot de color inline en columna Nombre.

Servicios extendidos en `core/services/`: `marcas-inventario.servicio.ts` y `etiquetas-inventario.servicio.ts` con `actualizar(id, datos)` y `eliminar(id)`.

Sidebar: ítems `marcas` y `etiquetas` bajo INVENTARIO, protegidos con `INVENTARIO_MARCAS_VER` / `INVENTARIO_ETIQUETAS_VER`.

Permisos SQL (migración 011): secciones `INVENTARIO_MARCAS` e `INVENTARIO_ETIQUETAS` con `VER/CREAR/EDITAR/ELIMINAR`.

## Bug: lista de productos renderizaba 1 fila + círculos grises (2026-05-14)

**Causa raíz**: `productos.servicio.ts` no tenía `include: [Etiqueta]` porque `Producto` no declaraba `@BelongsToMany`. Sequelize ignora `include` silenciosamente si no hay decoradores → `p.etiquetas` llega `undefined` → `@for (tag of p.etiquetas)` crashea silenciosamente → filas renderizadas como círculos grises del skeleton.

**Fixes aplicados**:
1. `producto.entidad.ts` — agregado `@BelongsToMany(() => Etiqueta, () => ProductoEtiqueta)`
2. `producto-etiqueta.entidad.ts` — `@ForeignKey` y `@BelongsTo` en ambas FKs
3. `productos.modulo.ts` — `Etiqueta` registrada en `SequelizeModule.forFeature`
4. `productos.servicio.ts` — `include: [{ model: Etiqueta, through: { attributes: [] }, required: false }]` + literal `marcaNombre`
5. `productos-inventario.servicio.ts` (frontend) — `etiquetas?: Etiqueta[]` (optional)
6. `productos-lista.component.html` — guards `(p.etiquetas ?? [])` en bloques desktop y mobile

**Regla**: Sequelize no puede eager-load si no se declaran `@BelongsToMany`/`@HasMany`/`@BelongsTo`. La ausencia no produce error — solo devuelve el campo ausente.

## Bug: SQL migration ON CONFLICT falla sin constraint (2026-05-14)

`ON CONFLICT (nombre) DO NOTHING` falla con `42P10` si la columna no tiene UNIQUE constraint.

**Fix universal**: reemplazar con `WHERE NOT EXISTS (SELECT 1 FROM tabla WHERE col = valor)`.

## Tabla productos — rediseño UI compacta (2026-05-14)

- **SKU + Descripción** fusionados: nombre en negrita, SKU badge debajo, descripción truncada. Columnas SKU y Descripción independientes eliminadas.
- **Paginación client-side** (20 por página): `paginaActual`, `totalPaginas`, `productosPaginados` como computed signals. Effect resetea a página 1 al cambiar filtro.
- **Sorting client-side**: `ordenCampo` + `ordenDir` signals; headers clickeables con íconos.
- **Scroll sin overflow de página** (cadena flex):
  ```
  :host (display:flex; height:100%) → .shell (flex:1) → .data-card (flex:1; overflow:hidden)
  → .table-wrapper (flex:1; flex column) → .table-scroll (flex:1; overflow-y:auto) ← scroll aquí
  <footer> fuera de .table-scroll, dentro de .table-wrapper (sticky)
  ```

## Bug: mobile solo mostraba N primeros productos (2026-05-14)

**Causa**: cards mobile usaban `productosFiltrados()` (sin paginar) sin `overflow-y:auto` ni altura definida.

**Fix**: cambiar a `productosPaginados()` en el `@for`, agregar footer de paginación, y `.mobile-wrapper { display:flex; flex-direction:column; flex:1; min-height:0; overflow-y:auto }`.

## Bug: etiquetas no se guardaban al editar producto (2026-05-14)

**Problema 1** (frontend): `p-multiSelect` devolvía IDs como strings. Fix en `guardar()`:
```typescript
etiquetaIds: (v.etiquetaIds ?? []).map(Number),
marcaId: v.marcaId != null ? Number(v.marcaId) : null,
```

**Problema 2** (backend, causa principal): `ignoreDuplicates: true` genera `ON CONFLICT DO NOTHING`. Si el par `(productoId, etiquetaId)` ya existía con `eliminado: true`, el insert se ignoraba.

```typescript
// Fix: updateOnDuplicate en lugar de ignoreDuplicates
await this.productoEtiquetaModelo.bulkCreate(
  etiquetaIds.map(etiquetaId => ({ productoId: id, etiquetaId, eliminado: false, ... })),
  { updateOnDuplicate: ['eliminado', 'usuarioActualizacion'] },
);
```

**Regla**: en tablas con soft-delete + relación N:M, nunca usar `ignoreDuplicates: true`. Usar `updateOnDuplicate` con `eliminado: false` explícito.

## UX refactoring productos-lista (sesión 31, 2026-06-04)

| Elemento | Antes | Después |
|----------|-------|---------|
| 11 columnas → scroll horizontal | — | Eliminadas columnas `CREADO POR` y `ACTUALIZADO POR` |
| Chips apilados — fila variable | — | Máximo 2 chips + badge `+N` |
| Label "BUSCAR" redundante | — | Eliminado — icono lupa suficiente |
| Precio alineado izquierda | — | `.num-col { text-align: right }` |
| Botón "Nuevo producto" | Azul plano | Gradiente `#2b7ea6→#3490bc` + sombra + hover lift |
| Row hover | Fondo gris | Fondo tintado + `inset 3px 0 0 var(--spa-primary)` |
| Badge Activo/Inactivo | Solo texto | Punto `::before` color + texto |
| Modal local | `border-radius: 8px` | `border-radius: 20px` + `border-top: 3px solid primary` |

---
titulo: Frontend — productos-premium UX Mobile y Tab Distribución
tipo: proyecto
tags: [inventario, angular, productos-premium, mobile, ux, bottom-sheet]
fecha_creacion: 2026-06-07
fecha_actualizacion: 2026-06-07
estado: activo
---

# productos-premium — UX Mobile y Tab Distribución

Sub-página de [[venta-inventario-frontend]]. Ver también [[venta-inventario-frontend-productos-premium]].

## Tab Distribución — 4° tab del modal (sesión 34)

Layout:
```
.cat-modal-body-full (full-width, sin preview-col)
└── .dist-layout (grid 2 col → 1 col en mobile <680px)
    ├── .dist-stock-panel  ← tabla stock por sede
    └── .dist-mov-panel    ← formulario movimiento
```

Cuando `tabActivo() === 'distribucion'` usa `cat-modal-body-full` (sin panel preview).

**Selector de tipo** con 3 botones semánticos:

| Tipo | Color activo | Endpoint |
|------|-------------|----------|
| INGRESO | fondo `#dcfce7`, texto `#16a34a` | `POST /stock/movimiento { tipo: 'INGRESO' }` |
| SALIDA | fondo `#fee2e2`, texto `#dc2626` | `POST /stock/movimiento { tipo: 'SALIDA' }` |
| TRANSFERENCIA | fondo `#eff6ff`, texto `#2563eb` | `POST /stock/transferencia` |

**Hint stock disponible**: para SALIDA y TRANSFERENCIA muestra `ℹ Disponible: N uds` naranja bajo el select de sede origen. Calculado desde `stockSedeSeleccionada()` computed.

**Badge en tab button**: `@if (distStockTotal() > 0) { <span class="cat-tab-badge">{{ distStockTotal() }}</span> }` — visible desde cualquier tab.

**Caché de sedes**: `if (this.sedes().length > 0) return;` — carga una sola vez por sesión de modal. Stock se recarga al abrir tab Y después de cada operación exitosa.

## Fix: vista previa visible en tab Distribución (sesión 35)

**Causa**: `.cat-modal-body-full` colapsa el grid pero `.cat-preview-col` permanecía en el DOM.

```html
@if (tabActivo() !== 'distribucion') {
  <div class="cat-preview-col">...</div>
}
```

## Fix: texto `dist-table` ilegible (sesión 35)

**Causa**: selector global `thead tr { background: linear-gradient(90deg, #14532D, #166534) }` aplica a TODOS los thead del componente. Color muted `#6B7280` era invisible sobre fondo verde oscuro.

```css
.dist-table th {
  text-align: left; font-size: .75rem; letter-spacing: .06em;
  text-transform: uppercase; color: rgba(255,255,255,.85); font-weight: 600;
}
```

**Regla**: cualquier tabla nueva que viva en este componente debe sobreescribir `color` a blanco en sus `th`.

## UX mobile modal edición (sesión 36)

### 1 — Preview como bottom sheet

`.cat-preview-col` oculta en `@media (max-width: 768px)`. Signal `previewMobileVisible = signal(false)`. Botón trigger en tab Catálogo (solo visible mobile) abre `.cat-preview-sheet-overlay` (z-index: 9200, `border-radius: 22px 22px 0 0`, animación `cat-sheet-up`).

### 2 — Modal con altura fija

```css
.cat-modal { height: min(88vh, 720px); }  /* antes: max-height: 90vh */
@media (max-width: 768px) { .cat-modal { height: 90vh; } }
@media (max-width: 480px) { .cat-modal { height: 92vh; } }
```

**Regla**: modales con tabs de contenido variable deben tener `height` fijo, no `max-height`.

### 3 — Formulario movimiento como bottom sheet en mobile

**Causa scroll horizontal**: doble padding — `.dist-layout` tiene `padding: 1.25rem` dentro de `.cat-tab-content` que también tiene `padding: 1.5rem`.

```css
@media (max-width: 768px) {
  .dist-mov-panel { display: none; }
  .dist-layout { padding: 0; }
  .cat-tab-content { overflow-x: hidden; }
  .dist-table { table-layout: fixed; }
}
```

Signal `movModalMobileVisible = signal(false)`. Al registrar exitosamente → `movModalMobileVisible.set(false)`. El formulario usa los mismos signals que el panel inline.

## Card redesign mobile: `prod-row` (sesión 44)

**Problema**: 4 filas apiladas (~180–200px) → solo 2 productos visibles en pantalla.
**Solución**: fila flex compacta `prod-row` — 2 líneas en ~76px → 4–5 productos visibles.

```
[avatar 28px] [body flex-col] [actions 2 botones]
               ├─ L1: nombre ················· dot de estado
               └─ L2: sku · precio · marca · chips
```

| Elemento | Antes | Después |
|---------|-------|---------|
| Estado | Pill texto | Dot 7px animado (pulse verde / gris estático) |
| Marca | Fila separada | Inline en L2 separado por `·` |
| Etiquetas | Chips grandes | `tag-chip-tiny` 0.62rem, máx 2 + badge `+N` |
| Avatar | Ausente | 28×28px `border-radius:8px` con gradiente |
| Acciones | Botones large | 28×28px con `mat-icon 13px` |

**Gotcha**: CSS Grid con `grid-row: span` mezcla colocación explícita e implícita — auto-placement impredecible. Usar flexbox con div wrapper explícito.

## Fix: paginación desaparece en mobile (sesión 44)

**Causa**: `.mobile-wrapper` con `overflow-y: auto` hacía que cards + footer fueran un solo bloque scrolleable.

```html
<div class="mobile-wrapper">
  <div class="mobile-list">
    @for (p of productosPaginados(); track p.id) { ... }
  </div>
  <footer class="mobile-footer">...</footer>
</div>
```

```css
.mobile-wrapper { display: flex; flex-direction: column; flex: 1; min-height: 0; overflow: hidden; }
.mobile-list    { flex: 1; min-height: 0; overflow-y: auto; }
.mobile-footer  { flex-shrink: 0; border-top: 1px solid var(--p-border); }
```

**Patrón generalizado** (equivalente mobile del desktop `.table-scroll + footer`):
```
.mobile-wrapper (overflow:hidden)
  → .mobile-list   (flex:1, overflow-y:auto)  ← scroll aquí
  → .mobile-footer (flex-shrink:0)             ← siempre visible
```

## Eliminación de botones redundantes (sesión 37)

Botones `dist` y `edit` de `.row-actions` y `.card-actions` eliminados. Solo quedan **Catálogo** y **Eliminar** por fila — el modal unificado ya tiene Tab Producto (editar) y Tab Distribución.

## Eliminación summary badge redundante (sesión 44)

`<span class="summary-badge">≡ N productos</span>` en toolbar eliminado — el conteo ya aparece en la KPI card "Total productos". No duplicar métricas entre toolbar y KPI cards.

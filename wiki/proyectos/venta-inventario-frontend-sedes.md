---
titulo: Frontend — Sedes y Stock
tipo: proyecto
tags: [inventario, angular, sedes, stock, drawer, design-system, adifnex]
fecha_creacion: 2026-06-07
fecha_actualizacion: 2026-06-07
estado: activo
---

# Sedes y Stock

Sub-página de [[venta-inventario-frontend]].

## sedes-lista — rediseño premium completo (sesión 45)

Componente reescrito desde cero con el mismo design language que `productos-premium`.

### Design tokens y KPI bar

| KPI | Gradiente |
|-----|-----------|
| Total sedes | `#14532D → #166534` (verde oscuro) |
| Activas | `#10B981 → #059669` (emerald) |
| Inactivas | `#6B7280 → #4B5563` (slate) |
| Con dirección | `#7C3AED → #6D28D9` (violet) |

Violet para "localización" — diferencia semánticamente del verde (inventario). También en el botón "Ver stock".

### Estructura de la pantalla

```
toolbar (búsqueda + botón crear)
KPI bar (4 cards con gradientes)
data-card (tabla desktop + mobile-wrapper — scroll interno)
  tabla: header verde, col: #/avatar+nombre / dirección / creador / estado / acciones
  mobile: avatar 28px, 2 líneas (nombre+dot | pin+dirección), 2 botones
paginación (desktop footer + mobile footer — siempre visible)
```

### Avatar por hash (reutilizable)

```typescript
private readonly GRADIENTS = [ /* 8 gradientes */ ];

avatarGradient(nombre: string): string {
  const hash = nombre.split('').reduce((a, c) => a + c.charCodeAt(0), 0);
  return this.GRADIENTS[hash % this.GRADIENTS.length];
}
```

Mismo patrón que `productos-premium` — reutilizable en cualquier entidad sin imagen.

### Modal premium (`cat-overlay`)

- Ícono con gradiente verde (crear) o azul (editar); strip identidad en modo editar
- Campo nombre + textarea dirección (4 filas); caja decorativa violet "mapa"
- Footer: Cancelar (ghost) + Guardar (verde / loading)
- **Mobile bottom sheet** (`< 600px`): `border-radius: 18px 18px 0 0`, `align-items: flex-end`, animación slide-up

### Signals y computed

```
sedes()            → raw array desde API
sedesFiltradas     → filtrado por texto (nombre + dirección)
sedesOrdenadas     → sorted por ordenCampo/ordenDir
sedesPaginadas     → slice por página
totalActivas       → count donde s.estado === true
totalInactivas     → count donde s.estado === false
totalConDireccion  → count donde s.direccion?.trim() tiene valor
```

### Permisos

| Acción | Permiso |
|--------|---------|
| Crear | `INVENTARIO_SEDES_CREAR` |
| Ver stock | `INVENTARIO_STOCK_VER` |
| Editar | `INVENTARIO_SEDES_EDITAR` |
| Eliminar | `INVENTARIO_SEDES_ELIMINAR` |

### Cambios en `panel.component`

```css
/* panel.component.css — regla combinada full-height */
.view-area.view-productos,
.view-area.view-sedes { overflow: hidden; display: flex; flex-direction: column; padding: 0; }
```

Skeleton: 6 filas desktop / 5 filas mobile con `@keyframes cat-shimmer`.

## sede-stock convertido a drawer (sesión 46)

La ruta `/dashboard/sede-stock` fue eliminada. `SedeStockComponent` ahora es un slide-over drawer dentro de `sedes-lista`.

### Cambios en `panel.component`

- Eliminado `SedeStockComponent` de `imports[]`
- Eliminado `'sede-stock'` de `PERMISOS_VISTA` y `tituloPagina()`
- Eliminado `@case ('sede-stock')` del switch
- Eliminados métodos `alVerStockSede()`, `alVolverSedes()` y propiedades `sedeSeleccionadaStock`, `claveAlmacSedeStock`

### Cambios en `sedes-lista`

- Signal: `readonly sedeStock = signal<Sede | null>(null)`
- Métodos: `abrirStock(sede)` y `cerrarStock()`
- `SedeStockComponent` agregado a `imports[]`

### Patrón del drawer

```html
@if (sedeStock()) {
  <div class="stock-backdrop" (click)="cerrarStock()">
    <aside class="stock-drawer" (click)="$event.stopPropagation()">
      <div class="stock-drawer-body">
        <inv-sede-stock [sede]="sedeStock()" [enModal]="true" (volver)="cerrarStock()"></inv-sede-stock>
      </div>
    </aside>
  </div>
}
```

| Clase | Valor clave |
|-------|------------|
| `.stock-backdrop` | `position:fixed; inset:0; z-index:9000; backdrop-filter:blur(4px)` |
| `.stock-drawer` | `position:fixed; top:0; right:0; bottom:0; width:min(86vw,1100px); z-index:9001` |
| Mobile | `width:100vw; border-left:none` |

Header con gradiente `#14532D → #065F46`. Body background `#F0FDF4` + dot pattern.
Animaciones: backdrop `stockFadeIn .2s`; drawer `stockSlideIn .3s cubic-bezier(.32,.72,0,1)`.

### `@Input() enModal` en `SedeStockComponent`

Cuando `enModal === true`: oculta `<header class="page-header">` (con botón volver); muestra KPI strip: productos en sede, unidades totales, stock bajo (≤5).

## Toggle custom + select nativo en sede-stock (sesión 46)

### Toggle de tipo de movimiento

Eliminado `MatButtonToggleModule`. 3 botones nativos con color semántico:

```html
<div class="tipo-toggle" role="group">
  <button type="button" class="tipo-btn"
    [class.tipo-active]="tipoMovimiento === 'INGRESO'"
    [class.tipo-ing]="tipoMovimiento === 'INGRESO'"
    (click)="alCambiarTipo('INGRESO')">
    <mat-icon>add_circle</mat-icon><span>Ingreso</span>
  </button>
  <!-- Salida (rojo #B91C1C) y Transferencia (azul #1D4ED8) igual -->
</div>
```

### Select nativo con flecha custom

```html
<div class="select-wrap">
  <select class="custom-select" formControlName="productoId">
    <option [ngValue]="null" disabled hidden>Selecciona un producto</option>
    @if (tipoMovimiento === 'TRANSFERENCIA') { @for... }
    @else if (tipoMovimiento === 'SALIDA')   { @for... }
    @else                                    { @for... }
  </select>
  <mat-icon class="select-caret">expand_more</mat-icon>
</div>
```

`.select-caret` tiene `pointer-events:none; position:absolute; right:.7rem` — rota 180° al `:focus-within`.

**Gotcha**: usar `[ngValue]` (no `value=""`) para `null` — `value` solo pasa strings. Funciona porque `FormsModule` provee `SelectControlValueAccessor`.

### SALIDA como 3er tipo

`StockInventarioServicio.registrarMovimiento()` ya aceptaba `tipo: 'INGRESO' | 'SALIDA'`. El componente solo tenía INGRESO y TRANSFERENCIA. Solo se extendió el tipo.

### KPI strip — `stockBajo` computed

```typescript
readonly stockBajo = computed(() => this.stockItems().filter(i => i.cantidad <= 5).length);
```

Chip ámbar con `warning_amber` solo cuando `stockBajo() > 0`.

---
titulo: Frontend — Componente productos-premium
tipo: proyecto
tags: [inventario, angular, productos-premium, modal, design-system, adifnex]
fecha_creacion: 2026-06-07
fecha_actualizacion: 2026-06-07
estado: activo
---

# Componente productos-premium

Sub-página de [[venta-inventario-frontend]].

**Ubicación**: `features/inventario/productos-premium/`
**Ruta**: `/dashboard/productos-premium`
**Permiso**: `INVENTARIO_PRODUCTOS_VER`

## Design system Adifnex — paleta `--p-*` (sesión 32)

| Token | Valor | Uso |
|-------|-------|-----|
| `--p` | `#14532D` | Verde oscuro — primario |
| `--p-accent` | `#EA580C` | Naranja — CTA, logo/avatar |
| `--p-emerald` | `#10B981` | Activo / positivo |
| `--p-amber` | `#F59E0B` | Precio / destaque |
| `--p-bg` | `#F0FDF4` | Fondo mint |
| `--p-surface` | `#FFFFFF` | Cards y tabla |
| `--p-text` | `#111827` | Texto oscuro |
| `--p-muted` | `#6B7280` | Texto secundario |
| `--p-border` | `#D1FAE5` | Bordes verdes sutiles |

Tokens promovidos a globales en `styles.css` y `tailwind.config.js` (sesión 39). Ver [[venta-inventario-frontend-diseno]].

## Elementos visuales clave

- **KPI bar**: 4 cards con gradientes independientes (total/activos/con precio/en catálogo). Hover `translateY(-3px)`.
- **Avatares**: gradiente determinista calculado desde hash del nombre (`avatarGradient(nombre)`) — 10 pares de colores.
- **Status activo**: badge `#ECFDF5` con `::before` pulsante `animation: pulse-dot 2s`.
- **Tag chips**: glass-style con `color-mix(in srgb, var(--eq-color) 18%, transparent)` + `backdrop-filter: blur(2px)`.
- **Botón CTA "Nuevo producto"**: gradiente naranja `#EA580C → #C2410C`.
- **Header tabla**: `background: linear-gradient(90deg, #14532D, #166534)` — texto blanco.
- **Paginación**: página activa con `box-shadow: 0 2px 8px rgba(20,83,45,.38)`.

## Modal unificado — 4 tabs (sesión 33 + 34)

Estructura:
```
.cat-overlay (z-index: 9000)
└── .cat-modal (max-width: 1060px, height: min(88vh, 720px))
    ├── header (nombre + X)
    ├── .cat-strip (acento naranja)
    └── body
        ├── .cat-form
        │   ├── .cat-tabs (sticky, no scrollea)
        │   └── .cat-tab-content (flex:1, overflow-y:auto)
        └── .cat-preview-col (imagen + KPIs — oculto en Distribución)
```

Override obligatorio: `.cat-form { overflow: hidden !important; padding: 0 !important; gap: 0 !important; }`

### Tab "Producto"
Nombre, SKU (toggle auto/manual), stock mínimo, marca (`<select>` nativo con SVG arrow), etiquetas (chips clickables con `color-mix`).

### Tab "Catálogo"
`catVisible` (toggle), `catPrecio`, `catPrecioOferta`, `catDescripcion`, `catOrden`.

### Tab "Imágenes"
Grid thumbnails 88×88px. Sub-modal de confirmación (z-index 9100). Toggle "Marcar como principal".

### Tab "Distribución" (sesión 34)
Ver [[venta-inventario-frontend-productos-premium-ux]] para detalle de layout y signals.

## Signals principales del componente

```typescript
readonly tabActivo = signal<'producto' | 'catalogo' | 'imagenes' | 'distribucion'>('catalogo');
readonly catNombre = signal(''); readonly catSkuManual = signal(false); readonly catSku = signal('');
readonly catStockMinimo = signal<number | null>(null); readonly catMarcaId = signal<number | null>(null);
readonly catEtiquetaIds = signal<number[]>([]);
readonly marcas = signal<Marca[]>([]); readonly etiquetas = signal<Etiqueta[]>([]);
readonly imagenes = signal<ProductoImagen[]>([]);
```

## "Guardar cambios" condicional por tab

```html
@if (tabActivo() === 'producto' || tabActivo() === 'catalogo') {
  <button class="btn-primary" (click)="guardarCatalogo()">Guardar cambios</button>
}
```

En tabs Imágenes y Distribución el botón desaparece — no hay datos de formulario que guardar.

## Toast notifications en acciones del modal

| Método | Toast éxito | Toast error |
|--------|-------------|-------------|
| `guardarCatalogo()` | `"[nombre] actualizado correctamente."` | `"No se pudo guardar el producto."` |
| `subirImagen()` | `"Imagen subida correctamente."` | `"No se pudo subir la imagen."` |
| `imgMarcarPrincipal()` | `"Imagen principal actualizada."` | `"No se pudo cambiar la imagen principal."` |
| `imgEliminar()` | `"Imagen eliminada."` | `"No se pudo eliminar la imagen."` |

## Fix: z-index toast por debajo del modal

`cdk-overlay-container` tiene `z-index: 1000` por defecto — queda detrás del overlay custom (`z-index: 9000`).

```css
/* styles.css — global */
.cdk-overlay-container { z-index: 10000 !important; }
```

## Funcionalidad eliminar producto

```typescript
confirmarEliminar(producto: Producto): void {
  const ref = this.dialog.open(ConfirmDialogComponent, {
    data: { titulo: 'Eliminar producto', mensaje: `¿Eliminar "${producto.nombre}"?`, textoConfirmar: 'Eliminar', tipo: 'peligro' }
  });
  ref.afterClosed().subscribe(confirmado => { if (confirmado) this.eliminar(producto); });
}

private eliminar(producto: Producto): void {
  this.servicio.eliminar(producto.id).subscribe({
    next: () => { this.productos.update(lista => lista.filter(p => p.id !== producto.id)); this.snack.exito(`${producto.nombre} fue eliminado.`); },
    error: () => this.snack.error('No se pudo eliminar el producto.'),
  });
}
```

## Modal "Crear nuevo producto" — SKU automático con Subject+debounce

```typescript
private readonly nombreCrearChange$ = new Subject<string>();

// En constructor:
this.nombreCrearChange$.pipe(
  debounceTime(500), distinctUntilChanged(),
  switchMap(nombre => {
    if (!nombre || nombre.trim().length < 2 || this.crearSkuManual()) return EMPTY;
    this.crearSugieriendoSku.set(true);
    return this.servicioEtiquetas.sugerirSku(nombre.trim());
  }),
  takeUntilDestroyed(this.destruirRef)
).subscribe({ next: (res) => { this.crearSku.set(res.sku); ... } });
```

Signals nuevas: `crearVisible`, `crearNombre`, `crearSkuManual`, `crearSku`, `crearStockMinimo`, `crearMarcaId`, `crearEtiquetaIds`, `crearDescripcion`, `crearError`, `creando`, `crearSugieriendoSku`.

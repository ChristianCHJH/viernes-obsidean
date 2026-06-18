---
titulo: Venta-Inventario — Variantes, Categorías y Atributos
tipo: proyecto
tags: [venta-inventario, variantes, categorias, atributos, inventario, catalogo]
fecha_creacion: 2026-06-17
fecha_actualizacion: 2026-06-17
estado: activo
---

# Variantes, Categorías y Atributos

Sub-página de [[venta-inventario]] / [[venta-inventario-inventario]]. Documenta el sistema de **clasificación de productos por variante** (Talla, Color) y atributos descriptivos. Implementado en la app principal (NestJS + Angular); el catálogo público lo consume desde 2026-06-17 (ver [[venta-inventario-catalogo]]).

## Modelo conceptual

```
Categoría (árbol global)  →  sugiere  →  Atributo Tipo  →  tiene  →  Atributo Valor
       │                                     │
   Producto ── tiene 1 ──┘            genera_variante?
       │                              ├── true  (Talla, Color)  → genera VARIANTES
       └── tiene N ── Variante         └── false (Material, Género, Temporada) → descriptivos
                         │                        (solo para filtros de catálogo)
                    combinación de Atributo Valor (Polo M-Blanco)
                         │
                    stock_variante (cantidad por variante)
```

- **Atributo Tipo** tiene dos flags clave: `genera_variante` (true = Talla/Color → producen variantes; false = Material/Género/Temporada → solo descriptivos) y `afecta_imagen` (true solo en Color → swatch / imagen por color).
- **Variante** = producto cartesiano de los valores generadores seleccionados. SKU autogenerado (`{sku-base}-{iniciales}`, ej. `POLO-M-BL`), editable.
- **1 producto = 1 categoría** (hoja del árbol). Categoría sugiere qué atributos aplican vía `categoria_atributo_tipo`.

## Tablas (10)

Migraciones `026-categorias-atributos.sql` y `031-variantes-stock.sql`:

| Tabla | Rol |
|-------|-----|
| `categoria` | Árbol jerárquico global (Ropa, Calzado, Otros + hijos), `padre_id` |
| `atributo_tipo` | Tipos globales; flags `genera_variante`, `afecta_imagen` |
| `atributo_valor` | Valores (S, M, 41, #EF4444…) con `etiqueta` y `orden` |
| `categoria_atributo_tipo` | Qué atributos sugiere cada categoría |
| `producto_categoria` | Asignación 1:1 producto → categoría |
| `producto_variante` | Una variante (combinación), `sku_variante`, `estado` |
| `producto_variante_valor` | Valores que tiene cada variante (link variante ↔ valor) |
| `producto_atributo_descriptivo` | Descriptivos (Material/Género/Temporada) — no generan variante |
| `producto_color_imagen` | Imagen por color (opcional) |
| `stock_variante` | Stock total por variante |

## Backend

- **`VariantesModulo`** — base `inventario/variantes`. Endpoints: `GET /producto/:id` (configuración), `POST /producto/:id/generar` (producto cartesiano), `PUT .../categoria`, `PUT .../descriptivos`, `PATCH/:id`, `DELETE/:id`. Permisos `INVENTARIO_VARIANTES_VER/GESTIONAR/STOCK`.
- **`AtributosModulo`** — base `inventario/atributos`. CRUD global de tipos y valores. Permisos `INVENTARIO_ATRIBUTOS_*`.
- Servicio clave: `generarVariantes()` calcula el cartesiano, reactiva variantes desactivadas y hace soft-delete de las combinaciones que ya no aplican.

## Frontend (app interna)

- Tab **"Variantes"** en `productos-premium.component.ts`: selector visual de tallas/colores → generación automática, edición de SKU/estado, asignación de categoría.
- Servicios: `VariantesInventarioServicio`, `AtributosInventarioServicio`.

## Catálogo público (consumo)

Desde 2026-06-17 el endpoint `/pub/{slug}/productos` incrusta `atributos[]` por producto (solo `genera_variante=true`) vía un `DISTINCT` sobre `producto_variante_valor`. El frontend deriva las facetas client-side (`lib/facetas.ts`) y muestra un **filtro de variantes en sidebar izquierdo** en ambas plantillas. Detalle en [[venta-inventario-catalogo]].

## Decisiones clave

1. Un producto = una sola categoría (hoja), no multi-categoría.
2. SKU de variante autogenerado por el sistema, editable.
3. Descriptivos NO generan variantes; reservados para filtros (categoría/navbar en el catálogo, pendiente).
4. Variantes opcionales: productos planos (sin variantes) coexisten con los que sí las tienen.
5. El filtro del catálogo muestra **todos los valores configurados**, sin filtrar por stock.

## Referencia

- Plan arquitectónico: `docs/planes/plan-variantes-categorias.md` (689 líneas) en el repo.

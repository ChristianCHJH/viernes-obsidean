---
titulo: Venta-Inventario — Categorías y Variantes de Producto
tipo: proyecto
tags: [inventario, variantes, categorias, catalogo, saas, adifnex, plan]
fecha_creacion: 2026-06-11
fecha_actualizacion: 2026-06-12
estado: en-construccion
---

# Categorías Jerárquicas y Variantes de Producto

Plan de arquitectura para clasificar productos (árbol de categorías) y manejar variantes (talla/color con stock e imagen propios). Objetivo: alimentar la clasificación y los filtros del **catálogo público** y sentar base para un **carrito futuro**. Diseñado global (multi-negocio SaaS).

**Estado:** 🔨 EN CONSTRUCCIÓN — Fase 1 completa, modelo cerrado, **Refactor A ✅**, Refactor B en turno, Fase 2 pendiente (ver [§ Estado de construcción](#estado-de-construcción--fases)).

**Documentos fuente:** `docs/planes/plan-variantes-categorias.{md,html}` (plan completo, sincronizados) · `docs/planes/handoff-construccion.md` (guía para el agente constructor) · `docs/planes/arbol-categorias-propuesta.md` (formato del árbol, a poblar con data real del usuario).

Relacionado: [[venta-inventario-catalogo]] · [[venta-inventario-inventario]] · hub [[venta-inventario]]

---

## Concepto central: dos clases de atributo

- **Generadores** (`genera_variante = true`): **Talla** y **Color**. Su combinación produce variantes con stock e imagen propios.
- **Descriptivos** (`genera_variante = false`): **Material, Género, Temporada** (y más por rubro). No crean variantes — son **facetas de filtrado** del catálogo.

Todo es **global del sistema** (seed). El negocio solo **elige** qué valores aplica, no los crea → evita 50 formas de escribir "XL".

### Atributos SEPARADOS POR CONTEXTO (clave)

No existe un "Talla" ni un "Material" único. Se separan por contexto, y cada categoría sugiere los suyos vía `categoria_atributo_tipo`. Así el filtro del catálogo muestra solo data relevante por categoría (a una crema no le sale "Algodón"; a un sofá no le sale "Talla M").

- Tallas: `talla-ropa-adulto`, `talla-ropa-nino`, `talla-bebe`, `talla-calzado-adulto`, `talla-calzado-nino`
- Materiales: `material-textil` (Algodón/Poliéster…), `material-mueble` (Madera/Metal/Vidrio…), `material-juguete` (Plástico/Madera/Peluche…)
- Otros descriptivos por rubro: `genero`, `temporada`, `edad-recomendada` (juguetes), `tipo-piel` (belleza), `etapa-vida` (mascotas)

Es el mismo patrón que ya se usó con las tallas: distintos contextos = atributos distintos con su propia data.

> **Dimensión** quedó **fuera del plan** (no encaja como variante ni faceta).
> **Seed de descriptivos:** se carga **al final** (junto con / tras el árbol multi-rubro). El modelo ya lo soporta; solo falta poblar valores.

---

## Modelo de datos — 9 tablas nuevas

Ninguna tabla existente se modifica estructuralmente (salvo la remoción de `orden_catalogo`, ver abajo). Coexistencia: productos sin variantes siguen usando `stock_sede`.

| Tabla | Rol |
|---|---|
| `categoria` | Árbol jerárquico global (`padre_id` nullable, `slug`, `orden`) |
| `atributo_tipo` | Tipos (Talla, Color, Material…) con flags `genera_variante` + `afecta_imagen` |
| `atributo_valor` | Valores seed (XS, M, #EF4444, Algodón…) |
| `categoria_atributo_tipo` | Qué atributos sugiere cada categoría |
| `producto_categoria` | Categoría del producto (1 sola, hoja) |
| `producto_atributo_descriptivo` | Facetas del producto (Material/Género/Temporada) |
| `producto_variante` | Combinación generadora; `sku_variante` (sin `precio_override` — precio único) |
| `producto_variante_valor` | Qué valores (Talla/Color) tiene cada variante |
| `producto_color_imagen` | 1 imagen por color (override sobre la galería general) |
| `stock_variante` | Stock por variante a nivel **negocio** (sin sede; sede = fase carrito) |

Stock por variante reemplaza la imprecisión de `stock_sede` (sabe si quedan M o XL).

---

## Decisiones del modelo — TODAS resueltas (2026-06-11)

Sesión completa de resolución de dudas. Modelo cerrado, sin ambigüedad.

| Tema | Decisión |
|---|---|
| Categorías por producto | **1 sola** (hoja); categoría **opcional** al crear (sin ella = producto simple) |
| Negocio crea categorías | Sí, subcategorías propias (`negocio_id` nullable) |
| Valores custom por negocio | **No** — solo seed global; SUPERADMIN amplía |
| Facetas descriptivas | Material/Género/Temporada — **múltiples** por producto |
| Ejes de variante | Talla × Color (categoría **sugiere** tallas, editable; sin categoría se elige a mano) |
| Generación de combinaciones | **Automática** + el negocio quita las que no maneja |
| SKU de variante | Auto-generado editable |
| **Precio** | **Único global** por producto — NO varía por talla/color. **Se elimina `precio_override`** |
| **Descuento** | Único por producto, **vigencia obligatoria máx 15 días**; expira solo → precio normal. Badge solo si vigente |
| **Stock variante** | **Total del negocio** (tabla `stock_variante`, sin sede). Excluyente: producto con variantes = 100% por variante. Total = suma de variantes |
| Stock al activar variantes | Se **reingresa por variante** |
| Movimientos | **Condicional** — exigen variante solo si el producto la tiene |
| Sede de variantes + venta por sede | **Fase futura** (carrito) |
| Imagen | Galería general + **override por color** (color sin foto → principal de galería) |
| Catálogo: vista | Un producto con **selector** (no items separados); agotado total = **visible "Agotado"** |
| Catálogo UI variantes | **Fase aparte** (pospuesta); carrito futuro → variante = unidad de venta |
| **Etiquetas** | **Se eliminan** (tabla, módulo, UI, permisos) — categorías+facetas+marcas las reemplazan |
| **Marcas** | Pasan a **globales** (seed) + máx **3 propias** por negocio; globales read-only. Existentes se conservan |
| `orden_catalogo` | Se elimina → reemplazo por flag `destacado` |

---

## Catálogo público — reparto backend vs `catalogo-react`

- **Backend (este repo):** modelo + endpoints. Árbol filtrado por negocio (`WITH RECURSIVE`, solo ramas con productos visibles), facetas disponibles, producto con variantes + stock **a nivel negocio** (`stock_variante`, sin sede) + imágenes por color + breadcrumb.
- **Catálogo (`catalogo-react`, Next.js):** UI. Menú de categorías, selector talla/color, **tachado de agotados** (stock 0 → deshabilitado, no oculto), **selección dependiente** (color elegido habilita solo tallas con stock), cambio de imagen, precio que cambia con la variante.

Endpoints nuevos/extender: `GET /pub/:slug/categorias` (árbol filtrado), `/productos?categoria=&talla=&color=&material=&genero=&temporada=`, `/facetas?categoria=` (nuevo), `/producto/:id` (variantes + stock agregado + breadcrumb).

---

## Remoción de `orden_catalogo` → `destacado`

Con navegación por categorías, el orden manual numérico pierde sentido. Se reemplaza por flag **`destacado`** + orden automático (`ORDER BY destacado DESC, fecha_creacion DESC`).

- **BD:** `DROP COLUMN orden_catalogo` + `ADD COLUMN destacado BOOLEAN DEFAULT false` → actualizar `setup-completo.sql`.
- **Backend:** ajustar `publico.servicio.ts`, quitar `ordenCatalogo` de entidad/DTOs.
- **Frontend Angular:** quitar control de orden en "Mi Página", agregar toggle Destacar.
- **Catálogo Next.js:** eliminar usos de `ordenCatalogo`.

Migración independiente y acotada — no bloquea el sprint de variantes.

---

## Previsión carrito futuro (no se construye aún)

Venta por variante → `carrito_item` / `pedido_item` apuntarán a `variante_id`. Productos simples necesitarán una **variante por defecto** (camino A: variante única implícita / camino B: pedido acepta `producto_id` o `variante_id`) — a decidir cuando se construya. Refuerza la decisión de coexistencia. Sin tablas nuevas ahora.

---

## Módulos y permisos

- **Módulos:** `CategoriasModulo` ✅, `AtributosModulo` ✅ (construidos Fase 1), `VariantesModulo` ⏳ (Fase 2).
- **Permisos reales seed (migración 026):** sección `INVENTARIO_CATEGORIAS` (VER/CREAR/EDITAR/ELIMINAR) + sección `INVENTARIO_ATRIBUTOS` (VER/CREAR/EDITAR/ELIMINAR). Fase 2 añade `INVENTARIO_VARIANTES_*`.
- **Refactorizados:** Marcas (→ globales + 3 propias), Productos (descuento vigencia + destacado).
- **Eliminado:** Etiquetas (módulo, tabla, permisos `INVENTARIO_ETIQUETAS_*`).

---

## Árbol de categorías multi-rubro (a poblar con data real)

- **Estructura: 3 niveles consistentes** (estilo Ripley): rubro → sub → hoja. El producto se asigna a la **hoja (nivel 3)** y queda enlazado a nivel 2 y 1 vía `padre_id` (breadcrumb).
- **Alcance: multi-rubro amplio** (Tecnología, Mujer, Hombre, Moda Infantil, Calzado, Deporte, Belleza, Hogar, Electrohogar, Mascotas, Juguetería, Accesorios…), no solo ropa/calzado.
- **Origen de la data:** Christian la entregará (capturas/texto de Ripley u otra fuente). Viernes/Claude solo **estandariza**: normaliza a 3 niveles, genera slugs únicos, asigna atributos sugeridos por hoja. **No inventar el árbol.**
- Las columnas "Marcas" de las capturas de Ripley NO son categorías → van al seed de marcas. Los "filtros especiales" (Outlet, Lo nuevo) tampoco.
- Formato destino: `docs/planes/arbol-categorias-propuesta.md`.

---

## Estado de construcción — Fases

> 🔨 **Construcción ACTIVA.** Al despertar, recordar que estamos en este proceso y por dónde vamos.

| # | Tarea | Estado |
|---|-------|--------|
| **Fase 1** | Backend Categorías + Atributos (migración 026, módulos, 21 tests verde) | ✅ COMPLETA |
| **Modelo de negocio** | Todas las decisiones resueltas + plan/wiki sincronizados | ✅ CERRADO |
| Refactor A | Eliminar etiquetas (migración 027 · tsc limpio · frontend + backend + DB limpio) | ✅ COMPLETO |
| Refactor B | Marcas → globales + máx 3 propias (+ seed marcas) — **▶ SIGUIENTE** | ⏳ pendiente |
| Refactor C | Descuento con vigencia + `orden_catalogo→destacado` | ⏳ pendiente |
| **Fase 2** | Variantes + stock por variante (`VariantesModulo` + 5 tablas) | ⏳ pendiente |
| Árbol categorías | Seed multi-rubro 3 niveles | ⏸️ esperando data del usuario |
| Seed descriptivos | Valores por contexto | ⏸️ al final |
| Catálogo UI variantes | Selector talla/color público (`catalogo-react`) | ⏸️ fase aparte pospuesta |

La construcción la ejecuta el agente Sonnet siguiendo `docs/planes/handoff-construccion.md`.

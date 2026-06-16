---
titulo: Frontend — Design System y CSS Global
tipo: proyecto
tags: [inventario, angular, design-system, adifnex, sidebar, tailwind, css]
fecha_creacion: 2026-06-07
fecha_actualizacion: 2026-06-16
estado: activo
---

# Design System y CSS Global

Sub-página de [[venta-inventario-frontend]].

## Tokens `--p-*` globalizados (sesión 39)

La paleta Adifnex que vivía scoped en `productos-premium` fue promovida a variables globales.

**`src/styles.css`** — nuevo bloque en `:root {}`:
```css
--p: #14532D;  --p-accent: #EA580C;  --p-emerald: #10B981;
--p-amber: #F59E0B;  --p-sky: #0EA5E9;  --p-rose: #F43F5E;
--p-bg: #F0FDF4;  --p-surface: #FFFFFF;  --p-text: #111827;
--p-muted: #6B7280;  --p-border: #D1FAE5;
--p-shadow: 0 2px 12px rgba(20,83,45,.08);
```

**`tailwind.config.js`** — colores `bg-p-primary`, `text-p-muted`, `border-p-border`, etc.

**Reglas de reuso**:
- CSS de componente → `var(--p)`, `var(--p-bg)`, etc.
- Template HTML → clases Tailwind `bg-p-primary`, `text-p-muted`
- No hardcodear `#14532D` — siempre por token
- `--p-shadow` solo disponible como CSS var (no hay clase Tailwind para `box-shadow` completo)

**Coexistencia con `--spa-*`**: los tokens `--spa-*` (azul `#2b7ea6`, crema `#faf8f6`) siguen activos para shell (modales globales, sidebar legacy, botones `.primary-btn`). Para componentes nuevos de inventario usar `--p-*`.

## Color naranja `#EA580C` — integración (sesión 42)

| Lugar | Elemento | Valor |
|-------|---------|-------|
| `barra-lateral` | Brand mark, avatar perfil, workspace icon | gradiente `#14532D → #EA580C` |
| `productos-premium` | Botón CTA "Nuevo producto" | gradiente `#EA580C → #C2410C` |
| `productos-premium` | Summary badge (eliminado sesión 44) | tint naranja |

**Principio**: naranja para identidad de marca (logo, avatares) y CTAs que requieren atención. No usar en tabla thead, items nav activos ni estados de stock.

## Truco: negative margins en `:host`

Cuando un componente necesita un fondo distinto al del panel, cancela y re-aplica el padding:

```css
:host {
  background: var(--mi-fondo);
  margin: -1.5rem -3rem -2.5rem;
  padding: 1.5rem 3rem 2.5rem;
  box-sizing: border-box;
}
@media (max-width: 1023px) { :host { margin: -1.25rem -2rem -2rem; padding: 1.25rem 2rem 2rem; } }
@media (max-width: 640px)  { :host { margin: -1rem -1.25rem -1.5rem; padding: 1rem 1.25rem 1.5rem; } }
```

**Cuándo NO usar**: vistas con tablas full-height. Para esas, usar el patrón de 3 pasos (ver sección siguiente).

## Regla estándar: tablas full-height sin scroll de página (sesión 39)

Toda vista con tabla debe ocupar el máximo de pantalla. La página nunca scrollea — el scroll va en `.table-scroll`.

### 3 pasos obligatorios

**Paso 1** — `panel.component.html`:
```html
<div class="view-area" [class.view-NUEVA_VISTA]="vistaActiva === 'NUEVA_VISTA'">
```

**Paso 2** — `panel.component.css`:
```css
.view-area.view-NUEVA_VISTA { overflow: hidden; display: flex; flex-direction: column; padding: 0; }
```

**Paso 3** — componente `:host`:
```css
:host { flex: 1; display: flex; flex-direction: column; min-height: 0; overflow: hidden; padding: 1.5rem 3rem 2.5rem; box-sizing: border-box; }
```

### Cadena flex obligatoria

```
view-area.view-VISTA (flex:1, overflow:hidden, flex column, padding:0)
  → :host (flex:1, overflow:hidden, flex column, padding propio)
    → .shell (flex:1, min-height:0, flex column)
      → KPIs / toolbar (flex-shrink:0)
      → .data-card (flex:1, min-height:0, overflow:hidden)
        → .table-wrapper (flex:1, min-height:0, overflow:hidden)
          → .table-scroll (flex:1, min-height:0, overflow-y:auto) ← scroll aquí
```

Implementado en `productos-premium` (referencia canónica) y `sedes-lista`.

## Patrón full-height split-pane (form + preview) (2026-06-16)

Extensión del patrón tablas full-height para vistas que tienen **formulario izquierda + preview derecha** (ej: "Mi Página").

Diferencia clave vs tablas: en lugar de `.table-scroll`, el lado derecho usa un **browser mockup** que también crece con `flex:1`.

```css
/* Mismos 3 pasos del panel (view-mi-pagina) */
.view-area.view-mi-pagina { overflow:hidden; display:flex; flex-direction:column; padding:0; }

/* En el componente */
:host { flex:1; display:flex; flex-direction:column; min-height:0; overflow:hidden; }
.shell  { flex:1; display:flex; flex-direction:column; min-height:0; }
.body   { flex:1; min-height:0; display:flex; overflow:hidden; }  /* ← row */
.form-col   { min-width:340px; width:38%; max-width:440px; flex-shrink:0; /* column scroll */ }
.preview-col { flex:1; min-height:0; overflow:hidden; display:flex; flex-direction:column; }
.browser-wrap { flex:1; min-height:0; display:flex; flex-direction:column; }
.browser-frame { flex:1; display:flex; flex-direction:column; overflow:hidden; }
/* Elemento que "empuja" al footer hacia abajo dentro del browser */
.preview-content-area { flex:1; }
```

**Gotcha — espacio vacío a los lados en el preview**: si dentro de `.preview-col` (que es `flex:1` y ocupa ~930px) pones un `.preview-col-inner` con `max-width: 480px; margin: 0 auto`, quedan ~225px vacíos a cada lado visibles como cajas blancas. Fix: **eliminar el max-width**, dejar que el inner también sea `flex:1` con `flex-direction:column`. El contenido interno escala al ancho disponible.

**Regla**: nunca centrar un contenedor con `max-width + margin: auto` dentro de una columna flex que ya tiene `flex:1` — a menos que quieras el efecto de centrado explícito y las cajas vacías sean aceptables.

## CSS budget de Angular superado (sesión 39)

**Síntoma**: `npm run build` falla con `exceeded maximum budget. Budget 24.58 kB was not met`.

```json
// angular.json — sección budgets de producción
{ "type": "anyComponentStyle", "maximumWarning": "40kB", "maximumError": "64kB" }
```

Solo aumentar para componentes complejos (modal multi-tabs, design system inline, animaciones).

## Top bar del panel — acento de marca (sesión 32)

```css
/* panel.component.css */
.page-header { border-bottom: 2px solid #14532D; /* antes: 1px solid #ebe6e1 */ }
```

Fondo del `page-header` se mantiene blanco — es chrome global compartido. El acento es solo la línea inferior.

## Sidebar v2 — blanco/verde Adifnex (sesión 41→43)

**Historial**: construido como `barra-lateral-v2` (sesión 41), promovido a `barra-lateral/` y sidebar v1 (dark navy) eliminado (sesión 43).

| Aspecto | v1 (dark navy) | v2 (actual) |
|---------|--------------|------------|
| Fondo | `#0f2030` | `#FFFFFF` blanco |
| Item activo | Fondo azul translúcido | `#F0FDF4` + barra izquierda emerald `3px` |
| Brand mark | Círculo flat azul | Cuadrado `border-radius:10px`, gradiente `#14532D → #EA580C` |
| Context selector | `mat-form-field` + badge | Workspace switcher custom dropdown |

## Workspace switcher — dropdown custom

**Problema**: `mat-select` abre overlay fuera del sidebar, sin styling controlable en layout angosto.

**Solución**: dropdown 100% custom con `@if (wsDropdownAbierto())`:
- `position: absolute; z-index: 200` dentro del sidebar
- Cada opción: avatar inicial (gradiente verde) + nombre `text-overflow: ellipsis`
- Check `✓` emerald en opción activa; chevron animado (rotate 180° al abrir)
- `@HostListener('document:click')` cierra al click fuera; `@HostListener('document:keydown.escape')` también cierra

**Regla**: en sidebars angostos, evitar `mat-select`. Usar dropdown custom con `position: absolute` + `max-height + overflow-y: auto`.

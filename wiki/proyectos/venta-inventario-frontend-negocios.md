---
titulo: Frontend — Negocios Lista (diseño premium)
tipo: proyecto
tags: [inventario, angular, negocios, premium-refactor, design-system, adifnex]
fecha_creacion: 2026-06-17
fecha_actualizacion: 2026-06-17
estado: activo
---

# Negocios Lista — Rediseño Premium

Sub-página de [[venta-inventario-frontend]].

Componente: `frontend/src/app/features/negocios/negocios-lista/`
Ruta: `/dashboard/negocios`

## Historial de rediseño

- **v0 (original)**: tabla simple con estilos `--spa-*` (azul/crema). Sin KPIs.
- **v1 (2026-06-17)**: rediseño premium con `/premium-refactor`. Diseño idéntico a `productos-premium` — paleta verde mint, KPIs, gradiente tabla, mobile rows.

El rediseño pasó por fase de preview (`negocios-lista-v2/`) aprobada por Christian y luego aplicada al original (Paso 2 del skill).

## Stack de diseño

Usa los tokens `--p-*` (paleta Adifnex verde) idénticos a `productos-premium` y `sedes-lista`. Ver [[venta-inventario-frontend-diseno]] para lista completa de tokens.

## KPI grid (4 tarjetas)

| Tarjeta | Ícono | Gradiente | Computed signal |
|---------|-------|-----------|-----------------|
| Total negocios | `business` | `#14532D → #166534` (dark green) | `totalNegocios()` |
| Activos | `check_circle` | `#10B981 → #059669` (emerald) | `negociosActivos()` |
| Inactivos | `block` | `#F43F5E → #E11D48` (rose) | `negociosInactivos()` |
| Sin expiración | `all_inclusive` | `#0EA5E9 → #0284C7` (sky) | `sinExpiracion()` |

Signals computados agregados al TS (no estaban en el original):
```typescript
readonly negociosActivos   = computed(() => this.negocios().filter(n => n.estado).length);
readonly negociosInactivos = computed(() => this.negocios().filter(n => !n.estado).length);
readonly sinExpiracion     = computed(() => this.negocios().filter(n => !n.fechaVencimiento).length);
avatarLetra(nombre: string): string { return (nombre?.[0] ?? 'N').toUpperCase(); }
```

## Tabla desktop

- Header: `linear-gradient(90deg, #14532D, #166534)` — texto blanco
- Columnas: ID (pill muted), Nombre (avatar inicial + nombre), RUC, Correo, Plan (nivel-badge), Usuarios, Sedes, Hero, Vencimiento, Estado (status-badge), Acciones
- Hover: `background: rgba(20,83,45,.04)` + inset left border verde `3px`
- Status badge: activo con `pulse-dot` animado, inactivo estático

## nivel-badge — colores por nivel

```css
.nivel-1 { background: #e0f2fe; color: #0369a1; }  /* sky */
.nivel-2 { background: #d1fae5; color: #065f46; }  /* emerald */
.nivel-3 { background: #fef3c7; color: #92400e; }  /* amber */
.nivel-4 { background: #ede9fe; color: #5b21b6; }  /* purple */
```

## Mobile rows (`.neg-row`)

Patrón idéntico a `prod-row` de productos-premium:
- Avatar inicial + nombre + nivel-badge + RUC
- Punto de estado (pulse verde / gris estático)
- Acciones alineadas a la derecha
- Border left verde `3px` para activos, gris para inactivos

## Responsive — por qué no usar Tailwind `md:hidden`

**Problema**: Tailwind JIT no escanea el archivo `.component.ts` del nuevo componente al ser creado (no está en el purge list pre-existente). Las clases `hidden md:block` / `block md:hidden` simplemente no generan CSS.

**Fix**: usar clases CSS custom con media query manual:
```css
.only-desktop { display: block; }
.only-mobile  { display: none; }
@media (max-width: 767px) {
  .only-desktop { display: none; }
  .only-mobile  { display: block; }
}
```

**Regla derivada**: para cualquier componente nuevo creado con `/premium-refactor` o similar, usar siempre media queries CSS custom para la alternancia desktop/mobile. No confiar en Tailwind JIT para clases responsive en archivos nuevos.

## Panel: patrón `view-mint`

Este componente usa el patrón `view-mint` (no `view-productos`):
- Permite scroll de página (la lista puede ser larga)
- El `view-area` aporta el fondo `#F0FDF4`
- El `:host` aporta el padding `1.5rem 3rem 2.5rem`

Ver [[venta-inventario-frontend-diseno#patrón-view-mint]] para detalle completo y la regla anti-doble-fondo.

## Acciones y permisos

| Botón | Permiso | Acción |
|-------|---------|--------|
| Nuevo negocio | `NEGOCIOS_CREAR` | Abre modal formulario |
| Escudo (permisos especiales) | `NEGOCIOS_EDITAR` | Emite `@Output() verPermisos` |
| Editar | `NEGOCIOS_EDITAR` | Abre modal en modo editar |
| Desactivar | `NEGOCIOS_CAMBIAR_ESTADO` | Confirma y llama `cambiarEstado(id, false)` |
| Activar | `NEGOCIOS_CAMBIAR_ESTADO` | Confirma y llama `cambiarEstado(id, true)` |
| Eliminar | `NEGOCIOS_ELIMINAR` | Confirma y llama soft delete |

## Modal formulario

Campos: nombre, RUC (opcional), correo (opcional), nivel de plan (select 1-4), max usuarios, max sedes, max marcas, max productos, max imágenes hero, fecha vencimiento (vacío = sin expiración).

Modal propio con overlay, no `MatDialog`. Animación `modal-in` (translateY + scale desde 0.98).

# Log de Viernes

Registro cronológico de operaciones. Formato de entrada: `## [YYYY-MM-DD] tipo | descripción`

Tipos: `init` | `ingest` | `query` | `lint` | `update`

## [2026-06-07] restructure | venta-inventario-frontend.md dividido en 11 sub-páginas atómicas — 2217 líneas → ~100 líneas/archivo

- Hub reemplazado con stack, estado actual y WikiLinks
- 10 sub-páginas nuevas: migracion-material, patrones, permisos-rbac, gotchas, productos, registro, productos-premium, productos-premium-ux, diseno, sedes
- index.md actualizado: 10 nuevas entradas, total páginas 27 → 37

## [2026-06-07] ingest | sesión 46: sede-stock → drawer, redesign premium, toggle custom + select nativo, SALIDA como 3er tipo

- `venta-inventario-frontend.md`: secciones nuevas:
  - **sede-stock convertido a drawer**: eliminada ruta standalone, panel limpio de SedeStockComponent. Drawer `position:fixed; right:0` con backdrop blur, dot-pattern background, header gradiente verde, animación slide-in
  - **`@Input() enModal`**: oculta page-header, muestra KPI strip (productos/unidades/stock-bajo)
  - **Toggle custom 3 tipos**: INGRESO (verde) / SALIDA (rojo) / TRANSFERENCIA (azul). Pill `#F1F5F9` + white card activo
  - **Select nativo**: `[ngValue]` + `SelectControlValueAccessor`. Caret `expand_more` que rota 180° al focus
  - **SALIDA expuesta**: ya existía en backend, faltaba en UI. `stockMaximoSeleccionado` aplica también a SALIDA
- `venta-inventario-testing.md`: sprint 2026-06-07 con 9 unit tests + 5 E2E pendientes (`e2e/tests/10-sedes/sedes-stock-drawer.spec.ts`)

## [2026-06-07] ingest | sesión 45: sedes-lista — rediseño premium completo con design system Adifnex

- `venta-inventario-frontend.md`: sección nueva "sedes-lista — rediseño premium completo (sesión 45)"
  - KPI bar 4 cards (verde/emerald/slate/violet), avatar por hash, modal con bottom sheet mobile
  - Signals: `sedesFiltradas`, `sedesOrdenadas`, `sedesPaginadas`, `totalActivas`, `totalConDireccion`
  - Panel: `view-sedes` ya combinado con `view-productos` en la regla CSS de full-height
  - Permisos: `INVENTARIO_SEDES_CREAR/EDITAR/ELIMINAR`, `INVENTARIO_STOCK_VER`
- `venta-inventario-testing.md`: sprint 2026-06-07 con 7 unit tests + 3 E2E pendientes

## [2026-06-07] ingest | sesión 44: compact mobile cards, paginación fix, redundancy removal

- `venta-inventario-frontend.md`: tres secciones nuevas
  - **Mobile card redesign** (`prod-row`): 4 filas → 2 líneas flex, ~76px/item, 4-5 visibles por pantalla
    - Avatar 28px con gradiente, dot animado de estado, SKU/precio/marca inline, chips tiny
    - Gotcha documentado: CSS Grid con `grid-row: span` rompe auto-placement — usar flexbox
  - **Eliminación summary-badge**: "≡ N productos" en toolbar era redundante con KPI card
  - **Fix paginación mobile**: `.mobile-list` + `.mobile-footer` pattern (mismo que `.table-scroll` + footer desktop)
- No hay tests nuevos en esta sesión (cambios puramente de presentación)

## [2026-06-06] ingest | /viernes — sesión 43: sidebar v2 promovido a definitivo — barra-lateral-v2 eliminado

- `venta-inventario-frontend.md`: sección "Sidebar v2" actualizada
  - `barra-lateral-v2/` eliminado del repo
  - Código migrado a `barra-lateral/` — selector `spa-sidebar`, clase `BarraLateralComponent`
  - Dark navy sidebar (v1) eliminado permanentemente
  - Referencias `barra-lateral-v2` en tabla naranja y sección bug fix corregidas a `barra-lateral`
- No hay tests sugeridos en esta sesión (solo limpieza de código muerto)

## [2026-06-06] ingest | /viernes — sesión 42: naranja #EA580C activo en design system + bug fix dropdown perfil

- `venta-inventario-frontend.md`: sección nueva "Color naranja `#EA580C` — integración en el sistema"
  - Gradiente logo/avatar/ws-icon/ws-opt-avatar sidebar: `#14532D → #EA580C`
  - btn-primary productos-premium: `#EA580C → #C2410C`
  - summary-badge: tint naranja
  - `DESIGN.md` frontend actualizado: rol de `--p-accent` y gradientes nuevos
  - Principio: naranja = identidad de marca (logos) + CTA; verde = inventario/semántica
- `venta-inventario-frontend.md`: sección nueva "Bug fix — dropdown perfil barra-lateral-v2 no se abría"
  - Causa: `@HostListener('document:click')` cerraba el menú en el mismo tick que `alternarMenuCuenta()` lo abría
  - Fix: `$event.stopPropagation()` en el click del perfil (el ws-switcher ya lo tenía)
  - Regla derivada documentada: con `@HostListener('document:click')`, todos los openers deben hacer stopPropagation

## [2026-06-06] ingest | /viernes — sesión 41: barra-lateral-v2 (diseño blanco, workspace switcher custom)

- `venta-inventario-frontend.md`: nueva sección "Sidebar v2 — construcción en paralelo"
  - 3 archivos nuevos en `shared/components/barra-lateral-v2/`
  - Tabla comparativa v1 (dark navy) vs v2 (white/green)
  - Paleta: tokens `--p-*` de `productos-premium` (verde `#14532D`, emerald `#10B981`)
  - Workspace switcher: dropdown custom en lugar de `mat-select`
  - Gotcha documentado: `mat-select` en sidebars angostos — usar dropdown custom `position: absolute`
  - `@HostListener('document:click')` para cierre al click fuera
  - Activado en `panel.component` para pruebas; `barra-lateral` original intacta

## [2026-06-06] update | /viernes — sesión 40: gotcha budget CSS angular.json

- `venta-inventario-frontend.md`: nueva sección "Gotcha: CSS de componente supera budget de Angular"
  - `productos-premium.component.css` = 36.99 kB → excedía límite default 24 kB en `angular.json`
  - Fix: aumentar `anyComponentStyle` en `angular.json` → `maximumWarning: 40kB` / `maximumError: 64kB`
  - Regla documentada: aumentar budget solo en componentes complejos que lo justifican

## [2026-06-06] ingest | /viernes — sesión 39b: tokens --p-* globalizados + DESIGN.md sección 0

- `venta-inventario-frontend.md`: sección nueva "Tokens `--p-*` globalizados" — 3 archivos modificados + reglas de reuso + coexistencia `--spa-*`
- `frontend/src/styles.css`: bloque `--p-*` en `:root {}` — disponibles en toda la app
- `frontend/tailwind.config.js`: 11 colores `p-*` mapeados como clases Tailwind
- `frontend/DESIGN.md`: sección 0 nueva — paleta Adifnex completa, gradientes, colores movimiento semánticos, regla reuso
- `productos-premium.component.css`: `:host {}` conserva sus `--p-*` locales (linter los restauró — no rompe nada, coexisten con root)

## [2026-06-06] ingest | /viernes — sesión 39: regla estándar tablas full-height + fix productos-premium sin scroll de página

- `venta-inventario-frontend.md`: nueva sección "Regla estándar: tablas full-height sin scroll de página"
  - Patrón 3 pasos: `view-VISTA` class en panel HTML + CSS `overflow:hidden/flex/padding:0` + `:host` con `flex:1` sin negative margins
  - Cadena flex documentada completa
  - Negative margins: marcados como inaplicables para tablas full-height (solo extensión de fondo)
  - Referencia canónica: `productos-premium`
- `frontend/CLAUDE.md`: sección obligatoria "REGLA: Tablas full-height sin scroll de página" con código de referencia
- Memoria persistente: `feedback_tabla_fullheight.md` creado e indexado en MEMORY.md
- No hay tests sugeridos en esta sesión

## [2026-06-06] ingest | /viernes — sesión 38: reemplazo productos-lista → productos-premium + mejora skill viernes

- `venta-inventario-testing.md`: nuevo sprint 2026-06-06 — 2 specs marcados obsoletos (productos-lista y catalogo-producto-dialog eliminados); 3 E2E pendientes para productos-premium (verificar reemplazo visual, crear desde modal, distribución inline)
- `frontend/panel.component`: `@case ('productos')` ahora renderiza `<inv-productos-premium>`; `@case ('productos-premium')` y import `ProductosListaComponent` eliminados
- Carpeta `productos-lista/` eliminada (7 archivos incluyendo `catalogo-producto-dialog/`)
- `C:\Users\Christian\.claude\commands\viernes.md` mejorado: nuevo paso 3 — capturar tests sugeridos de la conversación y almacenarlos en `*-testing.md` como sprint pendiente

## [2026-06-06] ingest | /viernes — documentación sesión 37 (productos-premium: crear modal, eliminar, toasts, z-index fix, UX refinements)

- `venta-inventario-frontend.md`: sección nueva "productos-premium — modal edición refinado y nuevas funcionalidades"
  - Preview column estirada (340→380px, img height 120→170px)
  - Botones "Distribución" y "Editar" eliminados de row-actions (desktop y mobile)
  - Modal "Crear nuevo producto" completo: signals puras, Subject debounce SKU, chips etiquetas, CSS `.crear-modal`
  - "Guardar cambios" condicional solo en tabs Producto/Catálogo — ocultado en Imágenes/Distribución
  - Click fuera del modal ya no cierra (eliminado del overlay cat-overlay y crear overlay)
  - Toast en 4 métodos: guardarCatalogo, subirImagen, imgMarcarPrincipal, imgEliminar
  - Fix z-index: `.cdk-overlay-container { z-index: 10000 !important }` en styles.css global
  - Funcionalidad eliminar: MatDialog + ConfirmDialogComponent, actualización local sin recarga

## [2026-06-06] ingest | /viernes — documentación sesión 36 (UX mobile modal productos-premium: 3 mejoras)

- `venta-inventario-frontend.md`: sección nueva "productos-premium — UX mobile modal edición"
  - Vista previa del catálogo → bottom sheet (signal `previewMobileVisible`, botón inline en tab Catálogo, animación slide-up)
  - Modal con altura fija `height: min(88vh, 720px)` — elimina encogimiento al cambiar tabs
  - Formulario "Registrar movimiento" → bottom sheet en mobile (signal `movModalMobileVisible`, trigger con badge verde, formulario duplicado con signals compartidos, auto-cierre tras éxito)
  - Causa raíz del scroll horizontal en distribución documentada: doble padding `.dist-layout` + `.cat-tab-content`
  - Regla: en modales con tabs de contenido variable → `height` fijo, no `max-height`
  - Regla: paneles secundarios de formularios en mobile → bottom sheet, no flujo apilado

## [2026-06-06] update | /viernes — documentación sesión 35 (2 fixes productos-premium: preview en distribución + contraste thead)

- `venta-inventario-frontend.md`: 2 fixes documentados en sección "Tab Distribución — Fixes post-implementación"
  - Fix 1: `@if (tabActivo() !== 'distribucion')` oculta `.cat-preview-col` del DOM en el tab distribución
  - Fix 2: `.dist-table th { color: rgba(255,255,255,.85) }` — cabecera legible sobre gradiente verde oscuro
  - Regla derivada: `thead tr` global aplica gradiente a todos los `thead` del componente — nuevas tablas deben sobreescribir `color` a blanco

## [2026-06-06] ingest | /viernes — documentación sesión 34 (tab Distribución en modal productos-premium: INGRESO+SALIDA+TRANSFERENCIA)

- `venta-inventario-frontend.md`: sección nueva "Tab Distribución en modal productos-premium"
  - 4° tab que expone los 3 tipos de movimiento: INGRESO (verde), SALIDA (rojo), TRANSFERENCIA (azul)
  - SALIDA ya existía en el backend pero nunca se expuso en UI — ahora visible
  - Layout full-width al abrir el tab (sin preview-col) — css `.cat-modal-body-full`
  - Grid 2 columnas: tabla stock por sede (con barras progreso) + formulario movimiento
  - Hint inline "Disponible: N uds" en sede para SALIDA y TRANSFERENCIA
  - Caché de sedes (una carga por apertura de modal)
  - Badge con total de stock en el botón del tab

## [2026-06-06] ingest | /viernes — documentación sesión 33 (modal unificado productos-premium: tabs Producto+Catálogo+Imágenes)

- `venta-inventario-frontend.md`: sección nueva "Modal unificado productos-premium — 3 tabs"
  - Tab Producto: nombre, SKU toggle manual/auto, stock mínimo, marca (native select estilizado), etiquetas (chips custom con `color-mix`)
  - Tab Catálogo: campos existentes sin cambios
  - Tab Imágenes: gallery grid 88×88, mark principal, delete, upload con `<label for>` pattern, sub-modal z-index 9100
  - Preview actualizado: muestra imagen real de `imagenPrincipal()`, fallback avatar de letra
  - Patrones: `forkJoin` con cache de listas, actualización local de signal tras guardar, PATCH unificado con todos los campos
- `fecha_actualizacion` implícita sesión 33

## [2026-06-05] ingest | /viernes — documentación sesión 32 (paleta Adifnex en productos-premium + top bar acento)

- `venta-inventario-frontend.md`: 3 secciones nuevas + 2 secciones actualizadas
  - Design system `productos-premium` actualizado: paleta indigo/violet → verde `#14532D` / naranja `#EA580C` / mint `#F0FDF4` (Adifnex brand)
  - Iteración de paleta documentada: se probó naranja como primario → descartado; verde como primario → versión final
  - Sección nueva: decisión top bar — fondo blanco conservado (chrome global), `border-bottom: 2px solid #14532D` como acento de marca
  - Sección nueva: eliminación header in-page `productos-premium` — eyebrow + h1 duplicaban el `tituloPagina()` del panel
  - Tabla diferencias `productos-lista` vs `productos-premium` actualizada
- `fecha_actualizacion` del archivo actualizada a 2026-06-05 (sesión 32)

## [2026-06-04] ingest | /viernes — documentación sesión 31 (UI/UX mejoras productos-lista + componente productos-premium)

- `venta-inventario-frontend.md`: 3 secciones nuevas añadidas
  - UI/UX mejoras productos-lista: gradiente botón, pill search, row hover accent, action buttons tipados por color, modal border-top accent, status dot
  - Componente `productos-premium`: pantalla nueva desde cero con design system indigo/violet (KPI bar, gradiente tabla, pulse-dot, avatares dinámicos, DM Sans)
  - Trick negative margins en `:host` para cubrir padding del panel (con breakpoints espejo)
- Tabla fases migración PrimeNG → Material: Fase 6 corregida de ⬜ a ✅ (estaba desfasada desde sesión 29)

## [2026-06-03] ingest | /viernes — documentación sesión 30 (gotchas Material Icons, CSS scoped, Docker, UX productos)

- `venta-inventario-frontend.md`: 5 secciones nuevas documentadas
  - Gotcha Material Icons vs Material Symbols — URLs distintas, `<mat-icon>` requiere la clásica
  - Gotcha CSS scoped Angular pisa Tailwind `md:hidden` — patrón correcto: `@media` explícito en CSS o `@if (esMovil)` en template
  - Gotcha `mat-icon` hereda `font-size` del botón padre — siempre definir en px con selector `button mat-icon`
  - Gotcha Docker: `@angular/material` no se instala si imagen no se reconstruye (`up --build`)
  - UX refactoring productos-lista: -2 columnas, chips limitados a 2+N, label Buscar eliminado, precio alineado derecha

## [2026-06-03] ingest | /viernes — documentación sesión 29 (Fase 6 Angular Material — mi-pagina)

- `venta-inventario-testing.md`: sprint Fase 6 añadido — 5 tests pendientes para `mi-pagina.component.spec.ts` (ConfirmDialogComponent vía MatDialog, snackBar.exito/error)
- `venta-inventario-frontend.md`: Fase 6 marcada ✅ (fecha actualizada). Migración PrimeNG → Angular Material **100% completa** (Fases 0-6)
- Patrón mock documentado: `DragDropModule` de CDK ya estaba en el componente — no genera cambios en mocks de tests

## [2026-06-03] ingest | Fase 6 migración Angular Material — mi-pagina catálogo (sesión 29)

- **Componente migrado**: `mi-pagina.component` — el más complejo del proyecto (split-pane, 5 tabs, drag-drop imágenes hero, preview vivo reactivo)
- **PrimeNG eliminado**: `MessageService`, `ConfirmationService`, `ToastModule`, `ButtonModule`, `InputTextModule`, `InputTextareaModule`, `InputSwitchModule`, `TabViewModule`, `ProgressSpinnerModule`, `SkeletonModule`, `TooltipModule`, `ConfirmDialogModule`; `providers: [MessageService, ConfirmationService]`
- **Material agregado**: `MatTabsModule`, `MatSlideToggleModule`, `MatIconModule`, `MatButtonModule`, `MatProgressSpinnerModule`, `MatTooltipModule`, `MatDialog` + `SnackBarServicio` + `ConfirmDialogComponent`
- **Mapa de iconos clave**: `pi-eye`→`visibility`, `pi-lock`→`lock`, `pi-arrows-alt`→`open_with`, `pi-star`→`star`, `pi-trash`→`delete`, `pi-spin pi-spinner`→`<mat-spinner>`, `pi-cloud-upload`→`cloud_upload`, `pi-check`→`check`, `pi-external-link`→`open_in_new`
- **Estado final**: todo el frontend compila sin errores PrimeNG. Fases 0-6 ✅. Migración completa.
- Actualizado: `venta-inventario-frontend.md` (Fase 6 ✅), `venta-inventario-testing.md` (sprint Fase 6)

## [2026-06-03] ingest | /viernes — documentación sesión 28 (Fase 5 Angular Material)

- `venta-inventario-testing.md`: sprint Fase 5 añadido — 4 tests pendientes (`marcas-lista`, `etiquetas-lista`, `productos-lista`, `catalogo-producto-dialog`)
- `venta-inventario-frontend.md`: Fase 5 marcada ✅; nota `secciones-permiso-lista` marcada como eliminada
- Ninguna página nueva creada — todo cabe en páginas existentes

## [2026-06-03] ingest | Fase 5 migración Angular Material — 7 componentes inventario (sesión 28)

- **Componentes migrados**: `marcas-lista`, `etiquetas-lista`, `sedes-lista`, `movimientos`, `sede-stock`, `producto-distribucion`, `productos-lista` + `catalogo-producto-dialog`
- **PrimeNG eliminado**: `DialogModule`, `ToastModule`, `ConfirmDialogModule`, `DropdownModule`, `SelectButtonModule`, `MultiSelectModule`, `InputNumberModule`, `InputSwitchModule`, `InputTextareaModule`, `MessageService`, `ConfirmationService`
- **Material agregado**: `MatDialogModule`, `MatButtonModule`, `MatIconModule`, `MatFormFieldModule`, `MatSelectModule`, `MatButtonToggleModule`, `MatSlideToggleModule`, `MatTooltipModule`, `MatProgressSpinnerModule`, `SnackBarServicio`, `ConfirmDialogComponent`
- **Eliminado**: `secciones-permiso-lista/` (legacy, sin referencias, sin ruta activa)
- **Estado**: solo `mi-pagina.component.ts` (Fase 6) tiene errores PrimeNG pendientes. Todo lo demás compila limpio.
- Actualizado: `venta-inventario-frontend.md` (Fase 5 marcada ✅)

## [2026-06-02] ingest | /viernes — documentación sesión 27 (Fase 4 Angular Material)

- `venta-inventario-testing.md`: sprint Fase 4 añadido — 2 tests pendientes (`permisos-lista` + `mi-negocio-vista`)
- `venta-inventario-frontend.md`: sección secciones-con-permisos-lista actualizada; nota sobre `secciones-permiso-lista` legacy con PrimeNG pendiente de eliminar
- Ninguna página nueva creada — todo cabe en páginas existentes

## [2026-06-02] ingest | Fase 4 migración Angular Material — 10 componentes CRUD admin (sesión 27)

- **Componentes con cambios profundos**:
  - `usuarios-lista`: 3 `p-dialog` → custom `modal-overlay`; `p-dropdown` → `mat-select`; `p-toast`/`p-confirmDialog` eliminados (ya usaba MatSnackBar/MatDialog en .ts); todos `pi pi-*` → `mat-icon`
  - `permisos-lista`: .ts reescrito — removido `MessageService`/`ConfirmationService`/`DialogModule`/`DropdownModule`, añadido `MatSnackBar`/`MatDialog`/`ConfirmDialogComponent`; `p-dialog` → modal-overlay; `p-dropdown` → `mat-select`; `pTooltip` → `matTooltip`
  - `mi-negocio-vista`: .ts reescrito — PrimeNG eliminado, `MatSnackBar` + `ModuloCompartido`; HTML: `pButton` → `primary-btn` custom; `pInputText` → `select-field`; `p-skeleton` → `animate-pulse`; `p-tag` → `<span>` Tailwind con `colorPlan()`
- **Componentes solo iconos** (6): `roles-lista`, `negocios-lista`, `secciones-con-permisos-lista`, `usuario-acceso`, `rol-acceso`, `negocio-permisos-modal`
- **Mapa de iconos clave**: `pi-arrow-left`→`arrow_back`, `pi-arrows-v`→`unfold_more`, `pi-ban`→`block`, `pi-check-circle`→`check_circle`, `pi-chevron-down`→`expand_more` (dinámico), `pi-star-fill`→`star`, `pi-spin pi-spinner`→`<mat-spinner>`
- **Estado plan**: Fases 0, 1, 2, 3, 4 ✅ — pendientes Fases 5, 6
- Actualizado: `venta-inventario-frontend.md` (Fase 4 marcada ✅)

## [2026-06-02] ingest | Fase 3 migración Angular Material — panel y dashboard (sesión 26)

- **Componentes migrados**: `panel.component` y `dashboard.component`
- **panel.component.html**: 4 iconos `pi pi-*` → `mat-icon` (notifications, domain, close, info)
- **panel.component.ts**: `MatIconModule` agregado a imports
- **panel.component.css**: selectores `.pi` → `mat-icon` con `width/height` explícitos
- **dashboard.component.html**: 8 iconos `pi pi-*` → `mat-icon`; 8 `p-skeleton` → `animate-pulse` divs; `pRipple` → `matRipple`
- **dashboard.component.ts**: `accesosRapidos.icono` actualizado a nombres Material (`inventory_2`, `domain`, `swap_horiz`); `MatIconModule` + `MatRippleModule` agregados
- **Estado plan**: Fases 0, 1, 2, 3 marcadas ✅ — pendiente Fases 4, 5, 6
- Actualizado: `venta-inventario-frontend.md` (Fase 3 marcada ✅)

## [2026-06-02] ingest | Fase 2 migración Angular Material — barra-lateral (sesión 25)

- **Componente migrado**: `barra-lateral.component` — el más complejo del proyecto (sidebar con contexto negocio)
- **`.ts`**: eliminados `RippleModule`, `AvatarModule`, `DropdownModule`, `TooltipModule` de PrimeNG → `MaterialModule`
- **Iconos nav**: array `elementos` migrado — PrimeIcon class names → Material Icons names (`dashboard`, `inventory_2`, `store`, `swap_horiz`, `label`, `sell`, `work`, `group`, `list`, `account_balance`, `language`, `domain`)
- **`p-dropdown` (contexto negocio)**: reemplazado por `mat-form-field` + `mat-select` + `mat-option` con dark theme via `::ng-deep`
- **`p-avatar`**: reemplazado por `<div class="avatar-circle">` custom (mismo diseño, sin dependencia PrimeNG)
- **`pRipple`**: → `matRipple`; **`pTooltip`**: → `[matTooltip]` + `matTooltipPosition`
- **Todos los `<i class="pi pi-*">`**: → `<mat-icon>nombre</mat-icon>` con clases CSS específicas para tamaño
- **`pi-circle-fill` (dot activo)**: → `<span class="dot-active">` con CSS puro (7px verde, no `mat-icon`)
- **Iconos dinámicos** (`pi-chevron-up/down`, `pi-angle-left/right`): → expresión Angular `{{ cond ? 'expand_less' : 'expand_more' }}`
- **CSS**: selectores `.pi` → `mat-icon`; dark theme overrides para `mat-form-field`; `.avatar-circle` nuevo; removidos todos los `:deep(.p-dropdown)` de PrimeNG
- **Plan HTML** actualizado: Fases 0, 1 y 2 marcadas ✅
- `venta-inventario-frontend.md` actualizado: Fases 1 y 2 marcadas ✅

## [2026-06-02] ingest | Fase 1 migración Angular Material — autenticación completa (sesión 24)

- **Componentes migrados**: `spa-input`, `spa-button`, `inicio-sesion`, `verificar-correo`, `registro-pendiente`
- `spa-input`: `pInputText` quitado; `pi-eye/eye-slash` → `mat-icon visibility/visibility_off`; `InputTextModule` → `MatIconModule`
- `spa-button`: `pButton` + `ButtonModule` + `RippleModule` → `matRipple`; `CommonModule` eliminado
- `inicio-sesion`: `p-checkbox` → `mat-checkbox` (label integrado); `CheckboxModule` eliminado; `ModuloCompartido` provee `MatCheckboxModule`
- `verificar-correo`: `p-progressSpinner` → `mat-spinner`; `pi-check-circle` → `mat-icon check_circle`; `pi-times-circle` → `mat-icon cancel`; CSS mat-icon 2rem añadido
- `registro-pendiente`: `pi-envelope` → `mat-icon mail`; `MatIconModule` añadido al componente; CSS mat-icon 2rem
- `registro`, `recuperar-contrasena`, `restablecer-contrasena`: sin PrimeNG directo — no requirieron cambios
- **Tests pendientes**: 3 frontend unit — `entrada.component.spec.ts` ×2, `inicio-sesion.component.spec.ts` ×1
- Actualizado: `venta-inventario-testing.md` (sprint 2026-06-02 Fase 1)

## [2026-06-02] ingest | Fase 0 migración PrimeNG → Angular Material — completada (sesión 23)

- **Desinstalado**: `primeng`, `primeflex`, `primeicons`
- **Instalado**: `@angular/material@18.2.14` (`@angular/cdk` ya estaba como dep)
- **Tema Material**: `azure-blue.css` — más cercano a `--spa-primary: #2b7ea6`
- **Archivos nuevos**: `material.module.ts` (23 módulos), `confirm-dialog.component.ts` (reemplaza `ConfirmationService`), `snack-bar.servicio.ts` (wrapper `MatSnackBar` con `exito/error/advertencia/info`)
- **angular.json**: estilos PrimeNG eliminados, `azure-blue.css` agregado
- **styles.css**: variables PrimeNG quitadas, `--spa-*` conservadas, clases `.snack-exito/error/aviso/info` para MDC override
- **index.html**: `Material Symbols Outlined` font agregada
- **shared.module.ts**: PrimeNG → `MaterialModule`
- **Estado app**: compila con errores (componentes aún referencian `pi pi-*`). Normal — se resuelve en Fases 1-6
- **Tests pendientes**: 4 unit tests frontend para `SnackBarServicio` y `ConfirmDialogComponent` — documentados en `venta-inventario-testing.md`
- Actualizados: `venta-inventario-frontend.md` (Fase 0 marcada ✅), `venta-inventario-testing.md` (sprint 2026-06-02)

## [2026-06-02] ingest | Plan migración PrimeNG → Angular Material — 7 fases

- **Decisión arquitectural**: reemplazar PrimeNG 17 completamente por Angular Material 18.x en el frontend de venta-inventario.
- **Versión compatible**: Angular 18.2.0 → `@angular/material@18` (paridad de versión mayor).
- **Dimensión**: 32 archivos HTML, 23 `.ts` con imports PrimeNG, 19 módulos distintos, 15 componentes únicos.
- **Plan en 7 sesiones**: Fase 0 setup → Fase 1 auth → Fase 2 shared → Fase 3 panel → Fase 4 CRUD admin → Fase 5 inventario → Fase 6 catálogo.
- **2 nuevos servicios/utilidades**: `SnackBarServicio` (reemplaza `MessageService`) + `ConfirmDialogComponent` (reemplaza `ConfirmationService`).
- **Plan HTML generado**: `docs/planes/plan-migracion-primeng-material.html` con tabla de equivalencias completa y estado de cada sesión.
- Estado al cierre: **Fase 0 no iniciada** — pendiente ejecución.
- Actualizado: `venta-inventario-frontend.md` (tabla equivalencias + fases + notas críticas)

## [2026-05-31] ingest | Landing page Adifnex + sistema de diseño

- Landing construida en `catalogo/app/page.tsx` (Next.js, ruta `/`)
- 8 secciones: Navbar, Hero, Problema, Cómo funciona, Pricing, Comparación, CTA final, Footer
- Paleta de marca definitiva: verde `#16A34A` / `#064E3B` + naranja `#EA580C` / `#C2410C`
- Angular (sidebar + login/registro) migrado de azul marino a verde esmeralda
- Imágenes generadas con Gemini — prompts documentados en conversación
- Regla prompts Gemini: siempre especificar "NO traditional/indigenous clothing", contexto Lima urbano moderno
- Pasarela elegida: Culqi (nativo Perú, soporta recurrencia, self-service vs Niubiz que requiere contrato)
- Onboarding P1 definido: 3 pasos — nombre negocio → primer producto → compartir link
- Actualizado: `wiki/modelos-negocio/adifnex-saas.md`

## [2026-05-31] ingest | Validación correo verificado en login + tests

- **Feature**: `iniciarSesion()` ahora bloquea usuarios no verificados antes de emitir tokens.
- **403 ForbiddenException** con campo `codigo`: `CORREO_NO_VERIFICADO_ACTIVO` (token vigente) o `CORREO_NO_VERIFICADO_EXPIRADO`.
- **Infraestructura**: `ExcepcionHttpFiltro` actualizado para propagar campo `codigo` en respuestas de error — patrón reutilizable para otros errores tipados.
- **Frontend** `inicio-sesion.component.ts`: detecta `codigo` → redirect a `/registro/pendiente` (token activo) o error inline (token expirado).
- **Tests**: +2 backend unit en `autenticacion.servicio.spec.ts` (25 total en la suite). +1 E2E spec `e2e/tests/01-autenticacion/correo-no-verificado.spec.ts` — crea usuario vía API de registro (autocontenido) → intenta login → verifica redirect.
- Actualizado: `venta-inventario-auth.md`, `venta-inventario-testing.md`

## [2026-05-31] ingest | Tests recuperación de contraseña — 5 backend unit + 2 E2E

- **Backend +5 tests** en `autenticacion.servicio.spec.ts`: `solicitarRecuperacion()` ×2 (correo existe/no existe) + `restablecerContrasena()` ×3 (válido/expirado/inexistente). Total suite: 23 tests 100% verde.
- **E2E nuevo** `e2e/tests/01-autenticacion/recuperar-contrasena.spec.ts` ×2: flujo completo email→banner + redirect sin token.
- **Patrón documentado**: tests anti-enumeración — verificar que el servicio no llama `correoServicio` cuando el correo no existe, pero retorna el mismo `{ mensaje }` en ambos casos.
- Actualizado: `venta-inventario-testing.md` (sprint 2026-05-31, totales ~188 backend / ~200 frontend)

## [2026-05-31] ingest | Flujo "Olvidé mi contraseña" — reset password completo

- **Migración 023**: `ALTER TABLE usuarios ADD COLUMN token_recuperacion / token_recuperacion_expira`
- **Backend**: 2 nuevos DTOs (`SolicitarRecuperacionDto`, `RestablecerContrasenaDto`), 2 métodos en `AutenticacionServicio`, 2 endpoints `@Publica()` en controlador
- **Anti-enumeración**: `solicitarRecuperacion()` siempre responde 200 aunque el correo no exista
- **Token expiry**: 15 minutos (vs 24h de verificación de cuenta)
- **URL fix**: `correo.servicio.ts` corregida — antes apuntaba a `/recuperar-contrasena`, ahora a `/restablecer-contrasena`
- **Frontend**: 2 nuevas rutas + componentes (`RecuperarContrasenaComponent`, `RestablecerContrasenaComponent`), validador cross-field "contraseñas coinciden", banner de éxito en login tras restablecer
- **Link en login**: "Olvide mi contrasena" agregado entre checkbox y botón
- Actualizado: `venta-inventario-auth.md`, `venta-inventario-correo.md`

## [2026-05-30] ingest | Tests unitarios: registro público, verificarCorreo, registrarNegocio, correo y guards

- **Backend +13 tests**: `autenticacion.servicio.spec.ts` ampliado (+11: registrar ×4, verificarCorreo ×4, registrarNegocio ×3). `correo.servicio.spec.ts` nuevo (+2).
- **Frontend +14 tests**: 4 archivos nuevos — `registro.component.spec.ts` (5), `verificar-correo.component.spec.ts` (4), `negocio-registro.component.spec.ts` (3), `negocio.guardia.spec.ts` (2).
- **Total acumulado**: ~183 backend / ~200 frontend / ~96 E2E.
- **Gotcha clave (RouterLink)**: componentes standalone que importan `RouterLink` en su array `imports` requieren `provideRouter([])` en TestBed — NO un mock de Router. `NO_ERRORS_SCHEMA` no resuelve esto porque el import es explícito, no solo en template.
- **Patrón transacción Sequelize**: mockear `modelo.sequelize = { transaction: jest.fn().mockResolvedValue(mockTx) }` — diferente a `getConnectionToken()` que se usa cuando la transacción se inyecta como Sequelize global.
- **Patrón nodemailer**: `jest.mock('nodemailer')` + `(createTransport as jest.Mock).mockReturnValue({ sendMail: spy })` en `beforeEach` antes de compilar el módulo.
- **Patrón guard funcional**: `TestBed.runInInjectionContext(() => guardiaFn(...))` para probar guards que usan `inject()`.
- Actualizado: `venta-inventario-testing.md`

## [2026-05-30] ingest | Separación registro usuario/negocio en dos pasos + campo teléfono

- **Token verificación no nullificado**: `tokenVerificacion` ya no se pone a null al activar cuenta — permite detectar doble click con mensaje correcto "ya verificada"
- **Re-registro con token expirado**: `registrar()` refresca token y reenvía correo si correo existe sin verificar, en lugar de lanzar ConflictException
- **Separación registro usuario/negocio** (cambio arquitectural):
  - `negocioId` ahora nullable en BD y entidad Sequelize
  - `PayloadJwt.negocioId: number | null`
  - `emitirTokens()` y `obtenerPermisosEfectivos()` manejan null sin crashear
  - Nuevo endpoint `POST /autenticacion/registrar-negocio` — crea negocio, vincula usuario, emite tokens actualizados
  - Frontend: formulario registro simplificado (sin campos negocio)
  - Nuevo componente `NegocioRegistroComponent` en `/negocio/nuevo`
  - Nuevo guard `guardiaNegocio` — redirige a `/negocio/nuevo` si `negocioId` null
  - Rutas `/dashboard` ahora requieren ambos guards: `guardiaAutenticacion` + `guardiaNegocio`
- **Campo teléfono requerido**: migración 021, columna `telefono VARCHAR(20)`, DTO backend, formulario frontend con validación de patrón
- Actualizado: `venta-inventario-auth.md`, `venta-inventario-frontend.md`

## [2026-05-31] ingest | Bugs responsive registro + OnPush signals + verificar cuenta ya activa

- **Pantalla registro responsive**: 3 bugs resueltos — grid no colapsaba (35fr persist), subtitle comprimido (flex header), scroll bloqueado (`overflow: hidden` global en body)
- **OnPush gotcha**: propiedades planas no disparan CD → convertido `enviando`/`errorGeneral` a Signals en `RegistroComponent`
- **Verificar correo**: distinción "token inválido" vs "cuenta ya verificada" — backend busca sin filtro `correoVerificado`, frontend nuevo estado `'ya-verificado'` con UI de éxito
- Actualizado: `venta-inventario-auth.md`, `venta-inventario-frontend.md`

## [2026-05-30] ingest | Registro público autoservicio con verificación de correo

- Migración 020: 3 columnas nuevas en `usuarios` (`correo_verificado`, `token_verificacion`, `token_verificacion_expira`)
- Endpoints nuevos: `POST /api/autenticacion/registro` y `GET /api/autenticacion/verificar?token=`
- Transacción: negocio(0) → usuario(0) → patch circular ref → SUPERADMIN rol → commit → email fire&forget
- Cuenta bloqueada hasta verificar; al verificar → login automático con tokens
- `CorreoServicio` ampliado: `enviarVerificacionCorreo()` — sin paquete nuevo (ya usaba nodemailer)
- Frontend: 3 componentes nuevos (`/registro`, `/registro/pendiente`, `/verificar`) + rutas sin guard
- Actualizado: `venta-inventario-auth.md`, `venta-inventario-correo.md`

## [2026-05-29] ingest | Módulo correo Gmail SMTP + plan registro autoservicio

- Plan registro público autoservicio documentado en `docs/planes/plan-registro-publico-autoservicio.html`
- Plan infraestructura correo documentado en `docs/planes/plan-correo-electronico.html`
- Decisión: Gmail SMTP (`adifnex@gmail.com`) con App Password — 30 emails/día máximo, no justifica proveedor externo
- Módulo `src/correo/` creado: `CorreoServicio` con `enviarBienvenida()` + `enviarRecuperacionContrasena()`
- Templates HTML inline compatibles con Gmail/Outlook
- `@Global()` — inyectable en toda la app sin reimportar
- Prueba exitosa: POST `/api/correo/test` → correo llegó a inbox
- Feedback guardado: no pedir autorización en cada tool use — proceder directo

## [2026-05-29] ingest | Adifnex SaaS — estrategia comercial inicial

- Decisión de enfoque: catálogo público (opción B, tipo Shopify)
- Creada página `modelos-negocio/adifnex-saas.md`
- Creado `.agents/product-marketing.md` (contexto para skills de marketing)
- Instalados 42 marketing skills (npx skills add coreyhaines31/marketingskills)
- Pricing en evaluación: S/15 / S/19 / S/29 mensuales
- Mercado inicial: Perú
- Gaps P0 identificados: registro público, pagos, activar vencimiento

---

## [2026-05-29] ingest | Migración Railway → Render + deploy catálogo F8

- **Railway agotó trial** — créditos a $0. Backend migrado a Render free tier.
- **Render Web Service**: Docker (no Node), región Oregon, PORT inyectado automático — no agregar como var.
- **Gotcha Render free tier**: cold start 30–50s — Vercel SSR puede timeout → `PaginaNoDisponible`. Solución: Starter $7/mes o health check externo.
- **Gotcha Vercel `NEXT_PUBLIC_*`**: se inyecta en build time — cambiar la var requiere redeploy sin cache.
- **`NEXT_PUBLIC_API_URL` correcto**: `https://inventario-multisede-backend.onrender.com/api` (con `/api`, sin trailing slash).
- Páginas actualizadas: `venta-inventario-catalogo.md` (F8 en progreso), `railway.md` (archivado).
- Página nueva: `tecnologias/render.md`.

## [2026-05-24] ingest | Segunda plantilla catálogo: catalogo-esencias (sesión 13)

- **Nueva plantilla `catalogo-esencias`**: estética de perfumería/colonias de lujo. Paleta: ivory #f7f3ee, espresso #1a1209, dorado #b8975c. Tipografía: Playfair Display (headings, --font-display) + Nunito Sans (cuerpo).
- **Selección de plantilla por API**: campo `plantilla: string | null` agregado a `NegocioConfig`. Router en `app/[slug]/page.tsx` hace switch — default = `CatalogoGrilla`. BD pendiente de migración.
- **Diferencia estructural clave vs grilla**: sidebar de filtros a la izquierda (desktop) + pills mobile — componente `FilterSidebar.tsx` separado. Imagen 3:4 vertical. Sin border/shadow en cards.
- **Playfair Display** cargada en `app/layout.tsx` como CSS variable `--font-display` (nombre genérico para futura extensibilidad).
- **Mock data esencias**: `MOCK_CONFIG_ESENCIAS`, `MOCK_CATEGORIAS_ESENCIAS`, `MOCK_PRODUCTOS_ESENCIAS` en `lib/mock-data.ts`.
- Página actualizada: `venta-inventario-catalogo.md`.

## [2026-05-24] ingest | Fix Cloudinary en producción - venta-inventario

- **Bug**: imágenes subidas en Railway se guardaban como `http://localhost:13200/uploads/...` en lugar de URLs Cloudinary.
- **Causa raíz**: `StorageServicio` usaba `process.env.STORAGE_PROVIDER ?? 'local'` — si la variable no se leía en el primer deploy, el fallback era `'local'`.
- **Fix**: lógica invertida a `process.env.STORAGE_PROVIDER === 'local' ? 'local' : 'cloudinary'`. Cloudinary por defecto en producción.
- **Error intermedio**: credenciales Cloudinary incorrectas en Railway → 500. Corregidas con valores de la consola Cloudinary (`cloud_name=dovxgcpc5`).
- **Aprendizaje Railway**: variables en morado = cambios pendientes sin aplicar. Siempre hacer Deploy después de editar variables.
- **Regla local**: `STORAGE_PROVIDER=local` en `.env` local para seguir usando disco en desarrollo.
- Páginas actualizadas: `venta-inventario-catalogo.md` (decisión StorageServicio), creada `tecnologias/railway.md`.

## [2026-05-23] update | galería imágenes — 2 fixes post-F7

- **Parpadeo al marcar imagen principal**: `marcarComoPrincipal()` llamaba `cargarImagenes()` causando HTTP + signal reset + scroll jump. Fix: actualización optimista con `imagenes.update(imgs => imgs.map(...))`. No recarga, no parpadeo.
- **StorageServicio.eliminarImagen()**: el DELETE de imagen solo hacía soft delete en BD — Cloudinary no liberaba espacio. Nuevo método `eliminarImagen(url)` que extrae `public_id` con regex y llama `cloudinary.uploader.destroy()`. Local: `fs.unlinkSync()`. Try/catch silencioso — BD no se bloquea si Cloudinary falla.
- Documentado en `venta-inventario-catalogo.md` — sección Decisiones de diseño.

## [2026-05-19] update | storage por negocio + suite completa de tests (sesión 9)

- **StorageServicio**: `subirImagen` ahora recibe `negocioId: number`. Cloudinary: `folder: inventario-ventas/{id}/productos`. Local: `uploads/negocio-{id}/productos/`. Controller multerStorage lee `req['negocioId']`. Imágenes existentes no migradas — solo nuevos uploads.
- **Suite backend +29 tests**: `storage.servicio.spec.ts` (nuevo, 4), `publico.servicio.spec.ts` (nuevo, 8), `productos.servicio.spec.ts` (ampliado +5 imágenes), `negocio.servicio.spec.ts` (ampliado +3 paginaConfig). Total backend ~170.
- **Suite frontend +47 tests**: `mi-pagina-catalogo.servicio.spec.ts` (5), `producto-imagenes.servicio.spec.ts` (6), `marcas-lista.component.spec.ts` (7), `etiquetas-lista.component.spec.ts` (8), `productos-lista.component.spec.ts` (7), `catalogo-producto-dialog.component.spec.ts` (14). Total frontend ~186.
- **Suite E2E +9 tests**: nuevo PO `mi-pagina.page.ts`, `13-catalogo/catalogo-mi-pagina.spec.ts` (5), `13-catalogo/catalogo-imagenes.spec.ts` (4). `productos.page.ts` ampliado con `abrirModalCatalogo`.
- **Regresiones**: ninguna. Pre-existentes: `sedes.servicio.spec.ts` y `usuarios.servicio.spec.ts` (`NullInjectorError: NegocioRepository`) — pendientes de diagnóstico.
- Páginas actualizadas: `venta-inventario-testing.md` (sesión 9), `venta-inventario-catalogo.md` (decisión storage).

## [2026-05-19] update | adifnex-catalogo — 2 bugs resueltos: esPrincipal FormData boolean + Docker npm install

- **Bug `esPrincipal` siempre `true`**: `enableImplicitConversion: true` en ValidationPipe convierte el string `'false'` (enviado por FormData) a `Boolean('false')` = `true` (non-empty string es truthy). El backend guardaba la imagen como principal aunque el checkbox estuviera desmarcado. Fix: en el servicio Angular, solo enviar `esPrincipal: 'true'` cuando es `true` — omitir el campo si es `false`. Backend recibe `undefined` → `dto.esPrincipal ?? false` = `false`. Regla derivada: **para booleans en FormData, nunca enviar el valor negativo**.
- **Docker npm install**: paquete `@nestjs/serve-static` faltaba en el contenedor. Instalar en el host no tiene efecto. Fix: `docker exec backend npm install @nestjs/serve-static`. Regla: nuevos paquetes deben instalarse dentro del contenedor o hacer rebuild.
- Página actualizada: `venta-inventario-catalogo.md` (sesión 8).

## [2026-05-18] update | adifnex-catalogo — plan MD+HTML actualizados: F7 ✅, suite de pruebas completa (sesión 8)

- **F7 marcada completa** en `plan-catalogo-react.md` y `plan-catalogo-react.html`: cabecera, badges del header, y bloque de fase pasan de "ACTIVA" a "✅ Completado 2026-05-18".
- **Sección 11 añadida al MD** (pruebas completas): 4 subsecciones — backend unit (publico.servicio, negocio.servicio ampliación, productos.servicio ampliación, storage.servicio), frontend services (mi-pagina-catalogo.servicio, producto-imagenes.servicio), frontend components (catalogo-producto-dialog, mi-pagina, 3 pendientes sprint anterior), E2E Playwright (13-catalogo/ nuevos, actualización productos.page.ts, permisos, pendientes sprint).
- **Sección "Suite de pruebas" añadida al HTML**: bloques visuales con color por categoría (violet=backend, blue=frontend services, emerald=frontend components, orange=E2E), badges NUEVO/AMPLIAR/PENDIENTE, checklist interactivo.
- **Impacto en E2E existentes verificado**: `productos-crud.spec.ts` sigue funcionando — botón `pi-globe` usa `aria-label="Catálogo"`, no colisiona con `aria-label="Editar"` ni `aria-label="Eliminar"`. No hay regresión.
- **Total pruebas nuevas mapeadas**: 9 backend unit + 7 frontend service unit + 11 frontend component unit + 10 E2E nuevos + 1 actualización page object.

---

## [2026-05-18] update | adifnex-catalogo — F7 implementada: modal Catálogo con precios + galería imágenes (sesión 7)

- **UX decisión clave**: F7 no es solo un dialog de galería — es un modal unificado `CatalogoProductoDialog` que consolida precio, precioOferta, visibleCatalogo, ordenCatalogo (MOVIDOS del edit dialog) + galería de imágenes. Edit dialog queda solo con datos básicos del producto.
- **Botón `pi-globe`**: agregado en desktop (tabla) Y mobile (cards) de `productos-lista`, gated con `CATALOGO_PRODUCTO_IMAGEN_VER`.
- **StorageServicio**: abstracción local/Cloudinary. `STORAGE_PROVIDER=local` → multer diskStorage en `uploads/productos/` (git-ignored) + ServeStaticModule. `STORAGE_PROVIDER=cloudinary` → `cloudinary.uploader.upload()` → `secure_url`. URL siempre absoluta con `BACKEND_URL`.
- **Endpoints nuevos**: `GET/POST/PATCH/DELETE /inventario/productos/:id/imagenes`. POST acepta `multipart/form-data` con campo `archivo`.
- **Gotcha Sequelize-TypeScript**: `@BelongsTo()` en `ProductoImagen` no tiene `CreationOptional<>` → `.create()` falla en TS. Fix: `as any`. No es bug — es límite del tipado de Sequelize-TypeScript.
- **Gotcha FormData + HttpBaseService**: no setear `Content-Type` manualmente — Angular lo hace automáticamente con el boundary. `esPrincipal` bool viaja como string `'true'`/`'false'` en FormData → DTO usa `@Transform` para convertir.
- **Backend compila sin errores**. Frontend TypeScript limpio (solo errores pre-existentes en specs).
- Página actualizada: `venta-inventario-catalogo.md` (sesión 7).

## [2026-05-18] update | adifnex-catalogo — UX Mi Página: spotlight granular, permisos slug, PaginaNoDisponible (sesión 6)

- **Spotlight "ojito" refinado**: cada campo apunta a un ID granular (`hero-titulo`, `hero-subtitulo`, `hero-imagen`, `hero-cta`, `navbar-nombre`, `navbar-logo`, `tab-favicon`). El glow `box-shadow` va al elemento exacto (h2, p, img, span), no al padre. Dimming usa `startsWith()` para respetar subsecciones.
- **Nombre público en dos lugares**: navbar + footer col 1 ambos brillan al activar `navbar-nombre`. La columna footer no se oscurece.
- **Botón CTA temporal**: aparece con placeholder si está vacío cuando se activa `hero-cta`.
- **Slug bloqueado**: nuevo permiso `CATALOGO_MI_PAGINA_SLUG_EDITAR` (migración 016, SUPERADMIN only). Sin permiso → campo readonly con badge de aviso.
- **Ojitos removidos**: Slug, Título SEO, Descripción SEO ya no tienen ojo.
- **PaginaNoDisponible**: mock fallback eliminado de `app/[slug]/page.tsx`. Reemplazado por componente de pantalla de "no disponible" con candado.
- **Favicon Next.js**: `app/favicon.ico` físico tiene prioridad absoluta sobre `generateMetadata.icons` — hay que eliminarlo para que el favicon dinámico funcione.
- **Reset botón**: `bg-transparent border-0 p-0 focus:outline-none` para cancelar estilos browser por defecto en `<button>`.
- **`w-fit` vs `inline-block`**: `w-fit` = bloque que abraza contenido — correcto para glow en h2/p sin que fluyan inline.
- **PrimeNG Tooltip**: `pTooltip` + `TooltipModule`, nunca `title=""`.
- Página actualizada: `venta-inventario-catalogo.md` (sesión 6).

## [2026-05-18] update | adifnex-catalogo — bug contexto negocio en Mi Página + regla @NegocioActual

- **Bug (2 causas)**: al cambiar negocio en el selector del panel, "Mi Página" no recargaba los datos del negocio nuevo.
- **Causa frontend**: `MiPaginaComponent.ngOnInit()` llamaba `cargar()` una sola vez — no suscrito al signal de contexto. Fix: `toObservable(negocioCtx.negocioActivo).pipe(filter, switchMap)`.
- **Causa backend**: `negocio.controlador.ts` usaba `req.user.negocioId` en 4 métodos (`obtenerMiNegocio`, `actualizarMiNegocio`, `obtenerPaginaConfig`, `actualizarPaginaConfig`) — bypasseaba `TenantInterceptor`. Fix: reemplazar por `@NegocioActual()`.
- **Regla documentada**: controllers tenant-scoped DEBEN usar `@NegocioActual()`, no `req.user.negocioId`. Documentado en `venta-inventario-rbac.md`.
- **Patrón documentado**: `toObservable + switchMap` para recarga HTTP reactiva al cambio de contexto. Documentado en `venta-inventario-frontend.md`.
- Páginas actualizadas: `venta-inventario-catalogo.md` (sesión 5), `venta-inventario-frontend.md`, `venta-inventario-rbac.md`.

## [2026-05-18] update | adifnex-catalogo — 2 bugs del catálogo Next.js documentados

- **Bug 1 (UX gotcha)**: productos con `visibleCatalogo=true` y `precio=null` no aparecen en catálogo público. El panel los muestra como "Visible" pero el backend filtra con `precio NOT NULL` (`publico.servicio.ts:47`). Fix: asignar precio al producto.
- **Bug 2 (TypeError)**: `producto.precioOferta.toFixed is not a function` en `ProductCard.tsx:117`. Causa: PostgreSQL NUMERIC → JSON string, no number. Fix: `Number(producto.precioOferta).toFixed(2)`.
- Página actualizada: `venta-inventario-catalogo.md` — decisiones de diseño ampliadas con ambos gotchas.

## [2026-05-17] ingest | adifnex-catalogo F4+F5+F6 completadas — módulo publico NestJS, tipos TypeScript alineados, panel Mi Página Angular

- **F4 ✅**: módulo `publico/` en NestJS. 3 endpoints públicos (`@Publica()`): `GET /pub/:slug`, `/pub/:slug/productos`, `/pub/:slug/categorias`. `resolverNegocioId()` privado compartido. Productos con `ORDER BY NULLS LAST` vía `literal()` de Sequelize.
- **F5 ✅**: tipos TypeScript alineados con backend real. `NegocioConfig` sin colores/layoutTemplate. `ProductoPublico` con `imagenes: ImagenProducto[]`, `sku`, `ordenCatalogo`. Nuevo patrón THEME: `templates/catalogo-grilla/theme.ts` con constantes de color — todos los componentes usan THEME en vez de `config.color*`. `app/[slug]/page.tsx` sin `switch(layoutTemplate)`.
- **F6 ✅**: panel Angular "Mi Página" implementado. Split-pane: form 4 tabs (PrimeNG `p-tabView`) + preview vivo con signal. Backend: `GET/PATCH /negocio/pagina-config` con `@Permisos`. Upsert: crea registro vacío en primer uso. Migración 015: permisos `CATALOGO_MI_PAGINA_VER` + `CATALOGO_MI_PAGINA_EDITAR`. Panel registrado en `panel.component.ts/html`.
- Estado fases: F1✅ F2✅ F3✅ F4✅ F5✅ F6✅ — pendientes F7 (galería imágenes) y F8 (deploy).
- Página actualizada: `venta-inventario-catalogo.md`

## [2026-05-17] ingest | adifnex-catalogo F3 completa + rediseño negocio_pagina_config + plan auditoría BD + F6 split-pane

- **F3 completada**: migración SQL `014-catalogo.sql` creada. Entidades NestJS nuevas: `ProductoImagen`, `NegocioPaginaConfig`. Entidad `Producto` actualizada con `precio`, `precioOferta`, `visibleCatalogo`, `ordenCatalogo`. Módulos registrados. `setup-completo.sql` reescrito completo (estaba muy desactualizado — faltaban ~10 tablas).
- **Rediseño negocio_pagina_config** (decisión clave): eliminados `color_primario`, `color_fondo`, `color_texto`, `layout_template`. Los colores son fijos en el CSS de cada plantilla — no son configurables por negocio. La tabla solo almacena contenido (identidad, hero, footer, SEO). Campos restantes: 21 columnas de contenido.
- **Regla setup-completo.sql**: formalizada en `CLAUDE.md` del proyecto y en memory persistente. Toda migración SQL debe actualizar también `setup-completo.sql` en la misma respuesta. Christian elimina los archivos de migración después de aplicarlos a producción — `setup-completo.sql` es el único documento permanente del esquema.
- **Plan de auditoría BD creado**: `docs/auditoria-bd/plan-auditoria-schema.html` + `.md`. Incluye 6 queries para correr en producción (tablas, columnas, triggers, funciones, índices, ENUMs) + checklist de 22 tablas esperadas + validación de timestamps automáticos (`fecha_actualizacion` con/sin trigger).
- **Comparativo config catálogo**: `docs/catalogo-landing/comparativo-config-catalogo.html` — tabla izquierda/derecha (campos actuales vs propuesta), preview del formulario admin con tabs, preview de la página pública con anotaciones de qué campo alimenta qué zona.
- **F6 diseñado**: Panel Angular "Mi Página" = split-pane. Formulario con 4 tabs (Identidad, Hero, Footer & Contacto, SEO) a la izquierda. Previsualización reactiva de la página pública a la derecha con anotaciones de campo. Signal compartida + `valueChanges`. Guard: `CATALOGO_CONFIG_EDITAR`.
- **plan-catalogo-react.html + .md actualizados**: nueva sección F6 con mockup split-pane, tabla de mapeo campo→zona. `NegocioConfig` TypeScript limpia (sin colores). Mock data sin `colorPrimario`. Decisiones de diseño corregidas.
- Página actualizada: `venta-inventario-catalogo.md`

## [2026-05-17] ingest | adifnex-catalogo F1 + F2 completadas — repo GitHub, Docker, plantilla Lullaby Soft

- **F1 completa**: repo `ChristianCHJH/inventario-multisede-catalogo-react` creado. Next.js 14, App Router, Tailwind. `lib/types.ts`, `lib/api.ts`, `app/[slug]/page.tsx`. `Dockerfile.dev` listo.
- **Puerto 3010**: 3001 estaba ocupado por `EMR.Patient.Service-dev` (otro proyecto en el mismo entorno). Se movió el catálogo a 3010 permanentemente.
- **Docker integrado**: servicio `catalogo` añadido al `docker-compose.dev.yml` del mono-repo. `NEXT_PUBLIC_API_URL=http://backend:3200/api` por red interna Docker.
- **F2 completa**: plantilla `catalogo-grilla` con 7 componentes React. Mock fallback en `page.tsx` — si API falla usa MOCK_CONFIG (Lullaby Soft). Accesible en `localhost:3010/lullaby-soft`.
- **Decisión mock fallback**: `app/[slug]/page.tsx` catch → usa mock en lugar de `notFound()`. Permite desarrollar F2-F5 sin backend activo.
- **Dark mode removido**: `globals.css` tenía `@media (prefers-color-scheme: dark)` que sobreescribía el fondo crema. Eliminado para que el branding del negocio prevalezca.
- **Próxima fase**: F3 — migración SQL 014 (precio, producto_imagen, negocio_pagina_config).
- Página actualizada: `venta-inventario-catalogo.md`

## [2026-05-15] ingest | Catálogo público adifnex-catalogo — plan, mockup, permisos Plan 2

- **Nuevo sub-proyecto**: `adifnex-catalogo` — Next.js 14 para páginas de catálogo públicas multi-negocio.
- **Decisión clave — plantillas**: cada plantilla es un diseño completamente distinto (layout, colores, tipografía), no un cambio de tema. El modelo de datos es homogéneo entre plantillas.
- **Permisos Plan 2**: 9 permisos nuevos en sección `CATALOGO` (precio, imágenes, footer, plantilla, visibilidad). Funcionalidad exclusiva del plan de suscripción intermedio.
- **Gap analysis BD**: faltan `precio`, `precio_oferta`, `visible_catalogo`, `orden_catalogo` en `producto`. Falta tabla `producto_imagen`. Falta tabla `negocio_pagina_config` (zero SQL existente).
- **Migración 014**: crea columnas en producto + tabla producto_imagen + tabla negocio_pagina_config completa.
- **Fase F4.5** añadida al roadmap: migración SQL permisos CATALOGO + asignar a rol Plan 2.
- **Archivos creados**: `docs/catalogo-landing/plan-catalogo-react.md`, `plan-catalogo-react.html`, `mockup-lullaby-soft.html`.
- Página creada: `venta-inventario-catalogo.md`. Hub `venta-inventario.md` actualizado.

## [2026-05-15] update | Fix E2E productos.page.ts — SKU locator y modo automático

- **Bug 1**: `inputSku` usaba `input[placeholder="Ej: ACE-LAV-001"]` que ya no existe tras rediseño del formulario. Fix: `input.sku-field`.
- **Bug 2**: Campo SKU está `disabled` por defecto (`skuAutomatico = true`). `crearProducto()` intentaba `fill()` en un input disabled — falla silenciosamente. Fix: hacer `check()` en el checkbox `.sku-toggle` antes de llenar el campo.
- **Parámetro SKU** vuelto opcional en `crearProducto(nombre, sku?, descripcion?)`.
- Página actualizada: `venta-inventario-testing.md`

---

## [2026-05-14] ingest | Rediseño tabla productos, paginación, sorting, bugs display:flex y bulkCreate

- **UI tabla productos**: SKU + descripción fusionados en columna Nombre. Auditoría con fecha+hora debajo del usuario. Fuentes reducidas (th .72rem, td .82rem). Scroll solo en `.table-scroll` — cadena flex `:host → .shell → .data-card → .table-wrapper → .table-scroll`.
- **Paginación client-side**: 20 por página, signals `paginaActual`/`totalPaginas`/`productosPaginados`. Footer con botones prev/next + numeración con ellipsis. Effect resetea a página 1 al cambiar filtro. Aplica igual en mobile.
- **Sorting client-side**: campos id, nombre, fechaCreacion, fechaActualizacion. Signals `ordenCampo`/`ordenDir`. Toggle asc/desc al hacer click en el mismo campo. Headers con iconos PrimeIcons tri-estado.
- **Bug `display:flex` en `<td>`**: rompe layout de tabla — columnas se mezclaban visualmente. Regla: nunca flex directo en td/th, usar div wrapper interno.
- **Bug mobile paginación**: cards usaban `productosFiltrados()` en lugar de `productosPaginados()` — solo se veían los primeros N sin scroll.
- **Bug etiquetas no se guardaban (Sequelize)**: `ignoreDuplicates: true` en `bulkCreate` genera `ON CONFLICT DO NOTHING`. Par (productoId, etiquetaId) ya existía con `eliminado=true` → insert ignorado → etiqueta nunca reactivada. Fix: `updateOnDuplicate: ['eliminado', 'usuarioActualizacion']` con `eliminado: false` explícito. Aplica en `crear()` y `actualizar()`.
- **Bug strings en multiSelect**: `p-multiSelect` puede retornar optionValue como string. Fix: `.map(Number)` en etiquetaIds y `Number(v.marcaId)` antes de enviar al backend.
- Página actualizada: `venta-inventario-frontend.md`

## [2026-05-14] ingest | Marcas + Etiquetas CRUD, fix lista productos, SQL ON CONFLICT

- **Nuevos módulos backend**: `MarcasModulo` y `EtiquetasModulo` con endpoints CRUD completos. Tablas `marca` y `etiqueta` (ambas con `negocio_id`, `nombre`, `color` en etiquetas). Pivote `producto_etiqueta` para relación N:M.
- **Asociaciones Sequelize**: `Producto` ahora declara `@BelongsToMany(() => Etiqueta, () => ProductoEtiqueta)`. Sin este decorador, `include: [Etiqueta]` en `listar()` devuelve `undefined` silenciosamente.
- **`listar()` productos extendido**: añade `include: [Etiqueta]` + literal subquery `marcaNombre`. Antes devolvía solo los campos de `producto`.
- **Bug fix lista productos**: `p.etiquetas` llegaba `undefined` al frontend → Angular crasheaba la fila → se veían círculos grises. Fix: interface `etiquetas?:` opcional + guards `(p.etiquetas ?? [])` en template.
- **Bug SQL `42P10`**: `ON CONFLICT (col) DO NOTHING` falla si no existe UNIQUE constraint en la DB. Fix universal: `WHERE NOT EXISTS (SELECT 1 FROM tabla WHERE col = val)` aplicado en migración 011.
- **Nuevos componentes Angular**: `marcas-lista/` y `etiquetas-lista/` — CRUD completo, tabla desktop + cards mobile, dialog, color picker (etiquetas), confirm eliminar.
- **Sidebar + panel**: ítems Marcas (`pi-tag`) y Etiquetas (`pi-tags`) añadidos bajo INVENTARIO.
- **Pruebas pendientes**: 5 suites unit backend + 5 suites unit frontend + 3 specs E2E documentados en `CLAUDE.md` del proyecto y en `venta-inventario-testing.md`.
- Páginas actualizadas: `venta-inventario-inventario.md`, `venta-inventario-frontend.md`, `venta-inventario-testing.md`.

## [2026-05-12] ingest | E2E ronda 3 — 6 fixes aplicados, 58→?? tests, 7 pendientes

- **Bug 1 — tilde faltante**: `acceptLabel: 'Si, activar/desactivar'` en usuarios/roles/negocios → Playwright `has-text("Sí")` no matcheaba. Fix: agregar tilde en 3 componentes.
- **Bug 2 — beforeEach inline sin page-title**: `secciones-permisos.spec.ts` y `dashboard.spec.ts` usaban `waitForFunction(!animate-pulse)` sin el `waitForSelector('.page-title')` previo. Race condition idéntica a la de ronda 2 pero en specs inline, no en page objects.
- **Bug 3 — host element vs inner div**: `page.locator('spa-sidebar')` (host Angular) → `page.locator('.spa-sidebar')` (inner div con CSS real).
- 7 tests pendientes de diagnóstico (necesitan output verbose de Playwright): editar usuario (14.4s), reset password (30.1s timeout global), sedes soft-delete, movimientos INGRESO/SALIDA, carga-inicial, transferencia.
- Hallazgo: `PermisosUsuarioServicio` in-memory, init en `PanelComponent.ngOnInit()` via JWT payload decode. `PermisoDirectiva` reactiva con `effect()`.
- Hallazgo: múltiples `<p-dialog>` en usuarios template — puede causar strict mode violation en tests.
- `venta-inventario-testing.md` actualizado con ronda 3.

## [2026-05-12] ingest | E2E ronda 2 — 3 bugs raíz resueltos, NegocioContexto persiste

- **Bug 1 race condition**: `esperarCarga()` resolvía antes de que Angular montara. Fix: `waitForSelector('.page-title')` antes del `waitForFunction`. Aplicado en 6 page objects.
- **Bug 2 negocio contexto**: `NegocioContextoServicio` era in-memory puro → `sinContexto()` siempre true en tests → tabla productos nunca visible. Fix doble: servicio ahora lee/escribe `localStorage['spa.negocio.contexto']`; `global-setup.ts` llama `GET /api/negocio/mi-negocio` (fallback `GET /api/negocio`) y guarda el negocio en el storageState. Mejora UX también en producción (contexto persiste al refrescar).
- **Bug 3 waitForResponse invertido**: en `usuarios-crud.spec.ts` el `await waitForResponse()` bloqueaba 10s antes de ejecutar la acción que dispara la respuesta. Fix: Promise sin await → acción → await. Test bajó de 16s → ~3s.
- Concepto documentado: alcance correcto de E2E = flujos que QA haría en staging (crear, editar, toggle estado, buscar, soft-delete).
- `venta-inventario-testing.md` actualizado con bugs, fixes y tabla de alcance E2E.

---

## [2026-05-12] ingest | Fase 3 E2E — Playwright implementado, bug envelope corregido

- Suite E2E creada en `e2e/` (package.json independiente): 19 specs, ~71 tests, Playwright 1.60 + Chromium
- Estructura: `pages/` POM (10 clases), `tests/01-10/`, `global-setup.ts`, `fixtures/`
- 2 proyectos Playwright: `autenticacion` (sin storageState) + `app` (con storageState)
- Credenciales dev confirmadas: `admin@museo.local` / `admin123` (no christian011215)
- Backend URL: `http://localhost:3200` prefijo `/api` — Docker expone al host, Playwright corre en host
- Token storage key: `spa.auth.tokens`, estructura `{tokenAcceso, tokenRefresco, tipoToken, expiraEn}`
- Guard `guardiaAutenticacion`: solo verifica `obtenerTokenAcceso()` — no valida expiración ni API call
- **Bug crítico resuelto**: `global-setup.ts` usaba `cuerpo.data` pero backend retorna `cuerpo.datos` (envelope estándar `{exito, datos}`). Token nunca se guardaba → todos los tests del proyecto `app` redirigían a login. Fix: `cuerpo.datos ?? cuerpo.data ?? cuerpo`
- Estado al cierre: 13 pasados, 51 fallando por selectores PrimeNG — ajuste en curso

---

## [2026-05-11] ingest | Fase 2 pruebas — Jest frontend, 110 tests, 14 servicios Angular

- Migración Karma → Jest + jest-preset-angular 16 en `frontend/`
- Setup: `jest.config.ts`, `setup-jest.ts`, `tsconfig.spec.json` actualizado
- 14 suites: 2 servicios de estado local (signals) + 12 servicios HTTP
- 110 tests, 100% verde, 4.8s
- Patrón: `HttpClientTestingModule` + `HttpTestingController.expectOne()` — un test por endpoint
- Branch coverage baja en variantes de envelope alternativas (ramas no producibles) — esperado
- Sub-página `venta-inventario-testing.md` actualizada: Fase 2 marcada completada, tabla de specs

---

## [2026-05-12] ingest | Estrategia de pruebas Jest backend — 114 tests, 15 servicios

- Conversación completa sobre estrategia de pruebas para venta-inventario
- Decisión: Jest para backend y frontend, Playwright para E2E (Cypress descartado)
- Fase 1 completada: 114 unit tests sobre los 15 servicios NestJS del backend
- Configuración: `jest.config.ts` con `diagnostics: false` en ts-jest (servicios con TS strict issues)
- Mock pattern Sequelize: `getModelToken(Entidad)` y `getConnectionToken()` para transacciones
- Lección crítica: mayoría de servicios usan `obj.prop = valor; obj.save()` — NO `obj.update({prop})`
- Lección: `NegocioServicio.buscarPorId` usa `findOne` (no `findByPk`) — siempre leer el servicio antes de asumir
- Explicación pedagógica: qué prueban los unit tests de Angular (contrato HTTP, no HTML ni clicks)
- ADR `ADR-estrategia-pruebas.md` creado con glosario E2E + pirámide de pruebas
- Nueva sub-página: `venta-inventario-testing.md`
- Hub `venta-inventario.md` actualizado con link y estado

---

## [2026-05-11] ingest | nivelMinimo inline, limpieza de permisos, migraciones 008-010

- **Selector inline de nivel**: pills Bás./Prof./Emp./Dev. en sub-tabla de secciones-con-permisos-lista. `nivelCargando` y `seccionNivelCargando` son `signal<Set<number>>` para rastrear estado por permiso y por sección. `forkJoin` para bulk update.
- **Bug double-unwrap**: servicios de inventario (productos, sedes, stock) usaban `RespuestaApi<T>` + `.map(r => r.datos)` — HttpBaseService ya desempaqueta. Fix: tipo directo en genérico.
- **Migración 008**: seed permisos ADMINISTRACION_NEGOCIO y NEGOCIOS; soft-delete PLAN_ACTIVO
- **Migración 009**: SEGURIDAD_PERMISOS consolidada en SEGURIDAD_SECCIONES; seed SEGURIDAD_USUARIOS_VER_HISTORIAL; assign SUPERADMIN
- **Migración 010**: soft-delete de 10 permisos sin uso en UI; sección DASHBOARD vacía → soft-delete
- **Guard mi-negocio**: PLAN_ACTIVO_VER → NEGOCIOS_VER en panel.component.ts; PLAN_ACTIVO_EDITAR → NEGOCIOS_EDITAR en mi-negocio-vista.component.html
- **diagnostico-permisos.sql**: script de auditoría DB vs. frontend (CTE con todos los permisos usados → LEFT JOIN)
- Documentado en `venta-inventario-rbac.md` y `venta-inventario-frontend.md`

---

## [2026-05-11] update | HttpBaseService centraliza desempaquetado del envelope API

- `desempaquetar<T>()` agregado a `HttpBaseService` — extrae `datos` (o `data`) automáticamente en los 5 métodos HTTP
- Servicios ya no necesitan hacer unwrap manual del envelope `{ exito, datos }`
- `obtenerHistorialSesiones` simplificado: de 6 líneas con casting manual a 2 líneas tipadas
- Napkin actualizado con 3 reglas nuevas: columnas reales SQL, botón mobile obligatorio, envelope centralizado
- Documentado en `venta-inventario-frontend.md`: patrón correcto vs incorrecto

---

## [2026-05-11] ingest | Dos errores corregidos — schema DB y botón mobile ausente

- **Error 1**: migración 006 usó `nombre`/`seccion_permiso_id` en tabla `permisos` y `nombre` en `roles` — columnas incorrectas. Correcto: `permiso`, `seccion_id`, `rol`. Regla documentada en `venta-inventario.md`: siempre leer `database/setup-completo.sql` o `database/autenticacion.sql` antes de escribir SQL.
- **Error 2**: botón "Ver historial" agregado solo en tabla desktop. Mobile (`card-actions`) quedó sin el botón. Regla documentada en `venta-inventario-frontend.md`: todo botón de acción debe aparecer en ambos bloques (desktop table + mobile cards).

---

## [2026-05-11] update | Log de sesiones — tabla, entidad, endpoint, modal

- Nueva tabla `log_sesion`: `usuario_id`, `accion` (LOGIN/LOGOUT/REFRESH), columnas de auditoría. Sin IP ni user agent (decisión de Christian).
- Backend: entidad Sequelize + servicio + módulo `LogSesionesModulo`. `iniciarSesion()` graba LOGIN; `cerrarSesion()` graba LOGOUT.
- Endpoint `GET /autenticacion/usuarios/:id/historial` — retorna últimas 20 entradas, protegido con `SEGURIDAD_USUARIOS_VER`.
- Permiso `SEGURIDAD_USUARIOS_VER_HISTORIAL` agregado a seed SQL (migración 006).
- Frontend: modal `p-dialog` en `usuarios-lista`, tabla con badges de color por acción, loading state, botón en desktop y mobile.

---

## [2026-05-09] ingest | Contexto negocio auto-set para no-SUPERADMIN + banner oculto

- Bug: no-SUPERADMIN veía "Selecciona un negocio" porque el contexto nunca se establecía automáticamente
- Fix: `AutenticacionServicio.obtenerNegocioId()` lee `negocioId` del JWT payload
- Fix: `NegocioServicio.buscarPorId()` corregido — estaba retornando el envelope completo `{ exito, datos }` sin extraer `datos`
- Fix: sidebar llama `autoEstablecerContextoNegocio()` para no-SUPERADMIN al init
- Fix: banner "Contexto activo" envuelto en `@if (esSuperAdmin())` — invisible para otros roles
- Patrón documentado: no usar `&&` con `as` en un solo `@if` — usar `@if` anidados para evitar inferencia de tipo `boolean | Objeto`
- Documentado en `venta-inventario-frontend.md`

## [2026-05-08] ingest | Regla permisos permanente, vista unificada Secciones+Permisos, bug pi-slash, bug password autocomplete

- Regla permanente plasmada en `backend/CLAUDE.md` y `frontend/CLAUDE.md`: cada módulo nuevo entrega SQL de permisos + `@Permisos()` + `*appPermiso` automáticamente
- Vista `/dashboard/secciones-permisos` creada: accordion tabla + sub-tabla de permisos; zero cambios backend — `GET /secciones-permiso` ya retorna `permisos[]` anidados
- ADR creado: `ADR-secciones-permisos-unificado.md`; endpoints huérfanos `GET /permisos` y `GET /permisos/:id` dejados para segunda etapa
- Bug `pi-slash` → `pi-ban`: icono inexistente hacía botones de desactivar invisibles (tooltip pero sin contenido)
- Bug pendiente: contraseña pre-rellena al abrir modal "Nuevo usuario" — probable browser autocomplete; fix: `autocomplete="new-password"`
- Documentado en `venta-inventario-frontend.md`

---

## [2026-05-08] ingest | Guard contexto negocio, form movimientos a modal, bug CSS dialog, fix PrimeNG padding

- Productos/Sedes/Movimientos: guard `sinContexto = computed(() => !negocioActivo())` — si null → limpiar datos + return en `effect()`
- Template: `@if (sinContexto())` como primer branch antes de skeleton/tabla
- Movimientos: formulario inline migrado a `p-dialog`; signal `formularioVisible`, métodos `abrirFormulario()`/`cerrar()`; `DialogModule` en imports
- Al registrar exitoso: `cerrar()` antes del reset del formulario
- Bug documentado: convertir form inline a modal requiere actualizar CSS en el mismo paso — clases `.mov-form` → `.dialog-form`, `.dialog-actions`, `.ghost-btn`, `.form-error`
- Fix PrimeNG: `::ng-deep .p-dialog-content { padding: 1.25rem 1.5rem 0; overflow: visible }` para labels recortadas
- Documentado en `venta-inventario-frontend.md`

---

## [2026-05-07] ingest | Tenant isolation usuarios, reasignación negocio, auto-save permisos rol

- `usuarios.servicio.ts`: `listar` y `crear` filtran/asignan por `negocioId` recibido desde el controlador
- `usuarios.controlador.ts`: usa `@NegocioActual()` para extraer `negocioId` del contexto JWT/interceptor
- `TenantInterceptor`: superadmin sin header `X-Negocio-Context` → `negocioId = 0` (listas vacías); fallback al negocio propio eliminado
- Frontend: lista de usuarios reactiva al cambio de tenant vía `effect({ allowSignalWrites: true })`
- Frontend: bloqueo de creación para superadmin sin contexto (toast warn + return early)
- Backend: nuevo DTO `cambiar-negocio-usuario.dto.ts` + endpoint `PATCH /autenticacion/usuarios/:id/negocio`
- Frontend: botón `pi-building` por fila + modal con `p-dropdown` de negocios (lazy); `appendTo="body"` para z-index
- Backend: `secciones-permiso/rol/:rolId` ahora retorna `rolPermisoId` en cada permiso asignado
- Frontend: interface `PermisoAsignableRol` ampliada con `rolPermisoId?`; flujo de pendientes eliminado; auto-save inmediato; seleccionar/quitar toda sección; spinners individuales por permiso y por sección
- Gotchas documentados en `venta-inventario-frontend.md`: NG0600, `appendTo="body"` en PrimeNG dentro de dialogs
- Decisión de diseño documentada en `venta-inventario-multitenancy.md`: comportamiento superadmin sin contexto

---

## [2026-05-07] update | Venta e Inventario — pantalla negocios, schema permisos, decisiones JWT

- Pantalla CRUD de negocios construida en Angular: servicio `negocio.servicio.ts` en `core/services/`, componente en `features/negocios/negocios-lista/`
- Ítem agregado al sidebar bajo sección "ADMINISTRACION" con permiso `ADMINISTRACION_NEGOCIOS_VER`
- Schema real de tabla `permisos` documentado: columna `permiso` (no `nombre`), FK `seccion_id` (no `seccion_permiso_id`), tabla de asignación `roles_permisos` (no `rol_permisos`)
- SQL correcto de seed de permisos documentado con el patrón de INSERT + asignación a rol
- Documentada decisión de diseño: permisos como strings en JWT (Auth0/AWS/Google usan el mismo estándar; guards y frontend lo usan directamente por nombre)
- Documentada nota sobre BIGINT-as-string en Sequelize y el fix pendiente (`Number()` en `autenticacion.servicio.ts`)
- Advertencia de UX: después de insertar permisos, el usuario debe cerrar sesión — el JWT no se actualiza en caliente

---

## [2026-05-07] update | Venta e Inventario — nivel_minimo expuesto en backend y frontend

- `permiso_secciones.nivel_minimo` ya existía en BD (migración 002, valores 1–4) pero no en el código
- Agregado `nivelMinimo` a: entidad Sequelize, `CrearSeccionPermisoDto` (`@IsInt @Min(1) @Max(4)`), mapeos de `listarConPermisosPorUsuario` y `listarConPermisosPorRol`, interface frontend
- `ActualizarSeccionPermisoDto` hereda automáticamente vía `PartialType`
- Bug corregido: `class-validator` exporta `Min`/`Max`, no `IsMin`/`IsMax` — el error crasheaba el backend bloqueando el login

---

## [2026-05-07] update | Venta e Inventario — Rediseño UI permisos (acordeones)

- `rol-acceso` y `usuario-acceso`: sección "Permisos asignados" eliminada (redundante)
- Rediseño accordion: secciones colapsables, badge activos/total, iconos de color, expandir/colapsar todo
- Label de permiso: descripción en español (principal) + código técnico monospace (secundario)
- Fix CSS: `min-width: 0` + `word-break: break-word` para texto largo en grid
- Documentado patrón `extraerDatos<T>()` y diseño de gestión de permisos en `venta-inventario.md`

---

## [2026-05-06] ingest | Napkin Skill — Memoria persistente de errores por repo

- Instalado `napkin` skill globalmente: `git clone https://github.com/blader/napkin.git ~/.claude/skills/napkin`
- Skill crea `.claude/napkin.md` por repo — runbook curado de errores, correcciones y patrones reutilizables
- Mecanismo: lectura automática vía hook (`SessionStart`), escritura manual por el modelo durante el trabajo
- `SessionStart` hook actualizado en `~/.claude/settings.json` — ahora tiene 2 hooks: caveman + napkin
- Creado napkin inicial para proyecto `venta-inventario` en `.claude/napkin.md`
- Actualizada página [[tecnologias/claude-code-skills]] con sección napkin y settings.json correcto

---

## [2026-05-06] ingest | Caveman Skills para Claude Code

- Instalado ecosystem caveman globalmente: `npx skills add JuliusBrussee/caveman --yes --global`
- 8 skills instalados en `~\.agents\skills\`: caveman, caveman-commit, caveman-review, caveman-compress, caveman-stats, caveman-help, cavecrew, compress
- Configurado `SessionStart` hook en `~/.claude/settings.json` para auto-activar caveman mode cada sesión
- Aprendizaje clave: `npx skills add` instala skills pero NO hooks JS — `/caveman-stats` y badge de statusline no funcionan sin el instalador completo (`install.ps1`)
- Creada página [[tecnologias/claude-code-skills]]

---

## [2026-05-06] ingest | Venta e Inventario + Prototipo Autenticación

- Ingestión profunda de `Venta e inventario` (Adifnex): backend NestJS + Sequelize, frontend Angular 18, Android Kotlin
- Ingestión profunda de `prototipo-de-proyecto-autenticacion`: implementación completa JWT + Refresh Tokens + RBAC
- Creada página [[tecnologias/patron-autenticacion-jwt]] con el patrón documentado
- Descubierto: el proyecto usa Sequelize (no TypeORM) — excepción al stack estándar
- Descubierto: la empresa/marca del proyecto Android es **Adifnex**
- Descubierto: el patrón de autenticación es consistente entre ambos proyectos
- Páginas actualizadas: venta-inventario.md, autenticacion-prototipo.md
- Páginas creadas: patron-autenticacion-jwt.md

---

## [2026-05-06] init | Inicialización del sistema Viernes

- Se crea la arquitectura completa del wiki
- Se define el esquema en `CLAUDE.md`
- Se exploran 14 proyectos en `C:\Users\Christian\Proyectos`
- Se crean páginas iniciales para todos los proyectos identificados
- Se inicializan `index.md` y `log.md`

**Proyectos identificados en exploración inicial:**

- comercio-centro-lima (fullstack Angular + Node)
- driver-acompanamiento (Android Kotlin)
- ecommerce (fullstack Angular + Node)
- landing-page-christina (HTML estático)
- museo (fullstack Angular + Node)
- metropolitano (fullstack Angular + Node — sistema de flota)
- autenticacion-prototipo (fullstack Angular + Node)
- proyecto-spa (fullstack Angular + Node)
- soporte (fullstack Angular + Node + n8n)
- super-mercado (Python + FastAPI + Angular + n8n + Playwright)
- venta-inventario (fullstack Angular + Node + Android)
- angular-aprendizaje (proyectos de estudio)

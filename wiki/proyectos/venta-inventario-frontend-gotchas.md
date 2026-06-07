---
titulo: Frontend — Gotchas Angular/CSS/Material
tipo: proyecto
tags: [inventario, angular, angular-material, css, gotchas, bugs]
fecha_creacion: 2026-06-07
fecha_actualizacion: 2026-06-07
estado: activo
---

# Gotchas Angular / CSS / Material

Sub-página de [[venta-inventario-frontend]]. Colección de errores no obvios y sus fixes.

## NG0600 — escritura de signal dentro de `effect`

Angular 18 lanza `NG0600` si se llama `.set()` dentro de un `effect()` sin configuración explícita.

```typescript
effect(() => { /* ... */ }, { allowSignalWrites: true });
```

## `p-dropdown` dentro de `p-dialog` — panel oculto por z-index

El panel queda detrás del overlay del dialog. Fix obligatorio para cualquier PrimeNG dropdown dentro de un dialog:

```html
<p-dropdown appendTo="body" ... />
```

Aplica también a `p-select`, `p-multiSelect`, `p-calendar` y cualquier componente con panel flotante.

## PrimeNG dialog — labels recortadas

Cuando el contenido del dialog tiene poco espacio superior, los labels quedan recortados.

```css
:host ::ng-deep .p-dialog-content { padding: 1.25rem 1.5rem 0; overflow: visible; }
```

## Bug: `pi-slash` no existe en PrimeIcons

`<i class="pi pi-slash">` renderiza el botón sin contenido visible (solo tooltip al hover).
Fix: reemplazar `pi-slash` → `pi-ban`. Para desactivar: `pi-ban`, `pi-times`, `pi-eye-slash`.

## Bug: double-unwrap en servicios de inventario (2026-05-11)

`HttpBaseService` YA extrae `datos`. Al tipear `get<RespuestaApi<T>>` y hacer `.map(r => r.datos)` se obtiene `undefined` porque el campo ya fue extraído.

```typescript
// ✗ Incorrecto
return this.http.get<RespuestaApi<Producto[]>>('api/...').pipe(map(r => r.datos ?? []));

// ✓ Correcto
return this.http.get<Producto[]>('api/inventario/productos', { params });
```

## OnPush + propiedades planas no disparan change detection (2026-05-31)

**Síntoma**: botón submit queda en `cargando` después de un error HTTP.
**Causa**: `ChangeDetectionStrategy.OnPush` + propiedades planas — las asignaciones en callbacks HTTP no disparan CD.

```typescript
// ✗ Incorrecto
enviando = false;

// ✓ Correcto
readonly enviando = signal(false);
```

**Regla**: en componentes `OnPush`, todo estado mutable debe ser Signal o `async` pipe.

## Material Icons vs Material Symbols — fuentes distintas (2026-06-03)

| Familia | URL | Uso |
|---------|-----|-----|
| Material Icons (clásica) | `?family=Material+Icons` | `<mat-icon>` — ligatures |
| Material Symbols (moderna) | `?family=Material+Symbols+Outlined` | Requiere clase `material-symbols-outlined`, NO compatible con `<mat-icon>` |

**Error cometido**: `index.html` cargaba `Material+Symbols+Outlined` → los iconos aparecían como texto ("da" en vez del ícono dashboard).

```html
<!-- ✓ Correcto para <mat-icon> -->
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
```

## CSS scoped de Angular pisa Tailwind responsive (2026-06-03)

**Síntoma**: `md:hidden` y `hidden md:flex` no funcionan cuando el componente define `display` en esa clase.
**Causa**: Angular scoped styles tienen prioridad sobre el CSS global de Tailwind.

**Opción A** — media queries explícitos en el CSS del componente (preferida para tablas):
```css
@media (min-width: 768px) { .mobile-wrapper { display: none; } }
@media (max-width: 767px) { .table-wrapper { display: none; } }
```

**Opción B** — `@if` con señal JS (preferida para elementos únicos):
```typescript
esMovil = window.innerWidth < 1024; // + HostListener resize
```

## `mat-icon` hereda `font-size` del botón padre (2026-06-03)

`mat-icon` usa `font-size` CSS para su tamaño. Si el botón padre tiene `font-size: .8rem`, el icon hereda ~13px.

```css
.row-actions .icon mat-icon { font-size: 17px; width: 17px; height: 17px; }
```

**Regla**: en botones de acción con `<mat-icon>`, siempre especificar `font-size`, `width` y `height` en px.

## @angular/material no instalado en Docker (2026-06-03)

**Síntoma**: `Cannot find module '@angular/material/*'`.
**Causa**: volumen anónimo `/app/node_modules` retiene `node_modules` viejo al agregar un paquete después del build.

```bash
docker-compose -f docker-compose.dev.yml up --build frontend
# o dentro del contenedor:
docker-compose -f docker-compose.dev.yml exec frontend npm install
```

## `display:flex` en `<td>` rompe layout de tabla (2026-05-14)

Aplicar `display: flex` directamente en `<td>` hace que la celda deje de comportarse como tabla.

```html
<!-- ✓ Correcto: flex en el div wrapper, no en td -->
<td><div class="audit-cell">...</div></td>
```

**Regla**: CSS de layout solo en `div`/`span` dentro de celdas, nunca en `<td>` o `<th>`.

## Click fuera backdrop no debe cerrar modales con formularios

Eliminado `(click)="cerrar()"` del overlay en modales con formularios (especialmente en mobile, donde un toque accidental fuera puede borrar datos). El modal solo se cierra con botón X o Cancelar.

## Bug: dropdown perfil no se abría (sesión 42, 2026-06-06)

**Causa**: `@HostListener('document:click')` cierra el menú en cualquier click. El click en el perfil ejecuta `alternarMenuCuenta()` (pone en `true`) y el mismo evento burbujea al documento bajándolo a `false`.

```html
<!-- Fix: stopPropagation en el elemento que abre -->
(click)="$event.stopPropagation(); alternarMenuCuenta()"
```

**Regla**: cuando se usa `@HostListener('document:click')` para cerrar dropdowns, todos los elementos que abran un dropdown deben tener `$event.stopPropagation()`.

## `class-validator` — decoradores de rango

Los decoradores correctos son `@Min(n)` y `@Max(n)`. **`@IsMin` y `@IsMax` no existen** — crashean el servidor en runtime sin error de compilación.

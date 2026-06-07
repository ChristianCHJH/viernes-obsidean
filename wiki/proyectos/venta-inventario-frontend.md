---
titulo: Venta e Inventario — Frontend Angular
tipo: proyecto
tags: [inventario, angular, primeng, angular-material, tailwind, ux, registro, guardia-negocio, migracion, ui-premium]
fecha_creacion: 2026-05-07
fecha_actualizacion: 2026-06-07 (sesión 46)
estado: activo
---

# Venta e Inventario — Frontend Angular

Sub-página de [[venta-inventario]].

---

## Migración PrimeNG → Angular Material — Estado actual (2026-06-02)

**Decisión**: reemplazar PrimeNG 17 completamente por Angular Material 18.x.

**Versión compatible**: Angular 18.2.0 → `@angular/material@18` (paridad exacta de versión mayor).

**Dimensión del trabajo**:
- 32 archivos `.html` a migrar
- 23 archivos `.ts` con imports PrimeNG
- 19 módulos PrimeNG distintos
- 15 componentes PrimeNG únicos
- 2 servicios PrimeNG a reemplazar: `MessageService` → `SnackBarServicio`, `ConfirmationService` → `MatDialog`

### Equivalencias PrimeNG → Angular Material

| PrimeNG | Angular Material | Módulo |
|---------|-----------------|--------|
| `p-button` / `pButton` | `mat-button` / `mat-raised-button` | `MatButtonModule` |
| `p-inputText` | `<input matInput>` en `mat-form-field` | `MatInputModule` |
| `p-inputTextarea` | `<textarea matInput>` en `mat-form-field` | `MatInputModule` |
| `p-inputNumber` | `<input matInput type="number">` | `MatInputModule` |
| `p-dropdown` | `<mat-select>` en `mat-form-field` | `MatSelectModule` |
| `p-multiSelect` | `<mat-select multiple>` | `MatSelectModule` |
| `p-checkbox` | `<mat-checkbox>` | `MatCheckboxModule` |
| `p-inputSwitch` | `<mat-slide-toggle>` | `MatSlideToggleModule` |
| `p-selectButton` | `<mat-button-toggle-group>` | `MatButtonToggleModule` |
| `p-dialog` | `MatDialog` (servicio) + componente | `MatDialogModule` |
| `p-confirmDialog` + `ConfirmationService` | `ConfirmDialogComponent` custom vía `MatDialog` | `MatDialogModule` |
| `p-toast` + `MessageService` | `SnackBarServicio` wrapper de `MatSnackBar` | `MatSnackBarModule` |
| `p-tabView` / `p-tabPanel` | `mat-tab-group` / `mat-tab` | `MatTabsModule` |
| `p-skeleton` | Tailwind `animate-pulse` custom | — |
| `p-progressSpinner` | `<mat-spinner>` | `MatProgressSpinnerModule` |
| `p-avatar` | `<img class="rounded-full w-8 h-8">` | — |
| `p-tag` | `<mat-chip>` | `MatChipsModule` |
| `pRipple` | `matRipple` | `MatRippleModule` |
| `pTooltip` | `matTooltip` | `MatTooltipModule` |
| PrimeIcons `pi pi-*` | `<mat-icon>nombre</mat-icon>` | `MatIconModule` |

### Fases (7 sesiones)

| Sesión | Fase | Qué se entrega | Estado |
|--------|------|---------------|--------|
| 1 | Fase 0 — Setup | `npm uninstall primeng primeflex primeicons`, instalar Material 18, crear `MaterialModule`, `ConfirmDialogComponent`, `SnackBarServicio` | ✅ Completado 2026-06-02 |
| 2 | Fase 1 | 6 componentes autenticación | ✅ Completado 2026-06-02 |
| 3 | Fase 2 | `barra-lateral`, `boton`, `entrada` | ✅ Completado 2026-06-02 |
| 4 | Fase 3 | Panel y dashboard | ✅ Completado 2026-06-02 |
| 5 | Fase 4 | 10 componentes CRUD administración | ✅ Completado 2026-06-02 |
| 6 | Fase 5 | 7 componentes inventario | ✅ Completado 2026-06-03 |
| 7 | Fase 6 | `mi-pagina` (catálogo) | ✅ Completado 2026-06-03 |

**Estado Fase 0** (sesión 23, 2026-06-02):
- `primeng`, `primeflex`, `primeicons` — desinstalados
- `@angular/material@18.2.14` — instalado (`@angular/cdk` ya estaba como dep)
- `provideAnimations()` — ya existía en `app.config.ts`, no requirió cambio
- `angular.json` — reemplazado tema PrimeNG por `azure-blue.css` de Material
- `styles.css` — variables PrimeNG eliminadas, `--spa-*` conservadas, clases snack-bar (`snack-exito/error/aviso/info`) para override del `mdc-snackbar`
- `index.html` — agregada font `Material Symbols Outlined`
- `shared.module.ts` — PrimeNG eliminado, `MaterialModule` agregado
- Creados: `shared/material.module.ts`, `shared/components/confirm-dialog/confirm-dialog.component.ts`, `core/services/snack-bar.servicio.ts`

**Tema elegido**: `azure-blue.css` — más cercano a `--spa-primary: #2b7ea6`. Override de color primario vía CSS en `styles.css`.

**Estado app tras Fase 0**: compila con errores (componentes aún referencian `pi pi-*` y directivas PrimeNG). Normal — se resuelve en Fases 1-6.

### Archivos nuevos a crear (Fase 0)

- `shared/material.module.ts` — exporta todos los módulos Material
- `shared/components/confirm-dialog/confirm-dialog.component.ts` — recibe `{ titulo, mensaje, textoConfirmar }` por `MAT_DIALOG_DATA`
- `core/services/snack-bar.servicio.ts` — wrapper con API `exito()`, `error()`, `advertencia()`, `info()`

### Notas críticas

- `PrimeFlex` se elimina — sus clases `p-col-*` se reemplazan con Tailwind (ya instalado)
- `PrimeIcons` se elimina — todos los `pi pi-*` → `<mat-icon>`
- `mat-form-field` obligatorio para cada `matInput` / `mat-select` — más verbose pero más semántico
- `MatDialog` exige componente formulario separado (standalone); `MatDialogRef` maneja el cierre
- Plan HTML en `docs/planes/plan-migracion-primeng-material.html`

---

## API envelope — desempaquetado centralizado en `HttpBaseService`

El backend retorna `{ exito: true, mensaje: "...", datos: T }`. El desempaquetado está centralizado en `HttpBaseService.desempaquetar<T>()` — los servicios reciben `T` directamente, nunca el envelope.

```typescript
// http-base.service.ts — aplica en get/post/patch/put/delete
private desempaquetar<T>(respuesta: unknown): T {
  if (respuesta && typeof respuesta === 'object' && !Array.isArray(respuesta)) {
    const r = respuesta as Record<string, unknown>;
    if ('datos' in r) return r['datos'] as T;
    if ('data' in r) return r['data'] as T;
  }
  return respuesta as T;
}
```

**Regla**: nunca hacer `respuesta?.['datos']` ni `respuesta?.datos` en un servicio. El servicio declara el tipo real en `get<T>()` y recibe `T` directo.

```typescript
// ✓ correcto
return this.http.get<EntradaLogSesion[]>(`api/.../historial`)
  .pipe(map((lista) => lista.map(this.mapear.bind(this))));

// ✗ incorrecto — nunca más
return this.http.get<unknown>(`api/.../historial`)
  .pipe(map((r) => (r as any)?.['datos'] ?? []));
```

`extraerDatos<T>()` en `api-mapper.util.ts` sigue existiendo como utilidad para casos edge (respuestas fuera del patrón estándar), pero no debe usarse en el flujo normal.

## Sidebar y permisos

El sidebar filtra los ítems de navegación según los permisos presentes en el JWT. No hace llamadas adicionales al backend para verificar acceso — todo viene del token.

## `class-validator` — decoradores de rango

Los decoradores correctos son `@Min(n)` y `@Max(n)`. **`@IsMin` y `@IsMax` no existen** — si se usan, crashean el servidor en runtime sin error de compilación.

---

## Tenant isolation — lista de usuarios reactiva

`usuarios-lista.component.ts` usa un `effect` de Angular para recargar datos cuando cambia el negocio activo:

```typescript
effect(() => {
  this.negocioContexto.negocioActivo();
  this.cargarUsuarios();
}, { allowSignalWrites: true });
```

Cuando el superadmin no tiene contexto seleccionado, se bloquea la creación: se muestra un toast `warn` y se hace `return` early antes de abrir el modal.

## Patrón preferido para recarga por contexto: `toObservable + switchMap`

Cuando un componente carga datos via HTTP y debe reaccionar al cambio de negocio, **usar `toObservable + switchMap`** en lugar de `effect(() => cargar())`. Razón: `switchMap` cancela la petición HTTP anterior automáticamente — evita race conditions cuando el usuario cambia negocio rápido.

```typescript
import { toObservable } from '@angular/core/rxjs-interop';
import { filter, switchMap } from 'rxjs';

// En ngOnInit() (no en constructor — necesita inyecciones completadas):
toObservable(this.negocioCtx.negocioActivo)
  .pipe(
    filter(n => n !== null),          // ignorar estado sin contexto
    switchMap(() => {
      this.cargando.set(true);
      return this.servicio.obtener(); // cancela la llamada anterior si llega nuevo valor
    }),
    takeUntilDestroyed(this.destroyRef),
  )
  .subscribe({
    next: (datos) => { /* aplicar datos, cargando.set(false) */ },
    error: () => { /* mensaje error, cargando.set(false) */ },
  });
```

**Cuándo usar cada patrón:**

| Escenario | Patrón |
| --- | --- |
| Componente con llamada HTTP que debe cancelar peticiones previas | `toObservable + switchMap` ← preferido |
| Acción simple sin HTTP (limpiar array, resetear estado) | `effect()` |
| Guardia "sin contexto → mostrar vacío" | `computed(() => !this.negocioCtx.negocioActivo())` |

**Bug resuelto (2026-05-18)**: `MiPaginaComponent` usaba `ngOnInit() { this.cargar() }` — no se suscribía al signal de contexto. Al cambiar negocio, el formulario mantenía los datos del negocio anterior. Fix: `toObservable + switchMap` en `ngOnInit`.

## Modal de reasignación de negocio

- Botón con icono `pi-building` visible por fila, solo para `esSuperAdmin()`.
- Abre un `p-dialog` con un `p-dropdown` que carga la lista de negocios de forma lazy.
- El dropdown usa `appendTo="body"` para evitar que el panel quede detrás del overlay del dialog (ver gotcha #2 abajo).
- Llama a `PATCH /autenticacion/usuarios/:id/negocio` con `{ negocioId }`.

## Permisos de rol — auto-save y seleccionar todo

Flujo eliminado: ya no existe el Set `permisosPendientes` ni el botón "Guardar cambios".

Flujo actual:

- **Checkbox individual** → `togglePermisoInmediato()`: si no asignado, POST batch con 1 ID; si asignado, DELETE con `rolPermisoId`.
- **Botón de sección** → `seleccionarTodaSeccion()`: si hay permisos sin asignar → batch assign; si todos asignados → `forkJoin` de DELETEs en paralelo.
- Spinners individuales: `guardandoPermisoId` (por permiso) y `guardandoSeccionId` (por sección).

El backend ahora expone `rolPermisoId` (ID de `roles_permisos`) en `secciones-permiso/rol/:rolId`. Sin este campo, el frontend no podía hacer DELETE directo.

Interface actualizada:

```typescript
interface PermisoAsignableRol {
  // ...campos previos...
  rolPermisoId?: number | string | null;
}
```

---

## Gotchas Angular / PrimeNG

### NG0600 — escritura de signal dentro de `effect`

Angular 18 lanza `NG0600` si se llama a `.set()` o se escribe en un signal dentro de un `effect()` sin configuración explícita.

Solución:

```typescript
effect(() => { /* ... */ }, { allowSignalWrites: true });
```

### `p-dropdown` dentro de `p-dialog` — panel oculto por z-index

El panel del dropdown queda detrás del overlay del dialog. Solución obligatoria para cualquier PrimeNG dropdown o select dentro de un dialog:

```html
<p-dropdown appendTo="body" ... />
```

Aplica también a `p-select`, `p-multiSelect`, `p-calendar` y cualquier componente con panel flotante.

### PrimeNG dialog — labels recortadas por padding insuficiente

Cuando el contenido del dialog tiene poco espacio superior, los labels de los campos quedan recortados o muy apretados.

Fix:

```css
:host ::ng-deep .p-dialog-content {
  padding: 1.25rem 1.5rem 0;
  overflow: visible;
}
```

---

## Guard de contexto de negocio en módulos de inventario

Los módulos **Productos**, **Sedes** y **Movimientos** deben verificar `negocioActivo` antes de cargar datos. Si el contexto es `null`, limpian su estado y muestran vacío en lugar de hacer peticiones al backend.

### Patrón TypeScript

```typescript
sinContexto = computed(() => !this.negocioContexto.negocioActivo());
```

En el `effect()` de carga:

```typescript
effect(() => {
  const negocio = this.negocioContexto.negocioActivo();
  if (!negocio) {
    this.datos.set([]);
    this.cargando.set(false);
    return;
  }
  this.cargarDatos();
}, { allowSignalWrites: true });
```

### Patrón en template

`@if (sinContexto())` debe ser el **primer branch**, antes del skeleton y de la tabla:

```html
@if (sinContexto()) {
  <!-- estado vacío: "Selecciona un negocio para ver..." -->
} @else if (cargando()) {
  <!-- skeleton -->
} @else {
  <!-- tabla / contenido -->
}
```

---

## Movimientos — formulario inline convertido a modal

El historial de movimientos tenía el formulario de registro debajo de la tabla (inline). Se migró a un `p-dialog` activado desde el header.

### Estructura

- Botón "Nuevo movimiento" en el header, protegido con `*appPermiso`.
- Signal `formularioVisible = signal(false)`.
- Métodos `abrirFormulario()` y `cerrar()`.
- Al registrar exitoso: llamar a `cerrar()` **antes** del reset de formulario.
- `DialogModule` agregado al array de `imports` del componente.

### Regla obligatoria: convertir form inline a modal

Al mover un formulario a un dialog, siempre actualizar el CSS en el mismo paso. Las clases cambian:

| Antes (inline) | Después (modal) |
| -------------- | --------------- |
| `.mov-form` | `.dialog-form` |
| `.mov-actions` | `.dialog-actions` |
| — | `.ghost-btn` |
| — | `.form-error` |

**Bug documentado**: se actualizó el HTML a `.dialog-form` pero no el CSS del componente — las clases quedaron sin estilos. El fix fue agregar `.dialog-form`, `.dialog-actions`, `.ghost-btn` y `.form-error` al archivo CSS.

---

## Regla permanente: permisos en cada módulo nuevo

Plasmado en `backend/CLAUDE.md` y `frontend/CLAUDE.md` del proyecto para que sea automático en cada sesión:

- **Backend**: al crear cualquier módulo, agregar bloque SQL de INSERT en `003-seed-permisos.sql` + decorator `@Permisos()` en cada endpoint del controlador.
- **Frontend**: cada botón/acción protegida debe llevar `*appPermiso="'PERMISO_CLAVE'"`. Formato obligatorio: `SECCION_ACCION` en mayúsculas.
- Ambas cosas van juntas — no se entrega frontend sin el SQL correspondiente.

---

## Vista unificada Secciones + Permisos

**Decisión**: dos páginas separadas (`/sections` y `/permissions`) unificadas en una sola vista accordion en `/dashboard/secciones-permisos`.

**ADR** en raíz del proyecto: `ADR-secciones-permisos-unificado.md`.

### Por qué cero cambios en backend

`GET /autenticacion/secciones-permiso` ya devuelve `permisos[]` anidados dentro de cada sección — el endpoint existente resuelve todo. Solo faltaba declarar la interface en el servicio Angular:

```typescript
export interface SeccionPermisoItem {
  id: number;
  permiso: string;
  descripcion?: string | null;
  estado: boolean;
}
// campo agregado a SeccionPermiso:
permisos?: SeccionPermisoItem[];
```

### Endpoints huérfanos (segunda etapa)

`GET /autenticacion/permisos` y `GET /autenticacion/permisos/:id` quedaron sin consumidor frontend. Solo los usaba `PermisosListaComponent` (eliminado). No se borran aún — están reservados para limpieza en segunda etapa.

**Componente legacy pendiente de eliminar**: `features/secciones/secciones-permiso-lista/` — aún tiene PrimeNG (`DialogModule`, `ToastModule`, `ConfirmDialogModule`, `MessageService`, `ConfirmationService`). No está en ninguna ruta activa. Candidato a borrar en limpieza post-migración.

### Componente activo

`features/secciones/secciones-con-permisos-lista/` — tres archivos (`.ts`, `.html`, `.css`). Migrado a Material en Fase 4.

- Tabla principal con chevron expandible por fila (`expand_more` / `expand_less` dinámico)
- Sub-tabla de permisos con `border-left: 4px solid var(--spa-primary)`
- CRUD completo para secciones y permisos vía modal-overlay custom
- `ConfirmDialogComponent` (Material) para cambios de estado
- `*appPermiso` en todos los botones

---

## Bug: `pi-slash` no existe en PrimeIcons

Los botones "Desactivar sección" y "Desactivar permiso" usaban `<i class="pi pi-slash">` — icono inexistente. El botón rendereaba pero sin contenido visible (solo aparecía el tooltip al hacer hover).

**Fix**: reemplazar `pi-slash` → `pi-ban` en ambos botones.

**Regla**: antes de usar un icono PrimeIcons, verificar que exista. Los más comunes para "desactivar": `pi-ban`, `pi-times`, `pi-eye-slash`.

---

---

## Contexto de negocio — auto-set para no-SUPERADMIN

### El problema

Los módulos Productos, Sedes y Movimientos muestran "Selecciona un negocio en el contexto" cuando `negocioContexto.negocioActivo()` es `null`. Para usuarios SUPERADMIN esto es correcto (el dropdown del sidebar deja elegir). Para cualquier otro rol, el contexto nunca se establecía porque el dropdown ni siquiera se muestra.

### La solución

El JWT payload ya incluye `negocioId` (campo en `PayloadJwt` del backend). El flujo correcto:

- **SUPERADMIN**: dropdown visible en sidebar, selecciona manualmente. Banner "Contexto activo" visible en panel.
- **No-SUPERADMIN**: al init del sidebar, se lee `negocioId` del JWT, se hace `GET /api/negocio/:id`, y se llama `negocioContexto.establecer()` automáticamente. Ningún banner visible.

### Archivos modificados

**`autenticacion.servicio.ts`** — nuevo método:

```typescript
obtenerNegocioId(): number | null {
  const token = this.obtenerTokenAcceso();
  if (!token) return null;
  try {
    const payload = this.decodificarPayloadJwt(token);
    const id = payload?.['negocioId'];
    return id !== undefined && id !== null ? Number(id) : null;
  } catch {
    return null;
  }
}
```

**`negocio.servicio.ts`** — `buscarPorId` corregido para extraer `datos`:

```typescript
buscarPorId(id: number) {
  return this.http.get<unknown>(`api/negocio/${id}`).pipe(
    map((r) => {
      if (r && typeof r === 'object' && 'datos' in (r as object)) {
        return (r as { datos: Negocio }).datos;
      }
      return r as Negocio;
    })
  );
}
```

> **Nota**: `buscarPorId` estaba tipado como `Observable<Negocio>` pero retornaba el envelope completo `{ exito, mensaje, datos }`. El `map` ahora extrae correctamente.

**`barra-lateral.component.ts`** — rama `else` en `ngOnInit`:

```typescript
if (this.esSuperAdmin()) {
  this.cargarNegocios();
} else {
  this.autoEstablecerContextoNegocio();
}

private autoEstablecerContextoNegocio(): void {
  const negocioId = this.autenticacion.obtenerNegocioId();
  if (!negocioId) return;
  this.negocioServicio.buscarPorId(negocioId).subscribe({
    next: (negocio) => {
      this.negocioContexto.establecer({ id: negocio.id, nombre: negocio.nombre });
    },
    error: () => {}
  });
}
```

**`panel.component.ts` + `.html`** — banner solo para SUPERADMIN:

```typescript
readonly esSuperAdmin = computed(() => this.autenticacion.obtenerRoles().includes('SUPERADMIN'));
```

```html
@if (esSuperAdmin()) {
  @if (negocioContexto.negocioActivo(); as negocio) {
    <div class="context-banner">...</div>
  }
}
```

> **Patrón**: no usar `&&` con `as` en un solo `@if` — Angular infiere tipo `boolean | Objeto` y TypeScript puede reclamar al acceder a `.nombre`. Usar `@if` anidados.

---

## Regla: siempre agregar botones en versión mobile

Cada botón de acción que se agrega en la tabla desktop (`hidden md:block`) **también debe agregarse** en las tarjetas mobile (`block md:hidden`). Son dos bloques independientes en el mismo template.

**Error cometido (2026-05-11)**: se agregó el botón "Ver historial" solo en la versión desktop. La versión mobile quedó sin él hasta corrección posterior.

**Checklist al agregar un botón de acción:**

1. Tabla desktop → fila `<td class="row-actions">`
2. Tarjeta mobile → div `<div class="card-actions">`

---

## Selector inline de nivelMinimo en permisos

**Fecha:** 2026-05-11

Reemplaza el badge estático de nivel por 4 pills clickeables (Bás./Prof./Emp./Dev.) que hacen PATCH inmediato sin abrir dialog.

### Señales de estado

```typescript
readonly nivelCargando = signal<Set<number>>(new Set());       // por permiso individual
readonly seccionNivelCargando = signal<Set<number>>(new Set()); // por sección completa
```

### Cambio individual (`cambiarNivelPermiso`)

- Agrega `permisoId` al Set → deshabilita los 4 pills de esa fila
- PATCH a `/api/autenticacion/permisos/:id`
- `finalize`: quita el ID del Set (siempre, aunque falle)
- `next`: parchea el signal `secciones` in-place → cero reload

### Aplicar a toda la sección (`cambiarNivelSeccion`)

```typescript
forkJoin(permisos.map(p =>
  this.servicioPermisos.actualizarPermiso(p.id, { nivelMinimo: nivel, usuarioActualizacion: 1 })
))
```

- Marca `seccionNivelCargando` con el `seccionId` Y marca todos los `permisoId` en `nivelCargando`
- `forkJoin` dispara todos los PATCHes en paralelo
- `finalize`: limpia ambos Sets
- `next`: parchea todos los permisos de esa sección en el signal local

### Template — header de sub-tabla

```html
<th>
  <div class="nivel-th-header">
    <span>Aplicar nivel a todos:</span>
    <div class="nivel-selector">
      @for (n of [1, 2, 3, 4]; track n) {
        <button
          class="nivel-btn nivel-{{ n }}"
          [disabled]="seccionNivelCargando().has(s.id)"
          (click)="cambiarNivelSeccion(s.id, n)"
        >{{ labelNivelCorto(n) }}</button>
      }
    </div>
  </div>
</th>
```

> `s` está en scope porque el `<thead>` está dentro del `@for (s of seccionesFiltradas())` → `@if (estaExpandida(s.id))`.

### CSS pills

```css
.nivel-btn.active.nivel-1 { background: #e0f2fe; color: #0369a1; border-color: #7dd3fc; }
.nivel-btn.active.nivel-2 { background: #d1fae5; color: #065f46; border-color: #6ee7b7; }
.nivel-btn.active.nivel-3 { background: #fef3c7; color: #92400e; border-color: #fcd34d; }
.nivel-btn.active.nivel-4 { background: #ede9fe; color: #5b21b6; border-color: #c4b5fd; }
```

---

## Bug: double-unwrap en servicios de inventario

**Fecha:** 2026-05-11

Tres servicios (`productos-inventario`, `sedes-inventario`, `stock-inventario`) tenían:

```typescript
// ✗ Incorrecto — envolvían la respuesta en RespuestaApi<T> y luego hacían .map(r => r.datos)
return this.http.get<RespuestaApi<Producto[]>>('api/...').pipe(map(r => r.datos ?? []));
```

`HttpBaseService` YA extrae `datos`. Al hacer `.datos` sobre el resultado ya desempaquetado se obtenía `undefined`. Los arrays llegaban vacíos aunque el backend devolvía datos correctos.

```typescript
// ✓ Correcto
return this.http.get<Producto[]>('api/inventario/productos', { params });
```

Regla reforzada: **nunca** usar `RespuestaApi<T>` ni `.map(r => r.datos)` en servicios — el tipo va directo en el genérico de `get<T>()`.

---

---

## Marcas y Etiquetas — páginas de gestión (2026-05-14)

Dos nuevas vistas CRUD standalone bajo `features/inventario/`:

- `marcas-lista/` — tabla + cards mobile, dialog crear/editar, confirm eliminar. Columnas: ID, Nombre, Estado, Acciones.
- `etiquetas-lista/` — igual que marcas pero con color picker (12 colores predefinidos) y dot de color inline en la columna Nombre.

**Servicios extendidos** en `core/services/`:

- `marcas-inventario.servicio.ts` — agregados `actualizar(id, datos)` y `eliminar(id)`
- `etiquetas-inventario.servicio.ts` — ídem

**Sidebar**: ítems `marcas` (`pi-tag`) y `etiquetas` (`pi-tags`) añadidos bajo la sección INVENTARIO, protegidos con `INVENTARIO_MARCAS_VER` y `INVENTARIO_ETIQUETAS_VER`.

**Panel**: `PERMISOS_VISTA` y `tituloPagina()` actualizados; `@case ('marcas')` y `@case ('etiquetas')` añadidos al switch del template.

**Permisos SQL** (migración 011): secciones `INVENTARIO_MARCAS` e `INVENTARIO_ETIQUETAS` con permisos `VER/CREAR/EDITAR/ELIMINAR` para ambas.

---

## Bug: lista de productos renderizaba 1 fila + círculos grises (2026-05-14)

### Causa raíz

`productos.servicio.ts` — método `listar()` — no tenía `include: [Etiqueta]` porque la entidad `Producto` no declaraba la asociación `@BelongsToMany`. Sin asociación, Sequelize ignora el `include` silenciosamente → `p.etiquetas` llega `undefined` en el frontend.

El template tenía `@for (tag of p.etiquetas; track tag.id)` — `undefined` no es iterable → Angular crasheaba silenciosamente la renderización de cada fila afectada → aparecían como círculos grises del skeleton.

### Fixes aplicados

1. **`producto.entidad.ts`** — agregado `@BelongsToMany(() => Etiqueta, () => ProductoEtiqueta)`.
2. **`producto-etiqueta.entidad.ts`** — agregados `@ForeignKey` y `@BelongsTo` en ambas FKs.
3. **`productos.modulo.ts`** — `Etiqueta` registrada en `SequelizeModule.forFeature`.
4. **`productos.servicio.ts`** — `listar()` añade `include: [{ model: Etiqueta, through: { attributes: [] }, required: false }]` y literal subquery `marcaNombre`.
5. **`productos-inventario.servicio.ts` (frontend)** — `etiquetas` cambiado de `Etiqueta[]` a `etiquetas?: Etiqueta[]` (optional).
6. **`productos-lista.component.html`** — guards `(p.etiquetas ?? [])` en bloques desktop y mobile.

### Regla aprendida

Sequelize no puede eager-load con `include: [Model]` si no se declaran `@BelongsToMany`/`@HasMany`/`@BelongsTo` en las entidades. La ausencia de decoradores no produce error — simplemente devuelve el campo ausente sin advertencia.

---

## Bug: SQL migration ON CONFLICT falla si no existe constraint (2026-05-14)

`database/011-marcas-etiquetas.sql` usaba `ON CONFLICT (nombre) DO NOTHING` en inserts de `permiso_secciones` y `permisos`. Falla con `42P10: no hay restricción única o de exclusión que coincida` si la columna no tiene UNIQUE constraint en la DB objetivo.

**Fix**: reemplazar todos los `ON CONFLICT (col) DO NOTHING` con `WHERE NOT EXISTS (SELECT 1 FROM tabla WHERE col = valor)`. Universalmente seguro independiente del estado de constraints en la DB.

---

---

## Tabla productos — rediseño UI compacta (2026-05-14)

### Cambios estructurales

- **SKU + Descripción** fusionados en la columna Nombre: nombre en negrita, SKU badge debajo, descripción truncada debajo. Eliminadas las columnas SKU y Descripción independientes.
- **Creado por / Actualizado por**: cada `<td>` muestra usuario en itálica (`audit-user`) y fecha+hora debajo en texto pequeño (`audit-date`). El `<td>` contiene un `<div class="audit-cell">` — NO aplicar `display:flex` directo en el `td` (ver bug abajo).
- **Tooltips**: `pTooltip` en nombre (si `length > 35`) y siempre en descripción.
- **Fuentes reducidas**: `th` → `.72rem`, `td` → `.82rem`, padding `0.6rem 1rem`.

### Paginación client-side

```typescript
readonly paginaActual = signal(1);
readonly porPagina = 20;
readonly totalPaginas = computed(() => Math.max(1, Math.ceil(this.productosFiltrados().length / this.porPagina)));
readonly productosPaginados = computed(() => {
  const inicio = (this.paginaActual() - 1) * this.porPagina;
  return this.productosFiltrados().slice(inicio, inicio + this.porPagina);
});
```

- Effect adicional que resetea `paginaActual` a 1 cuando cambia `filtro()`.
- Footer con botones Anterior / páginas numeradas con ellipsis / Siguiente.
- **Aplica igual en mobile**: la vista de cards usa `productosPaginados()` — nunca `productosFiltrados()` directamente.

### Scroll sin overflow de página

Cadena de flex necesaria para que el scroll quede dentro de la tabla (no en la página):

```text
:host → display:flex; flex-direction:column; height:100%
.shell → flex:1; min-height:0
.data-card → flex:1; min-height:0; overflow:hidden
.table-wrapper → flex:1; min-height:0; display:flex; flex-direction:column
.table-scroll → flex:1; min-height:0; overflow-y:auto   ← aquí scrollea
<table> → dentro de .table-scroll
<footer> → fuera de .table-scroll, dentro de .table-wrapper (sticky al fondo)
```

### Sorting client-side

```typescript
readonly ordenCampo = signal<'id'|'nombre'|'fechaCreacion'|'fechaActualizacion'>('id');
readonly ordenDir = signal<'asc'|'desc'>('asc');

ordenar(campo): void {
  if (this.ordenCampo() === campo) {
    this.ordenDir.set(this.ordenDir() === 'asc' ? 'desc' : 'asc');
  } else {
    this.ordenCampo.set(campo);
    this.ordenDir.set('asc');
  }
  this.paginaActual.set(1);
}
```

El `productosFiltrados` computed incluye el `[...lista].sort(...)` antes de devolver. Headers clickeables con iconos PrimeIcons: `pi-sort-alt` (neutro), `pi-sort-amount-up-alt` (asc activo), `pi-sort-amount-down` (desc activo). El campo activo se resalta con `color: var(--spa-primary)`.

---

## Bug: `display:flex` directo en `<td>` rompe layout de tabla (2026-05-14)

**Síntoma**: la columna "Creado por" mostraba tanto el usuario creador como el actualizador en la misma celda — dos "admin.spa" por fila.

**Causa**: aplicar `display: flex; flex-direction: column` directamente al elemento `<td>`. El `td` deja de comportarse como celda de tabla y su contenido se mezcla visualmente con el `td` adyacente.

**Fix**: nunca poner `display:flex` en `<td>`. Usar un `<div>` wrapper dentro:

```html
<!-- ✗ Incorrecto -->
<td class="audit-cell">
  <span>...</span>
</td>

<!-- ✓ Correcto -->
<td>
  <div class="audit-cell">   <!-- el flex va aquí -->
    <span>...</span>
  </div>
</td>
```

**Regla**: CSS de layout (`display:flex`, `display:grid`) solo en elementos `div`/`span` dentro de celdas de tabla, nunca en `<td>` o `<th>` directamente.

---

## Bug: mobile solo mostraba N primeros productos (2026-05-14)

**Síntoma**: en vista mobile solo aparecían los primeros 2 productos de 207.

**Causa**: la vista de cards mobile usaba `productosFiltrados()` (lista completa sin paginar) pero el wrapper no tenía `overflow-y:auto` ni altura definida → solo se veían los que cabían en pantalla sin scroll funcional.

**Fix**:

1. Cambiar `@for (p of productosFiltrados())` → `@for (p of productosPaginados())` en el bloque de cards.
2. Agregar footer de paginación idéntico al desktop (con flechas prev/next).
3. `.mobile-wrapper { display:flex; flex-direction:column; flex:1; min-height:0; overflow-y:auto }`.

---

## Bug: etiquetas no se guardaban al editar producto (2026-05-14)

### Causa raíz — doble problema

**Problema 1 — strings en lugar de numbers (frontend):**

`p-multiSelect` con `optionValue="id"` puede devolver los valores como strings en algunos contextos. El payload llegaba al backend con `etiquetaIds: ["1"]` en vez de `[1]`.

Fix frontend — coercionar en `guardar()`:

```typescript
etiquetaIds: (v.etiquetaIds ?? []).map(Number),
marcaId: v.marcaId != null ? Number(v.marcaId) : null,
```

**Problema 2 — `ignoreDuplicates: true` en Sequelize (backend, causa principal):**

El flujo de `actualizar()` era:

1. Soft-delete todas las `producto_etiqueta` existentes → `eliminado = true`
2. `bulkCreate` las nuevas con `{ ignoreDuplicates: true }`

`ignoreDuplicates: true` genera `ON CONFLICT DO NOTHING`. Si el par `(productoId, etiquetaId)` ya existía (con `eliminado = true`), el insert se ignoraba → la etiqueta quedaba con `eliminado = true` para siempre.

**Fix backend**: `updateOnDuplicate` en lugar de `ignoreDuplicates`, con `eliminado: false` explícito:

```typescript
await this.productoEtiquetaModelo.bulkCreate(
  etiquetaIds.map((etiquetaId) => ({
    productoId: id,
    etiquetaId,
    eliminado: false,          // ← crítico
    usuarioCreacion: usuarioId,
    usuarioActualizacion: usuarioId,
  })),
  { updateOnDuplicate: ['eliminado', 'usuarioActualizacion'] }, // ← ON CONFLICT DO UPDATE
);
```

**Aplica también en `crear()`**: si un producto fue eliminado y recreado con las mismas etiquetas, `ignoreDuplicates` también fallaba. Mismo fix aplicado.

**Regla**: en tablas con soft-delete + relación N:M, nunca usar `ignoreDuplicates: true` en `bulkCreate`. Usar `updateOnDuplicate` con `eliminado: false` explícito.

---

---

## Gotcha: OnPush + propiedades planas no disparan change detection (2026-05-31)

**Síntoma**: botón de submit queda en estado `cargando` indefinidamente después de un error HTTP, aunque el error handler asigne `enviando = false`.

**Causa**: componente con `ChangeDetectionStrategy.OnPush` + propiedades planas (`enviando = false`, `errorGeneral = null`). Angular OnPush solo re-renderiza cuando cambian `@Input()`, emite un `async` pipe, o se llama `markForCheck()`. Las asignaciones directas en un callback HTTP no disparan nada.

**Fix**: convertir a Signals.

```typescript
// ✗ Incorrecto — propiedad plana en OnPush
enviando = false;
errorGeneral: string | null = null;

// ✓ Correcto — Signal, dispara CD automáticamente
readonly enviando = signal(false);
readonly errorGeneral = signal<string | null>(null);
```

En el template: `[cargando]="enviando()"`, `@if (errorGeneral())`, `{{ errorGeneral() }}`.

**Regla**: en componentes `OnPush`, TODO el estado mutable debe ser Signal o estar en un observable con `async` pipe. Nunca propiedades planas para estado que cambia tras callbacks.

---

## Pantalla de registro — bugs responsive (2026-05-31)

### Bug 1: grid no colapsa en mobile → formulario comprimido al 65%

**Causa**: `login-shell` mantiene `grid-template-columns: 35fr 65fr` aunque `.visual-column` tenga `display: none`. El grid sigue reservando el 35% aunque la columna esté oculta.

**Fix**: en ≤768px, override explícito:
```css
@media (max-width: 768px) {
  .login-shell {
    grid-template-columns: 1fr; /* ← obligatorio */
  }
}
```

### Bug 2: subtitle comprimido por flex header con justify-content: space-between

**Causa**: `.form-header` es `display: flex; justify-content: space-between`. El link "¿Ya tienes cuenta?" ocupa espacio a la derecha → el div del título queda en un ancho reducido.

**Fix en mobile**:
```css
@media (max-width: 768px) {
  .form-header {
    flex-direction: column;
    gap: 0.5rem;
  }
}
```

### Bug 3: scroll bloqueado en toda la app — causa `overflow: hidden` global

**Causa raíz**: `styles.css` global tiene:
```css
html, body {
  height: 100%;
  overflow: hidden; /* ← bloquea scroll en toda la app */
}
```

Este override existe para que el panel/dashboard no muestre scroll de página (el scroll está dentro de los componentes). Pero afecta a todas las rutas, incluyendo el registro.

**Fix**: el componente de registro maneja su propio scroll en lugar del body:
```css
:host {
  display: block;
  height: 100vh;
  overflow-y: auto; /* el componente scrollea, no el body */
}
```

**Regla**: cuando `body` tiene `overflow: hidden` global, las pantallas full-page sin panel (login, registro, verificar) deben poner `height: 100vh; overflow-y: auto` en `:host`.

---

## Guards de autenticación (2026-05-30)

### `guardiaAutenticacion` — ya existía
`src/app/core/guards/autenticacion.guardia.ts` — verifica que el JWT exista en storage. Si no → redirige a `/`.

### `guardiaNegocio` — nuevo
`src/app/core/guards/negocio.guardia.ts` — verifica que `negocioId` en el JWT no sea null. Si null → redirige a `/negocio/nuevo`.

```typescript
export const guardiaNegocio: CanActivateFn = () => {
  const servicioAutenticacion = inject(AutenticacionServicio);
  const router = inject(Router);
  const negocioId = servicioAutenticacion.obtenerNegocioId();
  return negocioId ? true : router.createUrlTree(['/negocio/nuevo']);
};
```

**Uso en rutas**: `/dashboard` y `/dashboard/:vista` tienen `canActivate: [guardiaAutenticacion, guardiaNegocio]`. La ruta `/negocio/nuevo` solo tiene `guardiaAutenticacion` (accessible antes de tener negocio).

## Flujo de registro en dos pasos — frontend (2026-05-30)

### Paso 1 — `/registro`
Componente simplificado: solo `nombreUsuario`, `correo`, `telefono` (requerido), `contrasena`, `confirmarContrasena`. Sin campos de negocio.

### Paso 2 — `/negocio/nuevo`
Nuevo componente `NegocioRegistroComponent` (`features/negocio/negocio-registro/`). Formulario: `nombre` (requerido), `ruc` (opcional), `correoContacto` (opcional).

Al enviar:
1. POST `/api/autenticacion/registrar-negocio`
2. Backend retorna nuevos tokens con `negocioId` real
3. `persistirTokens()` actualiza storage
4. `inicializarPermisos()` recarga permisos del JWT
5. Navega a `/dashboard`

### `AutenticacionServicio` — métodos relevantes
- `registrar(datos: DatosRegistro)` — `DatosRegistro` ya no incluye campos de negocio
- `registrarNegocio(datos: DatosRegistroNegocio)` — nuevo, persiste tokens automáticamente
- `obtenerNegocioId()` — retorna `number | null` (ya lo hacía, ahora el null tiene semántica real)

---

## Bug pendiente: contraseña pre-rellena en modal "Nuevo usuario"

Al abrir el dialog de crear usuario, el campo contraseña aparece con puntos (valor pre-cargado). Probable causa: autocompletar del navegador (browser autocomplete). Posibles fixes:

1. Agregar `autocomplete="new-password"` al input de contraseña.
2. Resetear el form con `this.formulario.reset()` al abrir el modal.
3. Agregar un `<input type="password" style="display:none">` antes del visible para engañar al browser (hack, no recomendado).

---

## Gotcha: Material Icons vs Material Symbols — fuentes distintas (2026-06-03)

Google ofrece dos familias de iconos independientes con URLs distintas:

| Familia | URL | Uso en Angular Material |
|---------|-----|------------------------|
| **Material Icons** (clásica) | `https://fonts.googleapis.com/icon?family=Material+Icons` | `<mat-icon>` por defecto — sistema de ligatures |
| **Material Symbols** (moderna) | `https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined` | Requiere clase `material-symbols-outlined`, NO compatibe con `<mat-icon>` directo |

**Error cometido**: `index.html` cargaba `Material+Symbols+Outlined` mientras que `<mat-icon>` espera `Material Icons`. Los iconos aparecían como letras/abreviaciones del nombre (ej: `dashboard` → "da").

**Fix**: reemplazar la `<link>` en `index.html`:
```html
<!-- ✗ Incorrecto -->
<link href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined" rel="stylesheet">

<!-- ✓ Correcto para <mat-icon> -->
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
```

---

## Gotcha: CSS scoped de Angular pisa Tailwind responsive utilities (2026-06-03)

**Síntoma**: clases Tailwind `md:hidden` y `hidden md:flex` no funcionan cuando el componente tiene CSS propio que define `display` en esa clase.

**Causa**: Angular scoped styles tienen prioridad sobre el CSS global. Si el componente define `.mobile-wrapper { display: flex }`, el `@media (min-width: 768px) { .md\:hidden { display: none } }` de Tailwind global pierde.

**Regla**: nunca confiar en `md:hidden` / `hidden md:flex` cuando la clase tiene `display` definido en el CSS del componente. Usar uno de estos dos patrones:

**Opción A — Media queries explícitos en el componente CSS** (preferida para tablas):
```css
@media (min-width: 768px) {
  .mobile-wrapper { display: none; }
}
@media (max-width: 767px) {
  .table-wrapper { display: none; }
}
```

**Opción B — `@if` con señal JS** (preferida para elementos únicos):
```typescript
esMovil = window.innerWidth < 1024;
// + HostListener resize para actualizar
```
```html
@if (esMovil) {
  <button class="ghost-button" (click)="cerrarMovil()">...</button>
}
```

**Casos resueltos**:
- Botón cerrar sidebar `x` (`.ghost-button lg:hidden`) → fix con `@if (esMovil)` en `barra-lateral`
- Cards mobile de `productos-lista` (`.mobile-wrapper`) → fix con `@media` en CSS del componente
- Tabla desktop de `productos-lista` (`.table-wrapper`) → mismo fix

---

## Gotcha: `mat-icon` hereda `font-size` del botón padre (2026-06-03)

**Síntoma**: iconos de acción en tabla se ven muy pequeños o ilegibles.

**Causa**: `mat-icon` usa `font-size` CSS para determinar su tamaño. Si el botón padre tiene `font-size: .8rem`, el icon hereda ese valor y se renderiza a ~13px en lugar de los 24px por defecto.

**Fix**: siempre especificar el tamaño del `mat-icon` en px dentro de botones de acción:

```css
.row-actions .icon {
  width: 30px; height: 30px;
  display: inline-flex; align-items: center; justify-content: center;
  /* NO poner font-size aquí — heredaría al mat-icon */
}
.row-actions .icon mat-icon {
  font-size: 17px; width: 17px; height: 17px; /* ← explícito siempre */
}
```

**Regla**: en cualquier botón de acción con `<mat-icon>`, agregar el selector `button mat-icon` con `font-size`, `width` y `height` explícitos en px.

---

## Gotcha: @angular/material no instalado en contenedor Docker (2026-06-03)

**Síntoma**: todos los imports de `@angular/material/*` fallan con `Cannot find module`.

**Causa**: al agregar `@angular/material` al `package.json` después de haber construido la imagen Docker, el volumen anónimo `/app/node_modules` retiene el `node_modules` viejo sin el paquete.

**Fix**: reconstruir la imagen del contenedor:
```bash
docker-compose -f docker-compose.dev.yml up --build frontend
```
O instalar dentro del contenedor corriendo:
```bash
docker-compose -f docker-compose.dev.yml exec frontend npm install
```

**Contexto proyecto**: `docker-compose.dev.yml` usa volúmenes `./frontend:/app` + `/app/node_modules` (anónimo protegido). El segundo volumen protege `node_modules` del host-mount pero también lo congela hasta el próximo build.

---

## Tab Distribución en modal productos-premium (2026-06-06, sesión 34)

4° tab "Distribución" agregado al modal unificado de `productos-premium`. Implementa los tres tipos de movimiento de stock directamente desde el modal del producto.

### Cambio clave: SALIDA ya existía en backend — faltaba en UI

`StockServicio.registrarMovimiento()` ya aceptaba `tipo: 'INGRESO' | 'SALIDA'`. El componente `producto-distribucion` solo exponía INGRESO y TRANSFERENCIA. El tab nuevo expone los tres.

### Layout del tab

```
.cat-modal-body-full (full-width, sin preview-col)
└── .dist-layout (grid 2 columnas → 1 col en mobile <680px)
    ├── .dist-stock-panel   ← tabla stock por sede
    └── .dist-mov-panel     ← formulario movimiento
```

**Regla**: cuando `tabActivo() === 'distribucion'`, el modal usa `cat-modal-body-full` (sin el panel de preview del catálogo) porque es vista operativa, no de edición.

### Selector de tipo — 3 botones con color semántico

| Tipo | Color activo | Endpoint |
|------|-------------|----------|
| INGRESO | fondo `#dcfce7`, texto `#16a34a` verde | `POST /stock/movimiento { tipo: 'INGRESO' }` |
| SALIDA | fondo `#fee2e2`, texto `#dc2626` rojo | `POST /stock/movimiento { tipo: 'SALIDA' }` |
| TRANSFERENCIA | fondo `#eff6ff`, texto `#2563eb` azul | `POST /stock/transferencia` |

### Hint de stock disponible inline

Para SALIDA y TRANSFERENCIA (campo sede origen), se muestra un hint naranja debajo del select:
```
ℹ Disponible: 15 uds
```
Calculado desde `stockSedeSeleccionada()` — computed que cruza `movSedeId/movSedeOrigenId` con `distStockDesglose`. Evita enviar cantidad > stock disponible.

### Signals nuevas del componente

```typescript
// Distribución
readonly distStockDesglose = signal<StockDesglose[]>([]);
readonly distStockTotal    = signal(0);
readonly distCargando      = signal(false);
readonly sedes             = signal<Sede[]>([]);
readonly tipoMov           = signal<'INGRESO' | 'SALIDA' | 'TRANSFERENCIA'>('INGRESO');
readonly movSedeId         = signal<number | null>(null);
readonly movSedeOrigenId   = signal<number | null>(null);
readonly movSedeDestinoId  = signal<number | null>(null);
readonly movCantidad       = signal<number | null>(null);
readonly registrandoMov    = signal(false);
readonly errorMov          = signal<string | null>(null);
readonly exitoMov          = signal<string | null>(null);

// Computeds
readonly sedesConStock       = computed(() => /* filtrar sedes con stock > 0 */);
readonly sedesDestinoTransf  = computed(() => /* excluir sede origen */);
readonly stockSedeSeleccionada = computed(() => /* desglose de la sede elegida */);
```

### Caché de sedes

```typescript
cargarSedes(): void {
  if (this.sedes().length > 0) return; // no recarga si ya están
  // ...
}
```

Las sedes se cargan una vez por sesión de modal. El stock se recarga al abrir el tab Y después de cada operación exitosa.

### Badge en tab button

```html
@if (distStockTotal() > 0) {
  <span class="cat-tab-badge">{{ distStockTotal() }}</span>
}
```

Muestra el total de unidades del producto como badge en el botón del tab — visible al abrir el modal desde cualquier tab.

### Fixes post-implementación (2026-06-06, sesión 35)

#### Fix 1 — Vista previa visible en tab Distribución

**Síntoma**: al cambiar al tab Distribución, el panel "VISTA PREVIA DEL CATÁLOGO" seguía visible debajo del contenido (stock + formulario movimiento).

**Causa raíz**: `.cat-modal-body-full` colapsa el grid a `1fr`, pero `.cat-preview-col` sigue en el DOM — Angular lo renderiza debajo en el flujo de un solo columna.

**Fix**:
```html
@if (tabActivo() !== 'distribucion') {
  <div class="cat-preview-col">
    ...
  </div>
}
```

#### Fix 2 — Texto de cabecera `dist-table` ilegible

**Síntoma**: "SEDE", "CANTIDAD", "DISTRIBUCIÓN" en el `thead` de la tabla de stock no se leían — texto gris sobre fondo verde oscuro.

**Causa raíz**: selector global del componente `thead tr { background: linear-gradient(90deg, #14532D, #166534) }` aplica a TODOS los `thead`, incluyendo el de `.dist-table`. La regla `.dist-table thead { background: var(--p-bg) }` estaba en el elemento `thead`, no en el `tr` — no sobreponía el gradiente. El `color: var(--p-muted)` (#6B7280 gris) era invisible sobre verde oscuro.

**Fix** en `.dist-table th`:
```css
.dist-table th {
  text-align: left; font-size: .75rem; letter-spacing: .06em;
  text-transform: uppercase; color: rgba(255,255,255,.85);
  padding: .6rem .85rem; font-weight: 600;
}
```

**Regla derivada**: en este componente, `thead tr` tiene un gradiente verde oscuro global. Cualquier tabla nueva que necesite texto legible en su cabecera debe sobreescribir `color` a blanco en sus `th`.

---

## Modal unificado productos-premium — 3 tabs (2026-06-06, sesión 33)

`features/inventario/productos-premium/productos-premium.component.*` reescrito para fusionar el antiguo modal de catálogo con los campos del modal de edición de productos, incorporando también la galería de imágenes. El resultado es un único modal de 1060 px con tres pestañas.

### Estructura del modal

```
.cat-overlay (z-index: 9000)
└── .cat-modal (max-width: 1060px)
    ├── header (nombre del producto + X)
    ├── .cat-strip (color acento naranja)
    └── body
        ├── .cat-form
        │   ├── .cat-tabs (flex-shrink: 0, sticky — no scrollea)
        │   └── .cat-tab-content (flex: 1, overflow-y: auto — scrollea)
        └── .cat-preview-col (panel derecho con imagen principal + KPIs)
```

Override obligatorio para que el layout funcione:
```css
.cat-form { overflow: hidden !important; padding: 0 !important; gap: 0 !important; }
```

### Tab "Producto"

| Campo | Implementación |
|-------|---------------|
| Nombre | `<input>` bound a `catNombre` signal |
| SKU | Toggle manual/auto (`.cat-sku-toggle`). Auto → input deshabilitado. Manual → `catSkuManual = true` |
| Stock mínimo | `<input type="number">` → `catStockMinimo` signal |
| Marca | `<select>` nativo con `appearance: none` + SVG arrow via `background-image` data URI |
| Etiquetas | Chips custom clickables — `color-mix(in srgb, var(--eq-color) 18%, transparent)` para fondo. `.cat-etiq-sel` añade `box-shadow` al estado seleccionado |

### Tab "Catálogo"

Campos existentes: `catVisible` (toggle), `catPrecio`, `catPrecioOferta`, `catDescripcion`, `catOrden`.

### Tab "Imágenes"

- Grid `.cat-img-grid` de thumbnails 88×88 px
- Cada card: thumb + badge "Principal" + botones estrella/papelera
- Slot de subida: `<label for="catFileInput">` clickable — NO necesita ViewChild
- Sub-modal de confirmación al subir: `.cat-submodal-overlay` (z-index 9100, sobre el modal principal)
- Sub-modal tiene toggle "Marcar como principal"

### Preview panel

```typescript
readonly imagenPrincipal = computed(() =>
  this.imagenes().find(i => i.esPrincipal) ?? this.imagenes()[0] ?? null
);
```
Si hay imagen: muestra `<img>` real. Si no: avatar de letra con gradiente calculado de `avatarGradient(nombre)`.

### Signals nuevas en el componente

```typescript
// Tab navigation
readonly tabActivo = signal<'producto' | 'catalogo' | 'imagenes'>('catalogo');

// Producto tab
readonly catNombre      = signal('');
readonly catSkuManual   = signal(false);
readonly catSku         = signal('');
readonly catStockMinimo = signal<number | null>(null);
readonly catMarcaId     = signal<number | null>(null);
readonly catEtiquetaIds = signal<number[]>([]);

// Listas con cache
readonly marcas    = signal<Marca[]>([]);
readonly etiquetas = signal<Etiqueta[]>([]);

// Computed
readonly etiquetasSeleccionadas = computed(() =>
  this.etiquetas().filter(e => this.catEtiquetaIds().includes(e.id))
);
readonly marcaNombreSeleccionada = computed(() =>
  this.marcas().find(m => m.id === this.catMarcaId())?.nombre ?? null
);

// Imágenes tab
readonly imagenes            = signal<ProductoImagen[]>([]);
readonly subirDialogoVisible = signal(false);
readonly archivoSeleccionado = signal<File | null>(null);
readonly previewImgUrl       = signal<string | null>(null);
```

### Patrones de diseño aplicados

**forkJoin con cache de listas**:
```typescript
cargarListas() {
  if (this.marcas().length > 0 && this.etiquetas().length > 0) return; // cache hit
  forkJoin([this.servicioMarcas.listar(), this.servicioEtiquetas.listar()])
    .pipe(takeUntilDestroyed(this.destroyRef))
    .subscribe(([marcas, etiquetas]) => { ... });
}
```

**Actualización local tras guardar** (sin recarga):
```typescript
this.productos.update(lista =>
  lista.map(p => p.id === id ? {
    ...p,
    etiquetas: this.etiquetasSeleccionadas(),
    marcaNombre: this.marcaNombreSeleccionada(),
    // ... resto de campos
  } : p)
);
```

**PATCH unificado** — un solo endpoint actualiza todo:
```typescript
const datos: DatosActualizarProducto = {
  nombre, sku, stockMinimo, marcaId, etiquetaIds,
  visibleCatalogo, precio, precioOferta, descripcion, ordenCatalogo,
};
```

---

## UI/UX mejoras productos-lista — sesión 31 (2026-06-04)

Auditoría con `/ui-ux-pro-max` + mejoras aplicadas a `productos-lista.component.css` y `.html` (sin romper funcionalidad existente):

| Elemento | Antes | Después |
|----------|-------|---------|
| Botón "Nuevo producto" | Azul plano, opacity hover | Gradiente `#2b7ea6→#3490bc` + sombra + `translateY(-2px)` hover |
| Search bar | Bordes rectos | Píldora `border-radius: 50px` + focus glow indigo |
| "Total: N productos" | Texto gris plano | Badge pill con tinte primario |
| Row hover | Solo fondo gris | Fondo tintado + `inset 3px 0 0 var(--spa-primary)` en td:first-child |
| Botón catálogo | Hover gris genérico | Hover sky `#0ea5e9` + border semántico |
| Botón distribución | Hover gris genérico | Hover violet `#8b5cf6` |
| Botón editar | Hover gris genérico | Hover indigo `#6366f1` |
| Badge Activo/Inactivo | Solo texto | Punto `::before` color + texto |
| Modal (local) | `border-radius: 8px`, cabecera blanca | `border-radius: 20px`, `border-top: 3px solid primary`, cabecera tintada 5% |

**Cambio HTML**: clases tipadas añadidas a botones de acción — `class="icon catalog"`, `class="icon dist"`, `class="icon edit"`. Aplica en desktop (`.row-actions`) y mobile (`.card-actions`).

**Cambio `styles.css`** (global): `.modal-box` actualizado a `border-radius: 20px` + `border-top: 3px solid --spa-primary`. Aplica a TODOS los modales del proyecto (incluyendo catálogo dialog que usa clases globales).

---

## Componente productos-premium — pantalla nueva desde cero (2026-06-04)

**Ubicación**: `features/inventario/productos-premium/`  
**Selector**: `inv-productos-premium`  
**Ruta**: `/dashboard/productos-premium`  
**Permiso requerido**: `INVENTARIO_PRODUCTOS_VER` (igual que la tabla estándar)

### Design system aplicado — paleta Adifnex (2026-06-05, actualizado sesión 32)

Paleta original (indigo/violet) reemplazada por la identidad de marca Adifnex:

| Token | Valor | Uso |
|-------|-------|-----|
| `--p` | `#14532D` | Verde oscuro — primario (dominante) |
| `--p-accent` | `#EA580C` | Naranja — acento secundario |
| `--p-emerald` | `#10B981` | Activo / positivo |
| `--p-amber` | `#F59E0B` | Precio / destaque |
| `--p-bg` | `#F0FDF4` | Fondo mint (verde muy tenue) |
| `--p-surface` | `#FFFFFF` | Cards y tabla |
| `--p-text` | `#111827` | Texto oscuro neutro |
| `--p-muted` | `#6B7280` | Texto secundario |
| `--p-border` | `#D1FAE5` | Bordes sutiles verdes |

**Iteración de paleta**: durante la sesión 32 se probaron dos variantes —
1. Naranja como primario, verde como acento → descartado (verde se usaba demasiado en texto)
2. Verde `#14532D` como primario, naranja `#EA580C` como acento → **versión final**

**Tipografía**: DM Sans (Google Fonts, importado en `styles.css` vía `@import`)

### Elementos visuales clave

- **KPI bar**: 4 cards con gradientes independientes (total/activos/con precio/en catálogo). Hover con `translateY(-3px)`.
- **Título página**: `background-clip: text` con gradiente indigo→texto oscuro.
- **Header tabla**: `background: linear-gradient(90deg, #6366F1, #7C3AED)` — texto blanco.
- **Row hover**: `inset 3px 0 0 #6366F1` en primera celda + fondo `rgba(99,102,241,.04)`.
- **Avatares**: gradiente calculado desde hash del nombre (10 pares de colores, `avatarGradient(nombre)`).
- **Status activo**: badge verde `#ECFDF5` con `::before` que tiene `animation: pulse-dot 2s` (dot pulsante).
- **Status inactivo**: badge rojo estático `#FEF2F2`.
- **Tag chips**: glass-style con `color-mix` semántico + `backdrop-filter: blur(2px)`.
- **Botón primario**: `linear-gradient(135deg, #6366F1, #7C3AED)` + `box-shadow` glow + lift hover.
- **Paginación**: page activa con sombra indigo `0 2px 8px rgba(99,102,241,.38)`.

### Diferencias con productos-lista estándar

| Aspecto | productos-lista | productos-premium |
|---------|----------------|-------------------|
| Paleta | `--spa-*` (azul/beige) | `--p-*` (verde/naranja Adifnex) |
| KPI bar | No | Sí (4 cards) |
| Funcionalidad | Completa (CRUD, modales) | Solo visualización (sin modales) |
| Skeleton | `animate-pulse` gris | Shimmer `#ECFDF5→#D1FAE5` (mint) |
| Avatares | Color sólido | Gradiente dinámico |
| Header in-page | Sí (eyebrow + h1) | No — usa el top bar del panel |

### Archivos modificados para registrar la ruta

- `panel.component.ts`: import + `'productos-premium': 'INVENTARIO_PRODUCTOS_VER'` en `PERMISOS_VISTA` + `'productos-premium': 'Productos Premium'` en `titulos`
- `panel.component.html`: `@case ('productos-premium')` en el `@switch`

---

## Truco: negative margins en :host para cubrir padding del panel (2026-06-04)

**Problema**: el `.view-area` del panel tiene `padding: 1.5rem 3rem 2.5rem`. Cuando un componente define un fondo distinto en `:host`, se ve un "borde" del color del panel alrededor del componente.

**Solución**: negative margins exactos al padding del panel + re-aplicar el mismo padding:

```css
:host {
  background: var(--mi-fondo);
  margin: -1.5rem -3rem -2.5rem;   /* cancela el padding del panel */
  padding: 1.5rem 3rem 2.5rem;    /* lo re-aplica desde el host */
  box-sizing: border-box;
}

/* Espejear los breakpoints de panel.component.css */
@media (max-width: 1023px) {
  :host { margin: -1.25rem -2rem -2rem; padding: 1.25rem 2rem 2rem; }
}
@media (max-width: 640px) {
  :host { margin: -1rem -1.25rem -1.5rem; padding: 1rem 1.25rem 1.5rem; }
}
```

**Cuándo usar**: solo en componentes que necesitan un fondo de página distinto al del panel general (p.ej. `productos-premium` con `#F0FDF4` mint vs el beige `#faf8f6` del panel).

**Breakpoints de referencia** (`panel.component.css`):
- Default: `padding: 1.5rem 3rem 2.5rem`
- `≤1023px`: `padding: 1.25rem 2rem 2rem`
- `≤640px`: `padding: 1rem 1.25rem 1.5rem`

---

## Top bar del panel — acento de marca (2026-06-05)

**Decisión**: el `page-header` (barra blanca superior con el título de cada vista) mantiene fondo blanco — es chrome global compartido por todas las páginas. No se pinta per-page.

**Acento de marca aplicado**: `border-bottom` cambiado de gris neutro `#ebe6e1` a verde Adifnex:

```css
/* panel.component.css */
.page-header {
  border-bottom: 2px solid #14532D; /* antes: 1px solid #ebe6e1 */
}
```

**Razonamiento**: una línea de color conecta visualmente el shell con el contenido sin romper la consistencia cross-página. Pintar todo el header naranja o verde sería agresivo y rompería la identidad de páginas que no son de inventario.

---

## productos-premium — eliminación del header in-page (2026-06-05)

El componente tenía un `<header class="page-header">` propio dentro de la vista con eyebrow "INVENTARIO" + h1 "Productos". Esto duplicaba el título que ya muestra el panel en el top bar (`{{ tituloPagina() }}`).

**Fix**: eliminar el bloque `<header>` del template y sus estilos `.page-header`, `.page-eyebrow`, `.page-title` del CSS del componente.

**Resultado**: un solo título visible — el del top bar. La identidad visual viene de la paleta del componente (mint bg + verde/naranja), no de un header redundante.

---

## productos-premium — UX mobile modal edición (2026-06-06, sesión 36)

Tres mejoras mobile aplicadas al componente `productos-premium`:

### 1 — Vista previa del catálogo como bottom sheet

**Problema**: en mobile el split layout (form + preview) apilaba la preview debajo del formulario, ocupando espacio innecesario.

**Solución**:
- `.cat-preview-col` → `display: none` en `@media (max-width: 768px)` — oculta la columna de preview en mobile
- Signal `previewMobileVisible = signal(false)`
- Botón `cat-preview-inline-btn` al tope del tab Catálogo (solo visible en mobile): "Ver vista previa" con ícono `preview` + chevron derecho
- Al tocarlo → bottom sheet `cat-preview-sheet-overlay` (z-index: 9200) con la misma card de preview
- Bottom sheet: `border-radius: 22px 22px 0 0`, animación `cat-sheet-up` (`translateY(100%)→0`), handle gris, header con título y botón cerrar
- Backdrop click cierra el sheet

```typescript
readonly previewMobileVisible = signal(false);
```

```css
/* Botón invisible en desktop, visible en mobile */
.cat-preview-inline-btn { display: none; }
@media (max-width: 768px) {
  .cat-preview-inline-btn {
    display: flex; align-items: center; gap: .55rem;
    width: 100%; padding: .75rem 1rem; border-radius: 12px;
    border: 1.5px solid rgba(20,83,45,.22);
    background: rgba(20,83,45,.06); color: var(--p);
    min-height: 44px; /* touch target */
  }
}
```

### 2 — Modal con altura fija (evita encogimiento al cambiar tab)

**Problema**: `max-height: 90vh` hacía que el modal se encogiera al cambiar al tab "Imágenes" (contenido corto).

**Fix**: cambiar a `height` fijo usando `min()`:

```css
.cat-modal { height: min(88vh, 720px); } /* antes: max-height: 90vh */
@media (max-width: 768px) { .cat-modal { height: 90vh; } }
@media (max-width: 480px) { .cat-modal { height: 92vh; } }
```

El `cat-tab-content` ya tiene `flex: 1; overflow-y: auto` — el espacio sobrante queda en blanco scrollable en lugar de encoger el modal.

**Regla**: modales con tabs de contenido variable deben tener `height` fijo, no `max-height`, para mantener tamaño consistente entre pestañas.

### 3 — Formulario "Registrar movimiento" como bottom sheet en mobile

**Problema**: el tab Distribución mostraba el formulario de movimiento inline junto a la tabla de stock, generando scroll horizontal y layout roto.

**Causa raíz del scroll horizontal**: doble padding — `.dist-layout` tiene `padding: 1.25rem` pero vive dentro de `.cat-tab-content` que también tiene `padding: 1.5rem`. En mobile eso reduce el ancho útil ~90px.

**Solución**:
- `.dist-mov-panel` → `display: none` en `@media (max-width: 768px)`
- `.dist-layout { padding: 0 }` en mobile (quita el doble padding)
- `overflow-x: hidden` en `.cat-tab-content` — previene desbordamiento residual
- `table-layout: fixed` en `.dist-table` — columnas no explotan el ancho
- Botón trigger `dist-mov-mobile-trigger` visible solo en mobile, al final del panel de stock (badge verde + icono `swap_vert` + chevron)
- Signal `movModalMobileVisible = signal(false)`
- Bottom sheet z-index: 9200 con el formulario completo — usa los mismos signals que el panel inline (estado compartido)
- Al registrar exitosamente → `movModalMobileVisible.set(false)` cierra el sheet automáticamente

```typescript
readonly movModalMobileVisible = signal(false);

// En ambos paths de ejecutarMovimiento() success:
this.movModalMobileVisible.set(false);
```

```css
.dist-mov-mobile-trigger { display: none; }
@media (max-width: 768px) {
  .dist-mov-mobile-trigger {
    display: flex; align-items: center; gap: .6rem;
    width: 100%; margin-top: .85rem; padding: .85rem 1rem;
    border-radius: 14px; min-height: 52px;
    border: 1.5px solid rgba(20,83,45,.2); background: rgba(20,83,45,.06); color: var(--p);
  }
  .dist-mov-trigger-icon { /* badge cuadrado verde con mat-icon */ }
  .dist-mov-trigger-chevron { margin-left: auto; opacity: .45; }
}
```

**Regla derivada**: en mobile, los paneles secundarios de formularios largos (especialmente dentro de un modal ya pequeño) deben convertirse a bottom sheets en lugar de apilarse en el flujo normal.

---

## UX refactoring — productos-lista (2026-06-03)

Cambios aplicados tras auditoría impeccable (score 5.5/10 → objetivo 7+):

| Problema | Fix aplicado |
|----------|-------------|
| 11 columnas → scroll horizontal en desktop | Eliminadas columnas `CREADO POR` y `ACTUALIZADO POR` |
| Chips apilados → fila de altura variable | Máximo 2 chips visibles + badge `+N` para excedente |
| Label "BUSCAR" redundante | Eliminado — el ícono de lupa es suficiente |
| Precio alineado izquierda | Añadida clase `.num-col { text-align: right }` en `<th>` y `<td>` |
| Orden: solo ID y nombre útiles | `ordenCampo` simplificado a `'id' | 'nombre'` |

**CSS agregado** en `productos-lista.component.css`:
```css
.tag-more {
  display: inline-block; padding: .15rem .45rem; border-radius: 999px;
  background: color-mix(in srgb, var(--spa-border) 60%, transparent);
  color: var(--spa-text-muted); font-size: .72rem; font-weight: 600;
}
.num-col { text-align: right; }
```

---

## productos-premium — modal edición refinado y nuevas funcionalidades (2026-06-06, sesión 37)

### Preview column estirada

El panel derecho de vista previa del modal catálogo creció:
- `grid-template-columns: 1fr 340px` → `1fr 380px`
- `cat-card-img height: 120px` → `170px` — más espacio para la imagen del producto

### Eliminación de botones "Distribución" y "Editar"

Los botones `dist` y `edit` de `.row-actions` (desktop) y `.card-actions` (mobile) fueron eliminados del componente `productos-premium`. Ahora solo quedan dos botones por fila: **Catálogo** y **Eliminar**.

**Razón**: el modal unificado de catálogo ya tiene el tab "Producto" para editar datos y el tab "Distribución" para mover stock. Los botones de acción directa eran redundantes.

### Modal "Crear nuevo producto" en productos-premium

Se añadió un modal de creación completo al componente `productos-premium`, equivalente funcional al de `productos-lista` pero usando signals puras (sin FormBuilder).

**Campos del formulario**:
- Nombre (requerido, mín. 2 caracteres)
- SKU (toggle automático/manual + spinner mientras sugiere SKU)
- Descripción (textarea, opcional)
- Stock mínimo + Marca (grid 2 columnas)
- Etiquetas (chips clickables con color-mix, mismo patrón que tab Producto del modal edición)

**Implementación de SKU automático** — Subject con debounce:
```typescript
private readonly nombreCrearChange$ = new Subject<string>();

// En constructor:
this.nombreCrearChange$.pipe(
  debounceTime(500),
  distinctUntilChanged(),
  switchMap(nombre => {
    if (!nombre || nombre.trim().length < 2 || this.crearSkuManual()) return EMPTY;
    this.crearSugieriendoSku.set(true);
    return this.servicioEtiquetas.sugerirSku(nombre.trim());
  }),
  takeUntilDestroyed(this.destruirRef)
).subscribe({ next: (res) => { this.crearSku.set(res.sku); ... } });
```

**Signals nuevas**: `crearVisible`, `crearNombre`, `crearSkuManual`, `crearSku`, `crearStockMinimo`, `crearMarcaId`, `crearEtiquetaIds`, `crearDescripcion`, `crearError`, `creando`, `crearSugieriendoSku`.

**CSS nuevo** — `.crear-modal` (max-width 560px, `border-top: 3px solid #14532D`, animación `cat-slide-up` reutilizada) + `.crear-modal-body` (padding + scroll interno).

**Permiso**: botón protegido con `*appPermiso="'INVENTARIO_PRODUCTOS_CREAR'"`.

### "Guardar cambios" condicional por tab

```html
@if (tabActivo() === 'producto' || tabActivo() === 'catalogo') {
  <button class="btn-primary" (click)="guardarCatalogo()" ...>
    Guardar cambios
  </button>
}
```

En tabs **Imágenes** y **Distribución** el botón desaparece. Cancelar permanece visible en todos los tabs.

**UX rationale** (ui-ux-pro-max): en tabs operativos (imágenes, movimientos de stock) el usuario no edita datos del formulario → mostrar "Guardar cambios" genera confusión sobre qué se guardaría.

### Click fuera del modal no cierra

Eliminado `(click)="cerrarCatalogo()"` del `.cat-overlay` y `(click)="cerrarCrear()"` del overlay del modal crear. El modal solo se cierra con el botón X o Cancelar.

**UX rationale**: modales con formularios (especialmente en mobile donde un toque fuera es accidental) no deben cerrarse con click backdrop — evita pérdida de datos ingresados.

### Toast notifications en todas las acciones del modal

`SnackBarServicio` ya estaba inyectado. Se agregaron llamadas a `snack.exito()` / `snack.error()` en los cuatro métodos que antes tenían `error: () => {}` vacío:

| Método | Toast éxito | Toast error |
|--------|-------------|-------------|
| `guardarCatalogo()` | `"[nombre] actualizado correctamente."` | `"No se pudo guardar el producto."` |
| `subirImagen()` | `"Imagen subida correctamente."` | `"No se pudo subir la imagen."` |
| `imgMarcarPrincipal()` | `"Imagen principal actualizada."` | `"No se pudo cambiar la imagen principal."` |
| `imgEliminar()` | `"Imagen eliminada."` | `"No se pudo eliminar la imagen."` |

### Fix: z-index toast por debajo del modal

**Síntoma**: el SnackBar de Material aparecía detrás del overlay del modal (`z-index: 9000`).

**Causa**: `cdk-overlay-container` (contenedor de Material Overlays — snackbars, dialogs, tooltips) tiene `z-index: 1000` por defecto.

**Fix en `styles.css` (global)**:
```css
.cdk-overlay-container {
  z-index: 10000 !important;
}
```

**Regla**: siempre que el proyecto tenga modales custom con `z-index > 1000`, elevar `cdk-overlay-container` a `z-index` superior en `styles.css`. El valor `10000` está por encima de todos los modales propios del proyecto (9000 / 9100 / 9200).

### Funcionalidad eliminar producto en productos-premium

Botones de eliminar (desktop y mobile) estaban sin `(click)` handler. Implementado con el mismo patrón que `productos-lista`:

```typescript
// Imports añadidos
import { MatDialog, MatDialogModule } from '@angular/material/dialog';
import { ConfirmDialogComponent } from '@shared/components/confirm-dialog/confirm-dialog.component';

confirmarEliminar(producto: Producto): void {
  const ref = this.dialog.open(ConfirmDialogComponent, {
    data: {
      titulo: 'Eliminar producto',
      mensaje: `¿Eliminar "${producto.nombre}"? Esta acción no se puede deshacer.`,
      textoConfirmar: 'Eliminar',
      tipo: 'peligro',
    },
  });
  ref.afterClosed().subscribe(confirmado => {
    if (confirmado) this.eliminar(producto);
  });
}

private eliminar(producto: Producto): void {
  this.servicio.eliminar(producto.id).pipe(...).subscribe({
    next: () => {
      this.productos.update(lista => lista.filter(p => p.id !== producto.id));
      this.snack.exito(`${producto.nombre} fue eliminado.`);
    },
    error: () => this.snack.error('No se pudo eliminar el producto.'),
  });
}
```

**Optimización local**: al eliminar, se hace `productos.update(lista => lista.filter(...))` sin recargar del backend. Mismo endpoint que `productos-lista`: `servicio.eliminar(id)`.

---

## Regla estándar: tablas full-height sin scroll de página (2026-06-06, sesión 39)

**Regla**: toda vista con tabla dentro del panel debe ocupar el máximo de pantalla disponible. La página nunca debe scrollear. El scroll va dentro del componente, en `.table-scroll`.

### El problema

`.view-area` tiene `overflow-y: auto` por defecto → la página entera scrollea cuando la tabla supera el viewport. KPIs y toolbar quedan fuera de vista al bajar.

### La solución — 3 pasos obligatorios

**Paso 1: `panel.component.html`** — registrar la vista con clase propia:

```html
<div class="view-area"
  [class.view-dashboard]="vistaActiva === 'dashboard'"
  [class.view-productos]="vistaActiva === 'productos'"
  [class.view-NUEVA_VISTA]="vistaActiva === 'NUEVA_VISTA'">
```

**Paso 2: `panel.component.css`** — `overflow: hidden` + `display: flex`:

```css
.view-area.view-NUEVA_VISTA {
  overflow: hidden;
  display: flex;
  flex-direction: column;
  padding: 0;   /* el componente aplica su propio padding */
}
```

**Paso 3: componente `:host`** — `flex: 1`, sin negative margins:

```css
:host {
  flex: 1;
  display: flex; flex-direction: column; min-height: 0; overflow: hidden;
  padding: 1.5rem 3rem 2.5rem;   /* mismo valor que .view-area original */
  box-sizing: border-box;
}
@media (max-width: 1023px) { :host { padding: 1.25rem 2rem 2rem; } }
@media (max-width: 640px)  { :host { padding: 1rem 1.25rem 1.5rem; } }
```

### Cadena flex obligatoria

```
content-area (100vh, flex column)
  → view-area.view-VISTA (flex:1, overflow:hidden, display:flex column, padding:0)
    → :host (flex:1, overflow:hidden, flex column, padding propio)
      → .shell / raíz del componente (flex:1, min-height:0, flex column)
        → KPIs / toolbar (flex-shrink:0)
        → .data-card (flex:1, min-height:0, overflow:hidden)
          → .table-wrapper (flex:1, min-height:0, overflow:hidden)
            → .table-scroll (flex:1, min-height:0, overflow-y:auto) ← scroll aquí
```

### Cuándo NO usar negative margins

El truco de negative margins (`margin: -1.5rem -3rem -2.5rem`) **ya no aplica** para vistas con tabla full-height. Solo aplica si el componente necesita extender su fondo visualmente pero NO requiere contener scroll (caso poco común). Para tablas, siempre usar este nuevo patrón.

### Implementado en

- `productos-premium` — referencia canónica del patrón

Regla también documentada en `frontend/CLAUDE.md` del proyecto.

---

## Tokens `--p-*` globalizados en styles.css y Tailwind (2026-06-06, sesión 39)

La paleta Adifnex que vivía scoped en `:host {}` de `productos-premium` fue promovida a variables globales.

### Cambios

**`src/styles.css`** — nuevo bloque en `:root {}`:
```css
--p: #14532D;  --p-accent: #EA580C;  --p-emerald: #10B981;
--p-amber: #F59E0B;  --p-sky: #0EA5E9;  --p-rose: #F43F5E;
--p-bg: #F0FDF4;  --p-surface: #FFFFFF;  --p-text: #111827;
--p-muted: #6B7280;  --p-border: #D1FAE5;
--p-shadow: 0 2px 12px rgba(20,83,45,.08);
```

**`tailwind.config.js`** — colores `p-*` añadidos como clases `bg-p-primary`, `text-p-muted`, `border-p-border`, etc.

**`DESIGN.md`** — nueva sección 0 "Paleta Adifnex": tabla completa de tokens, gradientes recurrentes, colores semánticos de movimiento (ingreso/salida/transferencia).

### Reglas de reuso

- CSS de componente → `var(--p)`, `var(--p-bg)`, etc.
- Template HTML → clases Tailwind `bg-p-primary`, `text-p-muted`, etc.
- No hardcodear `#14532D` directamente — siempre por token.
- Override puntual: solo redefinir el token que cambia en `:host {}`, no todo el sistema.
- `--p-shadow` solo disponible como CSS var (no hay clase Tailwind para box-shadow completo).

### Coexistencia con `--spa-*`

Los tokens `--spa-*` (azul `#2b7ea6`, crema `#faf8f6`) siguen activos — alimentan los modales globales, sidebar, botones `.primary-btn`/`.ghost-btn`. Los dos sistemas son paralelos. Para componentes nuevos de inventario usar `--p-*`.

---

## Gotcha: CSS de componente supera budget de Angular (2026-06-06)

**Síntoma**: `npm run build` falla con error:

```
X [ERROR] src/app/features/inventario/productos-premium/productos-premium.component.css
exceeded maximum budget. Budget 24.58 kB was not met by 12.42 kB with a total of 36.99 kB.
```

**Causa**: `productos-premium.component.css` creció a 36.99 kB por el acumulado de tabs, modales, animaciones, responsive y design system. El límite default de Angular CLI para `anyComponentStyle` es 16 kB warning / 24 kB error.

**Fix en `angular.json`** (sección `budgets` de la configuración `production`):

```json
{
  "type": "anyComponentStyle",
  "maximumWarning": "40kB",
  "maximumError": "64kB"
}
```

**Cuándo aplicar**: cuando un componente complejo justifica un CSS grande (modal con múltiples tabs, sistema de diseño inline, animaciones). No aumentar el budget para componentes simples.

---

## Sidebar — diseño blanco/verde Adifnex (construido en sesión 41, promovido a definitivo en sesión 43)

**Historial**:
- Sesión 41 (2026-06-06): construido como `barra-lateral-v2` en paralelo, selector `spa-sidebar-v2`
- Sesión 43 (2026-06-06): promovido — `barra-lateral-v2/` eliminado, código migrado a `barra-lateral/` con selector `spa-sidebar` y clase `BarraLateralComponent`. Dark navy sidebar (v1) eliminado.

**Selector definitivo**: `spa-sidebar` — clase `BarraLateralComponent`.

**Archivos**:
- `shared/components/barra-lateral/barra-lateral.component.ts`
- `shared/components/barra-lateral/barra-lateral.component.html`
- `shared/components/barra-lateral/barra-lateral.component.css`

### Diferencias de diseño vs v1 (dark navy)

| Aspecto | v1 (original) | v2 (nuevo) |
|---------|--------------|-----------|
| Fondo | `#0f2030` (navy) | `#FFFFFF` blanco |
| Item activo | Fondo azul translúcido | `#F0FDF4` + barra izquierda emerald `3px` |
| Brand mark | Círculo flat azul | Cuadrado `border-radius:10px`, gradiente `#14532D → #10B981` |
| Avatar perfil | Círculo flat | `border-radius:10px` mismo gradiente verde |
| Icons activos | `#5db8df` azul | `#10B981` emerald |
| Context selector | mat-form-field + badge separado | Workspace switcher compacto — 1 sola fila |
| Paleta | Azul naval custom | Tokens `--p-*` de `productos-premium` |
| Width colapsado | 88px | 80px |

### Workspace switcher — custom dropdown (no mat-select)

**Problema documentado**: `mat-select` de Angular Material abre su overlay fuera del sidebar, con las opciones envolviéndose y sin styling controlable.

**Solución**: dropdown completamente custom con `@if (wsDropdownAbierto())`:
- Botón trigger con icono gradiente + label micro "NEGOCIO ACTIVO" + nombre + chevron
- Dropdown `position: absolute; z-index: 200` dentro del sidebar
- Cada opción: avatar con inicial (gradiente verde) + nombre truncado con `text-overflow: ellipsis`
- Opción "Plataforma (global)" con ícono `public` distinto
- Check `✓` emerald en opción activa
- Chevron animado (rotate 180° al abrir)
- `@HostListener('document:click')` cierra al click fuera
- `@HostListener('document:keydown.escape')` también cierra

**Regla derivada**: en sidebars angostos, evitar `mat-select` para listas de opciones. Usar dropdown custom con `position: absolute` y `max-height + overflow-y: auto`.

---

## Color naranja `#EA580C` — integración en el sistema (sesión 42, 2026-06-06)

**Decisión**: agregar `#EA580C` (naranja, ya token `--p-accent`) como color activo visible en la UI. Antes era un token sin uso real.

### Dónde aparece

| Lugar | Elemento | Antes | Después |
|-------|---------|-------|---------|
| `barra-lateral` | Brand mark logo | gradiente `#14532D → #10B981` | gradiente `#14532D → #EA580C` |
| `barra-lateral` | Avatar perfil (`.sl-avatar`) | gradiente `#14532D → #10B981` | gradiente `#14532D → #EA580C` |
| `barra-lateral` | Workspace icon (`.ws-icon`) | gradiente `#14532D → #10B981` | gradiente `#14532D → #EA580C` |
| `barra-lateral` | Workspace opt avatar (`.ws-opt-avatar`) | gradiente `#14532D → #10B981` | gradiente `#14532D → #EA580C` |
| `productos-premium` | Botón CTA "Nuevo producto" | gradiente verde `#14532D → #166534` | gradiente naranja `#EA580C → #C2410C` |
| `productos-premium` | Summary badge "N productos" | tint verde | tint naranja `rgba(234,88,12,.08)` |

### Principio de uso

- **Identidad de marca** (logo, avatares): gradiente verde→naranja — el naranja es el "calor" que complementa el verde oscuro
- **CTA primario** (botón crear): naranja sólido — los CTAs se benefician del naranja porque naturalmente llama la atención sin ser agresivo
- **No toca**: tabla thead (verde), items activos del nav (emerald), estados de stock, KPI cards — siguen siendo verdes (semántica de inventario)

### Tokens actualizados en DESIGN.md

- `--p-accent` rol actualizado: "Acento naranja — logo/avatar gradiente endpoint, botón CTA primario, summary badge"
- Gradiente nuevo documentado: `linear-gradient(135deg, #14532D 0%, #EA580C 100%)` — logo/avatar sidebar
- Gradiente CTA: `linear-gradient(135deg, #EA580C 0%, #C2410C 100%)`

---

## Bug fix — dropdown perfil barra-lateral no se abría (sesión 42, 2026-06-06)

**Síntoma**: click en el área de perfil (Administrador / Admin con chevron) no desplegaba el menú.

**Causa raíz**: `@HostListener('document:click')` en el componente cierra `menuCuentaAbierto` a `false` en cualquier click del documento. El click en el perfil primero ejecuta `alternarMenuCuenta()` (lo pone en `true`) y luego el mismo evento burbujea al documento y el listener lo baja de nuevo a `false` — el menú jamás se abre.

**El ws-switcher ya tenía el fix** (`$event.stopPropagation()` en su div, línea 49 del HTML) — por eso el workspace dropdown sí funcionaba. El perfil no lo tenía.

**Fix aplicado**: añadir `$event.stopPropagation()` al click del perfil:

```html
<!-- antes -->
(click)="alternarMenuCuenta()"

<!-- después -->
(click)="$event.stopPropagation(); alternarMenuCuenta()"
```

**Regla derivada**: cuando se usa `@HostListener('document:click')` para cerrar dropdowns, **todos** los elementos que abran un dropdown deben tener `$event.stopPropagation()` en su handler. Sin stopPropagation, el mismo click que abre también cierra.

**Valores anteriores**: `maximumWarning: 16kB` / `maximumError: 24kB`.

---

## productos-premium — mobile card redesign: compact flex row (sesión 44, 2026-06-07)

### Problema original

La tarjeta mobile (`prod-card`) tenía 4 filas apiladas (~180–200 px por item) → solo 2 productos visibles en pantalla en iPhone 12 Pro (390×844). La densidad era insuficiente para trabajo real de inventario.

### Solución: `prod-row` — 2 líneas en ~76 px

Reemplazada la estructura de 4 filas por una fila flex compacta:

```
[avatar 28px] [body flex-col] [actions 2 botones]
               ├─ L1: nombre ················· dot de estado
               └─ L2: sku · precio · marca · icono-catálogo · chips
```

**Resultado**: 4–5 productos visibles simultáneamente en la misma pantalla.

#### HTML clave

```html
<article class="prod-row" [class.prod-row-inactive]="!p.estado">
  <div class="prod-row-av" [style.background]="avatarGradient(p.nombre)">{{ avatarLetra(p.nombre) }}</div>
  <div class="prod-row-body">
    <div class="prod-row-l1">
      <strong class="prod-row-nombre">{{ p.nombre }}</strong>
      <span class="prod-status-dot" [class.prod-dot-activo]="p.estado" [class.prod-dot-inactivo]="!p.estado"></span>
    </div>
    <div class="prod-row-l2">
      <span class="sku-tiny">{{ p.sku }}</span>
      @if (p.precio != null) { <span class="meta-sep">·</span><span class="price-tiny">{{ p.precio | currency:'PEN':'S/ ':'1.2-2' }}</span> }
      @if (p.marcaNombre) { <span class="meta-sep">·</span><span class="marca-tiny">{{ p.marcaNombre }}</span> }
      @for (tag of (p.etiquetas ?? []).slice(0, 2); track tag.id) {
        <span class="tag-chip-tiny" [style.--chip-color]="tag.color">{{ tag.nombre }}</span>
      }
      @if ((p.etiquetas ?? []).length > 2) { <span class="tag-more-tiny">+{{ (p.etiquetas ?? []).length - 2 }}</span> }
    </div>
  </div>
  <div class="prod-row-acts">
    <button *appPermiso="'INVENTARIO_PRODUCTOS_CATALOGO'" type="button" class="action-btn catalog" (click)="abrirCatalogo(p)"><mat-icon>language</mat-icon></button>
    <button *appPermiso="'INVENTARIO_PRODUCTOS_ELIMINAR'" type="button" class="action-btn delete" (click)="confirmarEliminar(p)"><mat-icon>delete</mat-icon></button>
  </div>
</article>
```

#### Decisiones de diseño

| Elemento | Antes | Después | Razón |
|---------|-------|---------|-------|
| Indicador estado | Pill "Activo"/"Inactivo" con texto | Dot 7px animado (pulse verde / gris estático) | Ahorra ancho, legible con un vistazo |
| Marca | Fila separada con label | Inline en L2 separado por `·` | L2 puede wrappear — no necesita fila propia |
| Etiquetas | Chips grandes en fila propia | `tag-chip-tiny` 0.62rem inline, máx 2 + badge `+N` | Misma info, 0 filas extra |
| Avatar | Ausente | `28×28px border-radius:8px` con gradiente | Punto focal visual, lookup instantáneo por inicial |
| Acciones | Botones `.card-actions` estilo large | Botones 28×28px con `mat-icon 13px` | Zona táctil suficiente, huella mínima |
| Separadores | — | `·` (`meta-sep` gris claro) | Visualmente separa datos sin ocupar espacio |

### Gotcha: CSS Grid con `grid-row: span` — no usar

**Primer intento** usó `display: grid; grid-template-columns: 28px 1fr auto` con `grid-row: 1/3` en avatar y acciones. Resultado: avatar tomó el ancho completo como barra verde horizontal.

**Causa**: cuando items mezclan colocación explícita (`grid-row`) y auto-placement, CSS Grid auto-placement se comporta de forma impredecible. El avatar con `grid-row: 1/3` y sin `grid-column: 1` explícito quedaba auto-placed en la primera celda libre pero rompía el flow.

**Fix definitivo**: abandonar CSS Grid en este patrón. Usar flexbox con un div `.prod-row-body` explícito como wrapper de las dos líneas. Predecible y sin sorpresas.

---

## productos-premium — eliminación de summary badge redundante (sesión 44, 2026-06-07)

**Síntoma**: la toolbar del componente mostraba `≡ 279 productos` (`.summary-badge`). El mismo número ya aparecía en la KPI card "Total productos" debajo de la toolbar.

**Fix**: eliminar el `<span class="summary-badge">` del bloque `.toolbar-end` en el HTML. La KPI card es la única fuente de verdad para el conteo total.

**Regla derivada**: en vistas con KPI cards, no duplicar métricas en la toolbar. La toolbar sirve para acciones (filtros, botón crear), no para repetir datos que las KPIs ya muestran.

---

## productos-premium — fix paginación desaparece en mobile (sesión 44, 2026-06-07)

**Síntoma**: al hacer scroll en la lista mobile, la paginación (footer con controles prev/next) se iba hacia abajo y quedaba oculta. Solo era visible cuando el usuario llegaba al final de la lista.

**Causa**: `.mobile-wrapper` tenía `overflow-y: auto`, haciendo que todo el contenido (cards + footer) fuera un solo bloque scrolleable. La paginación no estaba "anclada".

**Fix**: mismo patrón que desktop (`.table-scroll` + `.table-footer`). Se añadió un div `.mobile-list` entre el wrapper y el footer:

```html
<div class="block md:hidden mobile-wrapper">
  <div class="mobile-list">
    @for (p of productosPaginados(); track p.id) { ... }
  </div><!-- /mobile-list -->
  <footer class="mobile-footer">...</footer>
</div>
```

```css
.mobile-wrapper {
  display: flex; flex-direction: column;
  flex: 1; min-height: 0; overflow: hidden;   /* ← overflow:hidden, no auto */
}
.mobile-list {
  display: flex; flex-direction: column; gap: 4px;
  flex: 1; min-height: 0; overflow-y: auto;   /* ← scroll aquí */
  padding: .15rem 0;
}
.mobile-footer {
  display: flex; justify-content: space-between; align-items: center;
  padding: .5rem .25rem; flex-shrink: 0;      /* ← no shrinkea = siempre visible */
  border-top: 1px solid var(--p-border); background: var(--p-surface);
}
```

**Patrón generalizado** (aplica a cualquier vista mobile con scroll + footer fijo):
```
.mobile-wrapper (overflow:hidden, flex column)
  → .mobile-list   (flex:1, min-height:0, overflow-y:auto)  ← scroll aquí
  → .mobile-footer (flex-shrink:0)                           ← siempre visible
```

Es el equivalente mobile de: `.table-wrapper → .table-scroll + .table-footer` usado en el desktop.

---

## sedes-lista — rediseño premium completo (sesión 45, 2026-06-07)

Componente `sedes-lista` reescrito desde cero aplicando el mismo design language que `productos-premium`. Los tres archivos del componente fueron reemplazados en su totalidad.

### Design tokens usados

Usa los tokens globales `--p-*` ya definidos en `styles.css`. Gradientes específicos del componente:

| KPI | Gradiente |
|-----|-----------|
| Total sedes | `#14532D → #166534` (verde oscuro) |
| Activas | `#10B981 → #059669` (emerald) |
| Inactivas | `#6B7280 → #4B5563` (slate) |
| Con dirección | `#7C3AED → #6D28D9` (violet) |

**Decisión de color**: violet para "localización" — diferencia semánticamente de verde (inventario). El violet también se usa en el botón de acción "Ver stock" (consistencia con el KPI).

### Estructura de la pantalla

```
toolbar (búsqueda + botón crear)
KPI bar (4 cards: total / activas / inactivas / con dirección)
data-card (tabla desktop + mobile-wrapper — con scroll interno)
  tabla desktop: header verde, col: #/avatar+nombre+id / dirección / creador / estado / acciones
  mobile: avatar 28px, 2 líneas (nombre+dot | pin+dirección), 2 botones
paginación (desktop footer + mobile footer — siempre visible)
```

### Avatar por hash

Las sedes no tienen imagen — se genera un avatar con inicial y gradiente determinista:

```typescript
private readonly GRADIENTS = [
  'linear-gradient(135deg, #14532D, #166534)',
  'linear-gradient(135deg, #064E3B, #065F46)',
  'linear-gradient(135deg, #1E3A5F, #1D4ED8)',
  'linear-gradient(135deg, #4C1D95, #6D28D9)',
  'linear-gradient(135deg, #7C2D12, #EA580C)',
  'linear-gradient(135deg, #0F172A, #1E293B)',
  'linear-gradient(135deg, #134E4A, #0F766E)',
  'linear-gradient(135deg, #831843, #BE185D)',
];

avatarGradient(nombre: string): string {
  const hash = nombre.split('').reduce((a, c) => a + c.charCodeAt(0), 0);
  return this.GRADIENTS[hash % this.GRADIENTS.length];
}
```

**Mismo patrón que `productos-premium`** — reutilizable en cualquier entidad sin imagen.

### Modal premium (`cat-overlay`)

- Ícono con gradiente verde para crear, azul para editar
- Strip de identidad en modo editar (avatar + nombre + id badge)
- Campo nombre + textarea dirección (4 filas)
- Caja decorativa violet "mapa" con descripción de la función de dirección
- Footer: Cancelar (ghost) + Guardar (verde / loading state)
- **Mobile bottom sheet** (`< 600px`): `border-radius: 18px 18px 0 0`, overlay `align-items: flex-end`, slide-up animation

### Signals y computed

```
sedes()          → raw array desde API
filtro()         → búsqueda en nombre + dirección
paginaActual()   → se resetea a 1 al cambiar filtro (via effect)
ordenCampo()     → 'nombre' | 'id'
ordenDir()       → 'asc' | 'desc'

sedesFiltradas   → sedes() filtrado por texto
sedesOrdenadas   → sedesFiltradas() ordenado
sedesPaginadas   → sedesOrdenadas() slice por página
totalActivas     → count donde s.estado === true
totalInactivas   → count donde s.estado === false
totalConDireccion → count donde s.direccion?.trim() tiene valor
```

### Permisos aplicados

| Acción | Permiso |
|--------|---------|
| Crear sede | `INVENTARIO_SEDES_CREAR` |
| Ver stock | `INVENTARIO_STOCK_VER` |
| Editar sede | `INVENTARIO_SEDES_EDITAR` |
| Eliminar sede | `INVENTARIO_SEDES_ELIMINAR` |

### Cambios en panel.component

**`panel.component.html`** — ya tenía `[class.view-sedes]="vistaActiva === 'sedes'"` en el `div.view-area`.

**`panel.component.css`** — regla combinada (sin scroll de página):

```css
.view-area.view-productos,
.view-area.view-sedes {
  overflow: hidden;
  display: flex;
  flex-direction: column;
  padding: 0;
}
```

### Skeleton loaders

- Desktop: 6 filas fake con `@keyframes cat-shimmer`
- Mobile: 5 filas fake con mismo patrón
- Aparecen durante `cargando() === true`, desaparecen al recibir datos

---

## sede-stock convertido a drawer (sesión 46, 2026-06-07)

La vista `/dashboard/sede-stock` fue eliminada como ruta standalone. En su lugar, `SedeStockComponent` ahora se renderiza como slide-over drawer dentro de `sedes-lista`.

### Cambios en `panel.component`

- Eliminado `SedeStockComponent` de `imports[]`
- Eliminado `'sede-stock'` del mapa `PERMISOS_VISTA` y `tituloPagina()`
- Eliminado `@case ('sede-stock')` del switch del template
- Eliminados métodos `alVerStockSede()` y `alVolverSedes()`
- Eliminadas propiedades `sedeSeleccionadaStock` y `claveAlmacSedeStock`

### Cambios en `sedes-lista.component`

- Eliminado `@Output() verStock = new EventEmitter<Sede>()`
- Agregado `SedeStockComponent` a `imports[]`
- Signal: `readonly sedeStock = signal<Sede | null>(null)`
- Métodos: `abrirStock(sede)` y `cerrarStock()`
- Botones "Ver stock" en tabla desktop y mobile card ahora llaman `abrirStock(s)` directamente

### Patrón del drawer (HTML en sedes-lista)

```html
@if (sedeStock()) {
  <div class="stock-backdrop" (click)="cerrarStock()" role="dialog" aria-modal="true">
    <aside class="stock-drawer" (click)="$event.stopPropagation()">
      <!-- header: avatar + nombre + dirección + badge estado + botón X -->
      <div class="stock-drawer-body">
        <inv-sede-stock [sede]="sedeStock()" [enModal]="true" (volver)="cerrarStock()"></inv-sede-stock>
      </div>
    </aside>
  </div>
}
```

### CSS del drawer

| Clase | Valor clave |
|-------|------------|
| `.stock-backdrop` | `position:fixed; inset:0; z-index:9000; background:rgba(3,16,8,.52); backdrop-filter:blur(4px)` |
| `.stock-drawer` | `position:fixed; top:0; right:0; bottom:0; width:min(86vw,1100px); background:#F0FDF4; box-shadow:-20px 0 80px rgba(0,0,0,.22); z-index:9001` |
| Header | Gradiente `#14532D → #065F46` + pseudo-circles decorativos |
| Body background | `#F0FDF4` + dot pattern `radial-gradient(circle, rgba(20,83,45,.06) 1px, transparent 1px); background-size:22px 22px` |
| Mobile | `width:100vw; border-left:none` |
| Animaciones | Backdrop: `stockFadeIn .2s`; Drawer: `stockSlideIn .3s cubic-bezier(.32,.72,0,1)` |

### `@Input() enModal` en `SedeStockComponent`

```typescript
@Input() enModal = false;
```

Cuando `enModal === true`:
- El `<header class="page-header">` (con botón volver) queda oculto (`@if (!enModal)`)
- Se muestra un KPI strip con 3 chips: productos en sede, unidades totales, stock bajo (≤5)

```html
@if (enModal && !stockCargando() && !stockError() && stockItems().length > 0) {
  <div class="kpi-strip"> ... </div>
}
```

---

## Toggle custom + select nativo en sede-stock (sesión 46, 2026-06-07)

Reemplazo de `mat-button-toggle-group` y `mat-select` por componentes completamente custom dentro de `SedeStockComponent`.

### Toggle de tipo de movimiento

Eliminado `MatButtonToggleModule` y `MatFormFieldModule`. Reemplazado por 3 botones nativos con color semántico:

| Tipo | Color activo | `tipoMovimiento` |
|------|-------------|-----------------|
| Ingreso | `#14532D` verde | `'INGRESO'` |
| Salida | `#B91C1C` rojo | `'SALIDA'` |
| Transferencia | `#1D4ED8` azul | `'TRANSFERENCIA'` |

```html
<div class="tipo-toggle" role="group">
  <button type="button" class="tipo-btn"
    [class.tipo-active]="tipoMovimiento === 'INGRESO'"
    [class.tipo-ing]="tipoMovimiento === 'INGRESO'"
    (click)="alCambiarTipo('INGRESO')">
    <mat-icon>add_circle</mat-icon><span>Ingreso</span>
  </button>
  <!-- Salida y Transferencia igual -->
</div>
```

Fondo inactivo: pill `#F1F5F9` con borde verde tenue. Activo: `background:#fff; box-shadow:0 1px 8px rgba(0,0,0,.12)`.

### Select nativo con flecha custom

Eliminado `mat-form-field` + `mat-select`. Reemplazado por `<select>` nativo con `formControlName`:

```html
<div class="select-wrap" [class.select-error]="controles.productoId.invalid && controles.productoId.touched">
  <select id="sel-producto" class="custom-select" formControlName="productoId">
    <option [ngValue]="null" disabled hidden>Selecciona un producto</option>
    @if (tipoMovimiento === 'TRANSFERENCIA') { @for (item of productosTransferenciaSelect; ...) }
    @else if (tipoMovimiento === 'SALIDA')   { @for (item of productosParaSelectSalida; ...) }
    @else                                    { @for (p of productosParaSelect; ...) }
  </select>
  <mat-icon class="select-caret">expand_more</mat-icon>
</div>
```

`.select-caret` tiene `pointer-events:none; position:absolute; right:.7rem` y gira 180° al `:focus-within`.

**Gotcha**: `[ngValue]` en `<option>` funciona porque `FormsModule` (ya importado) provee `SelectControlValueAccessor`. Necesario para valores `number | null` — `value=""` solo pasa strings.

### SALIDA expuesto como 3er tipo

`StockInventarioServicio.registrarMovimiento()` ya aceptaba `tipo: 'INGRESO' | 'SALIDA'`. El componente solo tenía `'INGRESO' | 'TRANSFERENCIA'`. Simplemente se extendió el tipo y se reutilizó el mismo endpoint.

```typescript
tipoMovimiento: 'INGRESO' | 'SALIDA' | 'TRANSFERENCIA' = 'INGRESO';
```

En `registrar()`: INGRESO y SALIDA comparten el mismo branch con `tipo: this.tipoMovimiento`. TRANSFERENCIA llama al endpoint separado `transferir()`.

`stockMaximoSeleccionado` ahora aplica para SALIDA también (validación de cantidad disponible):

```typescript
get stockMaximoSeleccionado(): number | null {
  const id = this.controles.productoId.value;
  if (!id) return null;
  if (this.tipoMovimiento !== 'TRANSFERENCIA' && this.tipoMovimiento !== 'SALIDA') return null;
  return this.stockItems().find(i => i.productoId === id)?.cantidad ?? null;
}
```

### KPI strip — señal `stockBajo`

```typescript
readonly stockBajo = computed(() =>
  this.stockItems().filter(i => i.cantidad <= 5).length
);
```

Muestra chip ámbar con ícono `warning_amber` solo cuando `stockBajo() > 0`. Los otros dos chips son siempre visibles al cargar.

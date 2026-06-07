---
titulo: Frontend — Patrones HTTP y Contexto
tipo: proyecto
tags: [inventario, angular, signals, rxjs, tenant, http]
fecha_creacion: 2026-06-07
fecha_actualizacion: 2026-06-07
estado: activo
---

# Patrones HTTP y Contexto

Sub-página de [[venta-inventario-frontend]].

## API Envelope — desempaquetado centralizado

El backend retorna `{ exito: true, mensaje: "...", datos: T }`. `HttpBaseService.desempaquetar<T>()` extrae `datos` automáticamente — los servicios reciben `T` directamente.

```typescript
// http-base.service.ts
private desempaquetar<T>(respuesta: unknown): T {
  if (respuesta && typeof respuesta === 'object' && !Array.isArray(respuesta)) {
    const r = respuesta as Record<string, unknown>;
    if ('datos' in r) return r['datos'] as T;
    if ('data' in r) return r['data'] as T;
  }
  return respuesta as T;
}
```

**Regla**: nunca usar `RespuestaApi<T>` ni `.map(r => r.datos)` en servicios — el tipo va directo en `get<T>()`.

```typescript
// ✓ correcto
return this.http.get<EntradaLogSesion[]>(`api/.../historial`);

// ✗ incorrecto — double-unwrap (ver gotcha en [[venta-inventario-frontend-gotchas]])
return this.http.get<RespuestaApi<Producto[]>>('api/...').pipe(map(r => r.datos ?? []));
```

## Tenant Isolation — lista de usuarios reactiva

`usuarios-lista.component.ts` recarga al cambiar el negocio activo:

```typescript
effect(() => {
  this.negocioContexto.negocioActivo();
  this.cargarUsuarios();
}, { allowSignalWrites: true });
```

Cuando el superadmin no tiene contexto: toast `warn` + `return` early antes de abrir modal.

## Patrón preferido: `toObservable + switchMap`

Para componentes con llamadas HTTP reactivas al contexto, usar `switchMap` — cancela peticiones previas y evita race conditions.

```typescript
import { toObservable } from '@angular/core/rxjs-interop';
import { filter, switchMap } from 'rxjs';

// En ngOnInit():
toObservable(this.negocioCtx.negocioActivo)
  .pipe(
    filter(n => n !== null),
    switchMap(() => {
      this.cargando.set(true);
      return this.servicio.obtener();
    }),
    takeUntilDestroyed(this.destroyRef),
  )
  .subscribe({
    next: (datos) => { /* aplicar datos, cargando.set(false) */ },
    error: () => { /* mensaje error, cargando.set(false) */ },
  });
```

| Escenario | Patrón recomendado |
|-----------|-------------------|
| Llamada HTTP que debe cancelar peticiones previas | `toObservable + switchMap` |
| Acción simple sin HTTP (limpiar array, resetear estado) | `effect()` |
| Guardia "sin contexto → mostrar vacío" | `computed(() => !this.negocioCtx.negocioActivo())` |

**Bug resuelto (2026-05-18)**: `MiPaginaComponent` usaba `ngOnInit() { this.cargar() }` — no reaccionaba al cambio de negocio. Fix: `toObservable + switchMap`.

## Guard de contexto en módulos de inventario

Productos, Sedes y Movimientos verifican `negocioActivo` antes de cargar datos.

```typescript
sinContexto = computed(() => !this.negocioContexto.negocioActivo());
```

```typescript
effect(() => {
  const negocio = this.negocioContexto.negocioActivo();
  if (!negocio) { this.datos.set([]); this.cargando.set(false); return; }
  this.cargarDatos();
}, { allowSignalWrites: true });
```

```html
@if (sinContexto()) {
  <!-- "Selecciona un negocio para ver..." -->
} @else if (cargando()) {
  <!-- skeleton -->
} @else {
  <!-- tabla / contenido -->
}
```

## Contexto auto-set para no-SUPERADMIN

**Problema**: para usuarios que no son SUPERADMIN, el dropdown de negocio no aparece en el sidebar — el contexto nunca se establecía.

**Solución**: al init del sidebar, si el rol no es SUPERADMIN, se lee `negocioId` del JWT y se llama `negocioContexto.establecer()` automáticamente.

```typescript
// barra-lateral.component.ts — ngOnInit
if (this.esSuperAdmin()) {
  this.cargarNegocios();
} else {
  this.autoEstablecerContextoNegocio();
}

private autoEstablecerContextoNegocio(): void {
  const negocioId = this.autenticacion.obtenerNegocioId();
  if (!negocioId) return;
  this.negocioServicio.buscarPorId(negocioId).subscribe({
    next: (negocio) => this.negocioContexto.establecer({ id: negocio.id, nombre: negocio.nombre }),
    error: () => {}
  });
}
```

**Archivos modificados**: `autenticacion.servicio.ts` (nuevo método `obtenerNegocioId()`), `negocio.servicio.ts` (`buscarPorId` extrae `datos` del envelope), `barra-lateral.component.ts`, `panel.component.ts` (banner `context-banner` solo para SUPERADMIN).

> Patrón: no usar `&&` con `as` en un solo `@if` con Angular. Usar `@if` anidados para evitar conflictos de tipo.

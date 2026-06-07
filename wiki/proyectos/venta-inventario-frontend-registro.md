---
titulo: Frontend — Registro, Guards y Autenticación
tipo: proyecto
tags: [inventario, angular, registro, guards, autenticacion, responsive]
fecha_creacion: 2026-06-07
fecha_actualizacion: 2026-06-07
estado: activo
---

# Registro, Guards y Autenticación

Sub-página de [[venta-inventario-frontend]].

## Pantalla de registro — bugs responsive (2026-05-31)

### Bug 1: grid no colapsa en mobile — formulario comprimido al 65%

**Causa**: `login-shell` mantiene `grid-template-columns: 35fr 65fr` aunque `.visual-column` tenga `display: none`. El grid reserva el 35% igual.

```css
@media (max-width: 768px) {
  .login-shell { grid-template-columns: 1fr; }
}
```

### Bug 2: subtitle comprimido por flex header con `justify-content: space-between`

**Causa**: `.form-header` es `display: flex; justify-content: space-between`. El link "¿Ya tienes cuenta?" comprime el div del título.

```css
@media (max-width: 768px) {
  .form-header { flex-direction: column; gap: 0.5rem; }
}
```

### Bug 3: scroll bloqueado en toda la app

**Causa**: `styles.css` global tiene `html, body { overflow: hidden }` — necesario para que el panel no scrollee. Pero afecta a todas las rutas incluyendo `/registro`.

```css
/* El componente maneja su propio scroll */
:host { display: block; height: 100vh; overflow-y: auto; }
```

**Regla**: cuando `body` tiene `overflow: hidden` global, las pantallas full-page sin panel (login, registro, verificar) deben poner `height: 100vh; overflow-y: auto` en `:host`.

## Guards de autenticación (2026-05-30)

### `guardiaAutenticacion` — ya existía

`src/app/core/guards/autenticacion.guardia.ts` — verifica JWT en storage. Si no existe → redirige a `/`.

### `guardiaNegocio` — nuevo

`src/app/core/guards/negocio.guardia.ts` — verifica `negocioId` en JWT. Si null → redirige a `/negocio/nuevo`.

```typescript
export const guardiaNegocio: CanActivateFn = () => {
  const servicioAutenticacion = inject(AutenticacionServicio);
  const router = inject(Router);
  const negocioId = servicioAutenticacion.obtenerNegocioId();
  return negocioId ? true : router.createUrlTree(['/negocio/nuevo']);
};
```

**Uso en rutas**: `/dashboard` y `/dashboard/:vista` tienen `canActivate: [guardiaAutenticacion, guardiaNegocio]`. La ruta `/negocio/nuevo` solo tiene `guardiaAutenticacion`.

## Flujo de registro en dos pasos (2026-05-30)

### Paso 1 — `/registro`

Componente simplificado: `nombreUsuario`, `correo`, `telefono` (requerido), `contrasena`, `confirmarContrasena`. Sin campos de negocio.

### Paso 2 — `/negocio/nuevo`

`NegocioRegistroComponent` (`features/negocio/negocio-registro/`). Campos: `nombre` (requerido), `ruc` (opcional), `correoContacto` (opcional).

Al enviar:
1. POST `/api/autenticacion/registrar-negocio`
2. Backend retorna nuevos tokens con `negocioId` real
3. `persistirTokens()` actualiza storage
4. `inicializarPermisos()` recarga permisos del JWT
5. Navega a `/dashboard`

### `AutenticacionServicio` — métodos relevantes

- `registrar(datos: DatosRegistro)` — `DatosRegistro` ya no incluye campos de negocio
- `registrarNegocio(datos: DatosRegistroNegocio)` — nuevo, persiste tokens automáticamente
- `obtenerNegocioId()` — retorna `number | null` (el null ahora tiene semántica real: sin negocio)

## Bug pendiente: contraseña pre-rellena en modal "Nuevo usuario"

Al abrir el dialog de crear usuario, el campo contraseña aparece con puntos (autocompletar del browser).

**Posibles fixes**:
1. `autocomplete="new-password"` en el input
2. `this.formulario.reset()` al abrir el modal
3. `<input type="password" style="display:none">` previo (hack, no recomendado)

## Movimientos — formulario inline convertido a modal

El formulario de registro de movimientos se migró de inline (debajo de la tabla) a `p-dialog` activado desde el header.

- Signal `formularioVisible = signal(false)`, métodos `abrirFormulario()` y `cerrar()`
- Al registrar exitoso: llamar `cerrar()` **antes** del reset de formulario

**Cambios CSS al migrar form a modal**:

| Antes (inline) | Después (modal) |
|---------------|-----------------|
| `.mov-form` | `.dialog-form` |
| `.mov-actions` | `.dialog-actions` |
| — | `.ghost-btn` |
| — | `.form-error` |

**Bug documentado**: el HTML se actualizó a `.dialog-form` pero no el CSS — las clases quedaron sin estilos. Siempre actualizar HTML y CSS en el mismo paso.

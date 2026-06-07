---
titulo: Venta e Inventario — Autenticación y JWT
tipo: proyecto
tags: [inventario, autenticacion, jwt, refresh-token, nestjs, registro, verificacion-correo, negocio-registro, recuperacion-contrasena, correo-verificado-login]
fecha_creacion: 2026-05-07
fecha_actualizacion: 2026-05-31
estado: activo
---

# Venta e Inventario — Autenticación y JWT

Sub-página de [[venta-inventario]].

## Módulos de autenticación (9)

- `AutenticacionModulo` — login, refresh, logout
- `UsuariosModulo` — CRUD de usuarios
- `RolesModulo` — CRUD de roles
- `PermisosModulo` — CRUD de permisos
- `SeccionesPermisoModulo` — agrupaciones de permisos
- `RolesPermisosModulo` — relación rol-permiso
- `UsuariosRolesModulo` — relación usuario-rol
- `UsuariosPermisosModulo` — permisos directos por usuario
- `RefreshTokensModulo` — gestión de tokens de refresco

## JWT Payload

```typescript
PayloadJwt {
  sub:           number        // usuario.id
  nombreUsuario: string
  negocioId:     number | null // null si el usuario aún no ha registrado su negocio
  nivelPlan:     number        // 1-4, valida acceso a secciones sin query extra
  permisos?:     string[]
}
```

**Expiración**: access token 15m, refresh token 30d.

**Importante**: `negocioId` puede ser `null` para usuarios recién registrados que aún no completaron el paso 2 (registro de negocio). El guard `guardiaNegocio` en el frontend detecta este estado y redirige a `/negocio/nuevo`.

## Tablas

### `usuarios`

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| negocio_id | BIGINT NULLABLE | → `negocio`, NULL hasta que registre negocio |
| nombre_usuario | VARCHAR(150) | UNIQUE |
| contrasena_hash | VARCHAR | bcrypt |
| correo | VARCHAR(150) | UNIQUE |
| telefono | VARCHAR(20) | NULLABLE — migración 021, requerido en registro |
| ultima_sesion | TIMESTAMPTZ | |
| correo_verificado | BOOLEAN | DEFAULT false — migración 020 |
| token_verificacion | VARCHAR(255) | NO se nullifica al verificar (ver nota abajo) |
| token_verificacion_expira | TIMESTAMPTZ | 24h desde registro |
| token_recuperacion | VARCHAR(255) | UUID para reset password — migración 023 |
| token_recuperacion_expira | TIMESTAMPTZ | 15 minutos desde solicitud — migración 023 |
| estado | BOOLEAN | |
| eliminado | BOOLEAN | |
| + auditoría estándar | | |

### `refresh_token`

| Campo | Tipo | Notas |
|-------|------|-------|
| jti | VARCHAR PK | UUID |
| usuario_id | BIGINT FK | |
| revocado | BOOLEAN | |
| expira | DATE | 30 días |
| reemplazado_por_jti | VARCHAR | trazabilidad de rotación |
| hash | VARCHAR | |
| ip | VARCHAR | |
| agente_usuario | VARCHAR | user-agent del cliente |

## Endpoints públicos

| Método | Ruta | Auth | Descripción |
|--------|------|------|-------------|
| POST | `/api/autenticacion/login` | ❌ | Login usuario/contraseña |
| POST | `/api/autenticacion/refresh` | ❌ | Renovar access token |
| POST | `/api/autenticacion/logout` | ❌ | Revocar refresh token |
| POST | `/api/autenticacion/registro` | ❌ | Registro solo usuario (sin negocio) |
| GET | `/api/autenticacion/verificar?token=UUID` | ❌ | Activar cuenta → retorna tokens (negocioId=null) |
| POST | `/api/autenticacion/registrar-negocio` | ✅ JWT | Crear negocio + vincular usuario → retorna tokens actualizados con negocioId |
| POST | `/api/autenticacion/recuperar-contrasena` | ❌ | Solicitar reset: genera token 15min + envía email |
| POST | `/api/autenticacion/restablecer-contrasena` | ❌ | Valida token + actualiza hash + limpia token |

## Flujo de registro — dos pasos (2026-05-30)

### Paso 1: Registro de usuario (`POST /registro`)

Cuerpo: `{ nombreUsuario, correo, contrasena, telefono }`

Lógica en `registrar()`:

1. Buscar por correo:
   - Si existe + `correoVerificado=true` → `ConflictException`
   - Si existe + `correoVerificado=false` → refrescar token, actualizar credenciales, reenviar correo
2. Verificar unicidad de `nombreUsuario` separado
3. Crear usuario con `negocioId=null`, `correoVerificado=false`, token 24h
4. Asignar rol `SUPERADMIN`
5. Enviar correo (fire & forget)
6. Retornar `{ verificacionPendiente: true }`

**Post-verificación**: JWT emitido con `negocioId=null`. Frontend redirige a `/negocio/nuevo`.

### Paso 2: Registro de negocio (`POST /registrar-negocio`)

Requiere JWT válido. Cuerpo: `{ nombre, ruc?, correoContacto? }`

Lógica en `registrarNegocio()`:

1. Verificar que `usuario.negocioId === null` — si ya tiene negocio → `ConflictException`
2. Crear negocio con `nivelPlan=1`
3. Actualizar `usuario.negocioId = negocio.id`
4. COMMIT → `usuario.reload()` → `emitirTokens()`
5. Retornar tokens actualizados (ahora con `negocioId` real)

Frontend persiste los nuevos tokens, inicializa permisos, navega a `/dashboard`.

## Variables de entorno

```env
JWT_EXPIRACION=15m
JWT_REFRESH_EXPIRACION=30d
JWT_SECRET
```

## Token de verificación — NO se nullifica al activar (2026-05-30)

**Decisión**: `tokenVerificacion` NO se pone a null después de verificar la cuenta.

**Razón**: si el usuario hace click al link de verificación por segunda vez, el backend busca por token. Si el token fuera null, no encontraría al usuario y lanzaría "token inválido" en lugar del mensaje correcto "ya verificada".

**Comportamiento correcto**: busca por token → encuentra usuario → revisa `correoVerificado=true` → lanza el mensaje informativo.

```typescript
// autenticacion.servicio.ts — verificarCorreo()
await usuario.update({ correoVerificado: true });
// NO se nullifica tokenVerificacion ni tokenVerificacionExpira
```

---

## Verificar correo — detección de cuenta ya verificada (2026-05-30)

**Problema original**: `verificarCorreo()` buscaba con `correoVerificado: false`. Si la cuenta ya estaba verificada, no encontraba al usuario y lanzaba el mismo error "Token inválido o ya utilizado" — sin distinguir los dos casos.

**Fix backend** — `autenticacion.servicio.ts`:
```typescript
// Antes: incluía correoVerificado: false en el where → no distinguía casos
// Ahora: busca por token solo, verifica correoVerificado explícito
const usuario = await this.usuarioModelo.findOne({
  where: { tokenVerificacion: token, eliminado: false },
});
if (!usuario) throw new BadRequestException('Token inválido o ya utilizado');
if (usuario.correoVerificado) {
  throw new BadRequestException('Tu cuenta ya fue verificada, puedes iniciar sesión');
}
```

**Fix frontend** — `verificar-correo.component.ts`:
- Nuevo estado `'ya-verificado'` en el type `EstadoVerificacion`
- Error handler detecta el mensaje por substring `'ya fue verificada'` → estado `'ya-verificado'`
- Template: bloque `@if (estado() === 'ya-verificado')` con ícono verde + "Ir a iniciar sesión"
- CSS: `.icon-wrap.success` con `rgba(39, 160, 90, 0.12)` + `color: #50d282`

---

## Flujo recuperación de contraseña — "Olvidé mi contraseña" (2026-05-31)

### Migración: 023-token-recuperacion-contrasena.sql

```sql
ALTER TABLE usuarios
  ADD COLUMN IF NOT EXISTS token_recuperacion        VARCHAR(255),
  ADD COLUMN IF NOT EXISTS token_recuperacion_expira TIMESTAMP WITH TIME ZONE;
```

### DTOs

- `SolicitarRecuperacionDto`: `correo: string` (`@IsEmail`)
- `RestablecerContrasenaDto`: `token: string`, `nuevaContrasena: string` (`@MinLength(8)`)

### Servicio backend

**`solicitarRecuperacion(dto)`** — patrón anti-enumeración:
```typescript
// Siempre retorna OK aunque el correo no exista
if (usuario) {
  token = randomUUID();
  expira = Date.now() + 15 * 60 * 1000;
  await usuario.update({ tokenRecuperacion: token, tokenRecuperacionExpira: expira });
  void correoServicio.enviarRecuperacionContrasena(correo, token);
}
return { mensaje: 'Si el correo está registrado, recibirás instrucciones...' };
```

**`restablecerContrasena(dto)`**:
1. Busca usuario por `tokenRecuperacion = token` + `eliminado: false`
2. No existe → `BadRequestException('Token inválido o ya utilizado')`
3. `tokenRecuperacionExpira < now` → `BadRequestException('El enlace de recuperación ha expirado')`
4. Hash nueva contraseña con bcrypt
5. Limpia `tokenRecuperacion = null`, `tokenRecuperacionExpira = null`

### Rutas frontend

| Ruta | Componente | Descripción |
|------|-----------|-------------|
| `/recuperar-contrasena` | `RecuperarContrasenaComponent` | Formulario de email → muestra confirmación in-place |
| `/restablecer-contrasena?token=UUID` | `RestablecerContrasenaComponent` | Formulario nueva + confirmar contraseña |

**Flujo de redirección**:
- `/restablecer-contrasena` sin `?token` → redirige automáticamente a `/recuperar-contrasena`
- Tras éxito: navega a `/` con `?mensaje=contrasena-restablecida` → login muestra banner verde
- Link "Olvide mi contrasena" visible en el formulario de login (entre el checkbox y el botón)

### Fix URL en correo servicio

```typescript
// Antes (incorrecto — apuntaba a la pantalla de solicitud, no de reset)
const url = `${this.appUrl}/recuperar-contrasena?token=${token}`;
// Después (correcto)
const url = `${this.appUrl}/restablecer-contrasena?token=${token}`;
```

### Patrón de seguridad: anti-enumeración

El endpoint `POST /recuperar-contrasena` **siempre responde 200** con el mismo mensaje, independientemente de si el correo existe o no. Esto evita que un atacante descubra qué correos están registrados.

---

## Validación correo verificado en login (2026-05-31)

**Problema**: `iniciarSesion()` no verificaba `correoVerificado` — usuarios registrados sin activar podían obtener tokens válidos.

**Fix backend** — `autenticacion.servicio.ts`, dentro de `iniciarSesion()` después de validar contraseña:

```typescript
if (!usuario.correoVerificado) {
  const tokenActivo =
    !!usuario.tokenVerificacionExpira &&
    new Date() <= usuario.tokenVerificacionExpira;
  throw new ForbiddenException({
    message: tokenActivo
      ? 'Debes verificar tu correo antes de iniciar sesión. Revisa tu bandeja de entrada.'
      : 'El enlace de verificación expiró. Vuelve a registrarte para recibir uno nuevo.',
    codigo: tokenActivo ? 'CORREO_NO_VERIFICADO_ACTIVO' : 'CORREO_NO_VERIFICADO_EXPIRADO',
  });
}
```

- **403 ForbiddenException** — las credenciales son válidas pero el acceso está bloqueado. Usar 403 (no 401) para que el interceptor de refresh no interfiera.
- **`codigo`** — campo nuevo en la respuesta de error, propagado por `ExcepcionHttpFiltro` (ver abajo).

**Fix filtro** — `excepcion-http.filtro.ts`:

```typescript
// respuestaExcepcion elevada al scope del método (antes era local al bloque if)
let respuestaExcepcion: unknown = null;

// En el JSON de respuesta:
codigo:
  typeof respuestaExcepcion === 'object' && respuestaExcepcion !== null
    ? ((respuestaExcepcion as Record<string, unknown>).codigo ?? null)
    : null,
```

**Fix frontend** — `inicio-sesion.component.ts`, handler de error:

```typescript
const codigo: string | undefined = error?.error?.codigo;
if (codigo === 'CORREO_NO_VERIFICADO_ACTIVO') {
  void this.router.navigate(['/registro/pendiente']);
  return;
}
this.errorInicioSesion =
  error?.error?.mensaje ||
  (codigo === 'CORREO_NO_VERIFICADO_EXPIRADO'
    ? 'Tu enlace de verificación expiró. Vuelve a registrarte para recibir uno nuevo.'
    : 'No fue posible iniciar sesion.');
```

**Comportamiento final**:
- Token activo → redirige a `/registro/pendiente` (ya existe, dice "revisa tu bandeja de entrada")
- Token expirado → error inline en login (el link "Registrate →" ya está visible en la página)

---

## Nota: BIGINT como string en Sequelize

Sequelize devuelve `BIGINT` como `string` en JS para evitar pérdida de precisión. Consecuencia: el JWT puede mostrar `"sub": "1"` (string) en vez de número.

Fix evaluado: `Number(usuario.id)` al construir payload. **Estado: revertido en 2026-05-07, pendiente de aplicar.**

```typescript
const payload: PayloadJwt = {
  sub: Number(usuario.id),
  negocioId: Number(usuario.negocioId),
};
```

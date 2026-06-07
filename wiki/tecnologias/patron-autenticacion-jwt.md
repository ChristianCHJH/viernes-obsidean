---
titulo: Patrón de Autenticación JWT + Refresh Tokens
tipo: tecnologia
tags: [jwt, refresh-token, autenticacion, nestjs, angular, rbac, patron, seguridad]
fecha_creacion: 2026-05-06
fecha_actualizacion: 2026-05-06
estado: activo
---

# Patrón de Autenticación JWT + Refresh Tokens

El patrón estándar de autenticación que Christian usa en todos sus proyectos fullstack. Implementado primero en [[autenticacion-prototipo]] y replicado en [[venta-inventario]] y otros.

## Decisiones de diseño

| Decisión | Valor | Razón |
|----------|-------|-------|
| Duración JWT | 15 minutos | Corta para limitar exposición si se compromete |
| Duración Refresh | 30 días | Larga para no interrumpir la UX |
| Storage del Refresh | Base de datos | Permite revocación activa |
| Primary key del Refresh | UUID (JTI) | Estándar JWT, opaco para el cliente |
| Hash de contraseña | bcrypt | Estándar de la industria |
| Permisos en JWT | NO | Se calculan dinámicamente en cada request |
| Protección por defecto | Global (opt-out) | Seguro por defecto, `@Publica()` para excepciones |

## Backend (NestJS)

### Módulos necesarios

```
autenticacion/
├── autenticacion.modulo.ts    ← JwtModule.registerAsync, RefreshTokensModulo
├── autenticacion.servicio.ts  ← iniciarSesion, refrescarToken, cerrarSesion, emitirTokens
├── autenticacion.controlador.ts
├── decoradores/
│   ├── publica.decorador.ts          ← @Publica() → SetMetadata('esPublica', true)
│   ├── permisos.decorador.ts         ← @Permisos(...) → SetMetadata('permisosRequeridos', [...])
│   └── usuario-actual.decorador.ts   ← @UsuarioActual() → req.user.sub
├── estrategias/jwt.estrategia.ts     ← Passport JWT + resolución de permisos
└── guardias/
    ├── jwt.guardia.ts                ← AuthGuard('jwt') + respeta @Publica()
    └── permisos.guardia.ts           ← valida permisos del usuario
refresh-tokens/
├── refresh-tokens.servicio.ts        ← crear, validarVigencia, revocar
└── refresh-tokens.modulo.ts
```

### Configuración global obligatoria

```typescript
// modulo-aplicacion.ts
providers: [
  { provide: APP_GUARD, useClass: JwtGuardia }  // Protege todo por defecto
]
```

### Payload del JWT

```typescript
interface PayloadJwt {
  sub: number;           // ID del usuario
  nombreUsuario: string;
}
```

El JWT intencionalmente **no incluye permisos** — se calculan en `JwtEstrategia.validate()` en cada request consultando BD. Esto permite revocar permisos sin invalidar el token.

### Cómo se resuelven los permisos (JwtEstrategia)

```
1. Busca roles del usuario → UsuarioRol (estado=true)
2. Busca permisos directos → UsuarioPermiso → permiso.permiso
3. Para cada rol: busca permisos del rol → RolPermiso → permiso.permiso
4. Une todo en Set<string> (deduplicación automática)
5. req.user = { sub, nombreUsuario, permisos: string[] }
```

### Tabla `refresh_token` (estructura mínima)

```sql
CREATE TABLE refresh_token (
    jti                  VARCHAR PRIMARY KEY,
    usuario_id           BIGINT NOT NULL,
    revocado             BOOLEAN DEFAULT false,
    expira_en            TIMESTAMPTZ NOT NULL,
    reemplazado_por_jti  VARCHAR,        -- trazabilidad de rotación
    hash                 VARCHAR,
    ip                   VARCHAR,
    agente_usuario       VARCHAR,
    creado_en            TIMESTAMPTZ DEFAULT NOW(),
    actualizado_en       TIMESTAMPTZ DEFAULT NOW()
);
```

## Frontend (Angular)

### Archivos necesarios

```
core/
├── interceptors/autenticacion.interceptor.ts  ← auto-attach JWT, auto-refresh 401
├── guards/
│   ├── autenticacion.guardia.ts              ← redirige a '/' si no autenticado
│   └── permiso.guardia.ts                    ← redirige a '/dashboard' si sin permiso
└── services/
    ├── permisos-usuario.servicio.ts           ← Signals con permisos del usuario
features/autenticacion/
├── autenticacion.servicio.ts                  ← login, refresh, logout, persistencia
└── inicio-sesion/inicio-sesion.component.ts
```

### Clave de almacenamiento

```typescript
'spa.auth.tokens' // en localStorage (recordarme=true) o sessionStorage (recordarme=false)
```

### Lógica del interceptor (patrón crítico)

```typescript
// Variable compartida fuera del interceptor — evita múltiples refresh concurrentes
let refrescarTokenEnProgreso$: Observable<...> | null = null;

// En el pipe del interceptor:
if (!refrescarTokenEnProgreso$) {
  refrescarTokenEnProgreso$ = servicioAuth.refrescarTokens().pipe(
    shareReplay(1),
    finalize(() => { refrescarTokenEnProgreso$ = null; })
  );
}
return refrescarTokenEnProgreso$.pipe(switchMap(() => next(requestConNuevoToken)));
```

`shareReplay(1)` es clave: si 3 requests fallan con 401 simultáneamente, solo se hace **un** call de refresh. Las otras 2 esperan y comparten el resultado.

### Decodificación del JWT (sin librería)

```typescript
const payload = JSON.parse(atob(token.split('.')[1].replace(/-/g, '+').replace(/_/g, '/')));
// payload.sub → userId
// payload.nombreUsuario → nombre
```

## Proyectos que usan este patrón

- [[autenticacion-prototipo]] — implementación de referencia (TypeORM)
- [[venta-inventario]] — implementado con Sequelize en lugar de TypeORM
- [[proyecto-spa]] — también implementa este patrón (por verificar detalles)

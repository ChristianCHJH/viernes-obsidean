---
titulo: Prototipo de Autenticación (Template base)
tipo: proyecto
tags: [autenticacion, jwt, refresh-token, rbac, nestjs, typeorm, angular, prototipo, template]
fecha_creacion: 2026-05-06
fecha_actualizacion: 2026-05-06
estado: activo
---

# Prototipo de Autenticación

Sistema de autenticación JWT + Refresh Tokens con RBAC completo. Es el **template de referencia** que se replica en proyectos como [[venta-inventario]] y otros.

> Estado: aunque inicialmente estaba marcado como "archivado", el código está completamente implementado y es la base del patrón de autenticación de Christian.

## Stack

- **Backend**: NestJS 10 + TypeORM 0.3 + PostgreSQL
- **Frontend**: Angular 18.2 + PrimeNG + RxJS 7.8
- **Auth**: JWT (15m) + Refresh Tokens UUID (30d) + bcrypt
- **Docs**: Swagger/OpenAPI

## Arquitectura del backend

### Estructura de módulos

```
src/
├── modulo-aplicacion.ts        ← Módulo raíz, JwtGuardia global
├── autenticacion/
│   ├── autenticacion/
│   │   ├── autenticacion.servicio.ts
│   │   ├── autenticacion.controlador.ts
│   │   ├── autenticacion.modulo.ts
│   │   ├── decoradores/
│   │   │   ├── publica.decorador.ts       ← @Publica()
│   │   │   ├── permisos.decorador.ts      ← @Permisos(...)
│   │   │   └── usuario-actual.decorador.ts ← @UsuarioActual()
│   │   ├── dto/
│   │   │   ├── iniciar-sesion.dto.ts
│   │   │   ├── refrescar-token.dto.ts
│   │   │   └── cerrar-sesion.dto.ts
│   │   ├── estrategias/jwt.estrategia.ts  ← Passport JWT + enriquecimiento de permisos
│   │   └── guardias/
│   │       ├── jwt.guardia.ts             ← Guard global
│   │       └── permisos.guardia.ts        ← Guard por permiso
│   ├── usuarios/
│   ├── refresh-tokens/
│   ├── roles/, permisos/, secciones-permiso/
│   ├── usuarios-roles/, usuarios-permisos/, roles-permisos/
└── comun/
    ├── filtros/      ← HttpExceptionFilter global
    └── interceptores/ ← TransformInterceptor global
```

## Flujo completo de autenticación

### Login
```
POST /autenticacion/login  (@Publica())
├─ Acepta: { correo } o { nombreUsuario } + { contrasena }
├─ Valida bcrypt.compare(contrasena, contrasenaHash)
├─ Actualiza ultimaSesion
├─ Genera JWT firmado: payload = { sub: usuarioId, nombreUsuario }
│  Expiración: 15m (JWT_EXPIRACION)
├─ Genera UUID → JTI del refresh token
├─ Persiste en BD: refresh_token { jti, usuarioId, expira: +30d, revocado: false }
└─ Retorna: { tokenAcceso, tipoToken: 'Bearer', expiraEn: '15m', refreshToken: jti }
```

### Request protegida
```
Authorization: Bearer <JWT>
├─ JwtGuardia (global) valida firma y expiración
├─ JwtEstrategia.validate(payload):
│  ├─ Busca roles del usuario (UsuarioRol WHERE estado=true)
│  ├─ Busca permisos directos (UsuarioPermiso → permiso.permiso)
│  ├─ Busca permisos heredados de roles (RolPermiso → permiso.permiso)
│  └─ Retorna req.user = { sub, nombreUsuario, permisos: string[] }
└─ PermisosGuardia (si hay @Permisos(...)): valida que el usuario tenga al menos uno
```

### Refresh token
```
POST /autenticacion/refresh  (@Publica())
├─ Recibe: { refreshToken: jti }
├─ Busca en BD: valida NOT revocado AND NOT expirado
├─ Emite nuevos tokens (JWT nuevo + JTI nuevo)
├─ Marca JTI anterior como revocado (con reemplazado_por_jti = nuevo JTI)
└─ Retorna nuevos tokens
```

### Logout
```
POST /autenticacion/logout  (@Publica())
├─ Recibe: { refreshToken: jti }
├─ Marca refresh_token.revocado = true
└─ Retorna: { mensaje: 'Sesión cerrada correctamente' }
```

## Decoradores clave (backend)

```typescript
// Marca una ruta como pública (sin JWT)
@Publica()

// Requiere permisos específicos (al menos uno)
@Permisos('crear-producto', 'editar-producto')

// Inyecta el ID del usuario autenticado en el parámetro
@UsuarioActual() usuarioId: number
```

## Configuración global (modulo-aplicacion.ts)

```typescript
providers: [
  { provide: APP_GUARD, useClass: JwtGuardia }  // Todas las rutas protegidas por defecto
]
```

Las rutas públicas se marcan con `@Publica()` — es opt-out, no opt-in.

## Modelo de permisos RBAC

```
SeccionPermiso (inventario, usuarios, reportes)
  └─ Permiso (crear-producto, ver-sedes, registrar-movimiento)
      ├─ RolPermiso ──> Rol (administrador, operario, gerente)
      │                   └─ UsuarioRol ──> Usuario
      └─ UsuarioPermiso ──> Usuario (permisos directos)
```

Los permisos se resuelven al validar el JWT — no se guardan en el token.

## Arquitectura del frontend

### Flujo del interceptor

```
Request saliente
├─ Si es login/refresh: NO agrega Authorization header
└─ Si es protegida: agrega Authorization: Bearer <JWT>

Response
└─ Si error 401:
   ├─ Obtiene refreshToken del storage
   ├─ Llama /autenticacion/refresh (shareReplay — evita múltiples calls concurrentes)
   ├─ Persiste nuevos tokens
   ├─ Reintenta request original con nuevo JWT
   └─ Si refresh falla: cerrarSesion() + propaga error
```

### Persistencia de tokens

```typescript
clave: 'spa.auth.tokens'  // en localStorage o sessionStorage

{
  tokenAcceso: string,    // JWT
  tokenRefresco: string,  // JTI
  tipoToken: 'Bearer',
  expiraEn: '15m'
}
```

- `recordarme: true` → localStorage (persiste entre sesiones)
- `recordarme: false` → sessionStorage (solo sesión actual)

### Decodificación JWT en frontend

Sin librería externa — decodifica el payload manualmente:
```typescript
const payload = JSON.parse(atob(token.split('.')[1]));
// Extrae: payload.sub (userId), payload.nombreUsuario
```

### Servicio de permisos (Signals)

```typescript
// PermisosUsuarioServicio usa Angular Signals
private readonly permisosUsuario = signal<string[]>([]);

tienePermiso(permiso: string): boolean
tieneAlgunPermiso(permisos: string[]): boolean
tieneTodosLosPermisos(permisos: string[]): boolean
```

### Guards de Angular

```typescript
// guardiaAutenticacion: redirige a '/' si no está autenticado
canActivate: [guardiaAutenticacion]

// guardiaPermiso: redirige a '/dashboard' si no tiene permisos
canActivate: [guardiaPermiso('crear-producto', 'editar-producto')]
```

## Variables de entorno

```env
# Backend
JWT_SECRETO=tu_secreto_fuerte
JWT_EXPIRACION=15m
JWT_REFRESH_EXPIRACION=30d
DB_HOST, DB_PORT, DB_USER, DB_PASS, DB_NAME, DB_SCHEMA
PORT=3000
```

## Por qué es el template base

Este prototipo establece el patrón que se replica en todos los proyectos de Christian:

1. **JwtGuardia global + @Publica()** — protección opt-out
2. **JwtEstrategia con enriquecimiento de permisos** — los permisos se resuelven en cada request, no se guardan en el JWT
3. **Refresh tokens en BD** con revocación y trazabilidad (`reemplazado_por_jti`)
4. **Interceptor Angular con shareReplay** — evita múltiples llamadas de refresh concurrentes
5. **Nomenclatura en español** — módulos, servicios, DTOs, guardias

## Links relacionados

- [[venta-inventario]] — implementa este mismo patrón (con Sequelize)
- [[tecnologias/patron-autenticacion-jwt]] — documentación centralizada del patrón
- [[proyecto-spa]] — también implementa este patrón

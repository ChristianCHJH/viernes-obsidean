---
titulo: Venta e Inventario — App Android (Adifnex)
tipo: proyecto
tags: [inventario, android, kotlin, jetpack-compose, mvvm, adifnex]
fecha_creacion: 2026-05-07
fecha_actualizacion: 2026-05-07
estado: activo
---

# Venta e Inventario — App Android (Adifnex)

Sub-página de [[venta-inventario]].

## Stack

- **Application class**: `AdifnexApp`
- **Activity**: `MainActivity` (Jetpack Compose)
- **Arquitectura**: MVVM + Repository pattern + DI manual (NetworkModule)
- **Keystore**: `adifnex.jks` en `apps/inventario/`

## Pantallas

| Pantalla | ViewModel |
|----------|-----------|
| Splash | SplashViewModel |
| Login | LoginScreen + LoginViewModel |
| Dashboard | DashboardScreen + DashboardViewModel |

## Endpoints que consume

```kotlin
POST /autenticacion/login
POST /autenticacion/refresh   // refresh automático vía TokenAuthenticator
POST /autenticacion/logout
GET  /autenticacion/perfil
GET  /inventario/productos
GET  /inventario/sedes
GET  /inventario/stock/estadisticas-dashboard
GET  /inventario/stock/movimientos
```

## TokenAuthenticator

Maneja el refresh automático en `OkHttp`. Cuando el servidor retorna 401:
1. Llama automáticamente a `/autenticacion/refresh`
2. Actualiza el token en almacenamiento local
3. Reintenta la request original con el nuevo token

Es el equivalente Android del `HttpInterceptor` de Angular que hace refresh automático.

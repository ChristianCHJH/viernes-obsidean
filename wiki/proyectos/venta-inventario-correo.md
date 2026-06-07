---
titulo: Venta e Inventario — Módulo Correo
tipo: proyecto
tags: [correo, gmail, smtp, nodemailer, transaccional, verificacion]
fecha_creacion: 2026-05-29
fecha_actualizacion: 2026-05-31
estado: activo
---

# Módulo Correo — Adifnex

## Decisión de proveedor

Gmail SMTP (`adifnex@gmail.com`) con App Password. Volumen máximo: ~30 emails/día. No justifica proveedor externo (Resend, SendGrid) en esta etapa.

**Si el volumen crece >500/día**: migrar a Resend (3,000/mes gratis) — solo cambia `CorreoServicio`, nada más.

## Configuración

Variables en `backend/.env`:
```
GMAIL_USER=adifnex@gmail.com
GMAIL_APP_PASSWORD=xdsahfcgflcjpubg
CORREO_REMITENTE=adifnex@gmail.com
APP_URL=http://localhost:4200
```

App Password generada el 2026-05-29 para la app `adifnex-backend`.

## Arquitectura

- `src/correo/correo.servicio.ts` — lógica de envío con nodemailer
- `src/correo/correo.modulo.ts` — `@Global()`, exporta `CorreoServicio`
- Registrado en `modulo-aplicacion.ts` antes de `AutenticacionModulo`

## Métodos disponibles

| Método | Uso |
|--------|-----|
| `enviarBienvenida(email, nombreNegocio)` | Post-registro autoservicio (fire & forget) |
| `enviarVerificacionCorreo(email, nombreUsuario, token)` | Activación de cuenta — enlace 24h (**implementado 2026-05-30**) |
| `enviarRecuperacionContrasena(email, token)` | Reset password — enlace 15min (**implementado 2026-05-31**) |

## Flujo de verificación de correo (2026-05-30)

- Enlace: `${APP_URL}/verificar?token=UUID`
- El token se genera con `randomUUID()` en el registro y se guarda en `usuarios.token_verificacion`
- Expira 24h desde creación (`token_verificacion_expira`)
- Se llama **fire & forget** (fuera de la transacción): si el correo falla, el registro no se revierte
- Al hacer clic: `GET /api/autenticacion/verificar?token=...` → valida, limpia token, emite JWT (login automático)

## Flujo recuperación de contraseña (2026-05-31)

- Enlace: `${APP_URL}/restablecer-contrasena?token=UUID` (**nota**: URL corregida — antes apuntaba a `/recuperar-contrasena`)
- Token generado con `randomUUID()`, guardado en `usuarios.token_recuperacion`
- Expira **15 minutos** desde la solicitud (`token_recuperacion_expira`)
- Se limpia al restablecer correctamente (`null` en ambas columnas)
- Patrón anti-enumeración: endpoint siempre responde 200 aunque el correo no exista

## Pendiente

- Endpoint de prueba `POST /api/correo/test` — **eliminar antes de producción**

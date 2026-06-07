---
titulo: Railway — Plataforma de despliegue
tipo: tecnologia
tags: [railway, deploy, hosting, produccion, paas]
fecha_creacion: 2026-05-24
fecha_actualizacion: 2026-05-29
estado: archivado
---

# Railway

~~Plataforma PaaS usada para desplegar el backend de venta-inventario.~~ **Reemplazada por Render (2026-05-29)** — trial se agotó.

## Estado de la cuenta

- Plan: Trial (agotado)
- Créditos al cierre (2026-05-29): $0 — servicio migrado a [[tecnologias/render]]

## Gotchas críticos

### Variables en morado = cambios pendientes sin aplicar

Cuando se edita una variable de entorno en Railway, la variable aparece resaltada en **morado**. Esto significa que el cambio **aún no está activo** — el servicio sigue corriendo con el valor anterior.

**Para que el cambio tome efecto**: hacer clic en el botón **Deploy** (esquina superior derecha) después de modificar variables. Sin ese paso, el servicio no ve el nuevo valor aunque la UI lo muestre actualizado.

Esto causó confusión en la sesión 2026-05-24: `STORAGE_PROVIDER=cloudinary` estaba seteado en la UI pero el servicio seguía usando `local` hasta que se hizo un nuevo deploy.

### Variables de entorno requeridas en producción

Para el backend de venta-inventario:

```env
STORAGE_PROVIDER=cloudinary        # NUNCA omitir — default era 'local' antes del fix
CLOUDINARY_CLOUD_NAME=dovxgcpc5
CLOUDINARY_API_KEY=<de la consola de Cloudinary>
CLOUDINARY_API_SECRET=<de la consola de Cloudinary>
BACKEND_URL=https://<dominio-railway>
```

### Verificar credenciales Cloudinary

El `cloud_name` correcto aparece en la URL del Cloudinary Media Library: `https://cloudinary.com/console/c-<hash>/media_library` — el valor de `cloud_name` es el que aparece en el panel principal, no el hash de la URL.

Error frecuente: copy-paste de `CLOUDINARY_API_KEY` o `CLOUDINARY_API_SECRET` con espacios o caracteres extra → el servidor devuelve 500 con error genérico de Cloudinary.

## Links relacionados

- [[proyectos/venta-inventario-catalogo]] — StorageServicio Cloudinary/local

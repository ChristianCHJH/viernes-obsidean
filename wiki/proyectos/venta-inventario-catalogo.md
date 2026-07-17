---
titulo: Venta e Inventario — Catálogo Público Multi-Negocio (adifnex-catalogo)
tipo: proyecto
tags: [catalogo, nextjs, react, tailwind, landing, multi-negocio, plan2, permisos, adifnex]
fecha_creacion: 2026-05-15
fecha_actualizacion: 2026-06-30
estado: activo
---

# Catálogo Público Multi-Negocio (adifnex-catalogo)

Nuevo repositorio Next.js separado del mono-repo principal. Sirve páginas de catálogo públicas para cada negocio registrado en Adifnex. Sin login — acceso libre.

## Stack

| Campo | Valor |
|---|---|
| Repo | `adifnex-catalogo` (submodule en `/catalogo`) |
| GitHub | `ChristianCHJH/inventario-multisede-catalogo-react` |
| Stack | Next.js 14 + React 18 + TypeScript 5 + Tailwind CSS |
| Router | App Router (Next 14) |
| Puerto dev | 3010 (3001 ocupado por EMR.Patient.Service-dev) |
| Docker | `docker compose -f docker-compose.dev.yml up catalogo` |
| URL dev | `http://localhost:3010/lullaby-soft` (usa mock si backend no disponible) |

## Qué es una "plantilla" — definición crítica

> ⚠️ **Reencuadrado el 2026-06-30** — ver sección [Refactor 2026-06-30 — Unificación + modelo de "Temas"](#refactor-2026-06-30--unificación-de-plantillas--modelo-de-temas). La estructura/layout dejó de ser distinta por plantilla: ahora **todas comparten el mismo shell** y solo cambian **fuente + formato de card + color**. El término "plantilla" pasó a llamarse **"Tema"** en la UI. Esta sección histórica se conserva como contexto del modelo original (2 plantillas con código duplicado).

Una plantilla **NO es un cambio de colores**. Es un **diseño completamente distinto**: layout, paleta, tipografía y componentes son diferentes entre plantillas. Lo que sí es homogéneo es el **modelo de datos** — todas consumen los mismos endpoints y el mismo contrato TypeScript.

| Distinto por plantilla | Compartido entre todas |
|---|---|
| Layout y estructura visual | `NegocioConfig`, `ProductoPublico` |
| Paleta de colores | Endpoints `/pub/:slug` |
| Tipografía | Tabla `negocio_pagina_config` |
| Componentes (Navbar, Cards, Footer) | Sistema de permisos |

Plantillas activas:

- `catalogo-grilla` — mint #366758, crema #fbf9f5, Quicksand + Nunito Sans, borders pill-shape, **sin hero** (eliminado 2026-06-16), filtros en navbar (chips horizontales)
- `catalogo-esencias` ✅ 2026-05-24 — dorado #b8975c, ivory #f7f3ee, espresso #1a1209, Playfair Display + Nunito Sans, sidebar filtros a la izquierda, imagen vertical 3:4, **sin hero** (eliminado 2026-06-16)

## Selección de plantilla — mecanismo

Campo `plantilla: string | null` en `NegocioConfig` (y en BD `negocio_pagina_config.plantilla`). El router en `app/[slug]/page.tsx` lo lee:

```tsx
if (config.plantilla === 'esencias') return <CatalogoEsencias ... />
return <CatalogoGrilla ... /> // default
```

El backend incluye el campo en el `ConfigPublicaDto`. El panel "Mi Página" expone la selección con permiso `CATALOGO_PLANTILLA_ELEGIR` (ya existía en el plan — aún pendiente de implementar en el panel Angular).

## Estructura de carpetas

```
adifnex-catalogo/
├── app/
│   ├── [slug]/          ← router: lee config.plantilla → renderiza template
│   └── layout.tsx       ← Quicksand + NunitoSans + Playfair Display (--font-display)
├── templates/
│   ├── catalogo-grilla/ ← plantilla 1: lullaby-soft, min/verde, aspect 4:3
│   ├── catalogo-esencias/ ← plantilla 2: perfumes/colonias, dorado, aspect 3:4
│   ├── catalogo-lista/  ← futura
│   ├── landing-simple/  ← futura
│   └── portfolio/       ← futura
├── lib/
│   ├── api.ts           ← fetch a /pub/:slug
│   └── types.ts         ← NegocioConfig (+plantilla), ProductoPublico, EtiquetaPublica
└── public/
    └── placeholder-product.svg
```

## Gap analysis — BD existente vs. necesidades del catálogo

### Tabla `producto` — columnas que FALTAN

| Campo | Tipo | Por qué |
|---|---|---|
| `precio` | `NUMERIC(10,2)` | Sin precio → producto no aparece en catálogo |
| `precio_oferta` | `NUMERIC(10,2) NULL` | Precio tachado con descuento |
| `visible_catalogo` | `BOOLEAN DEFAULT false` | Control de publicación |
| `orden_catalogo` | `INTEGER NULL` | Orden manual de destacados |

### Nueva tabla `producto_imagen` (no existe)

Galería de imágenes por producto. Campos: `id`, `producto_id`, `url`, `alt`, `orden`, `es_principal`, + auditoría estándar.

### Nueva tabla `negocio_pagina_config` (no existe — cero SQL)

Configuración de la página pública por negocio. Solo almacena **contenido de información**, no colores ni plantilla — esos son fijos en el CSS de cada plantilla.

Grupos de campos:
- **Identificación**: `slug`, `dominio_propio`, `activo`
- **Identidad visual**: `logo_url`, `favicon_url`, `nombre_publico`
- **Hero**: `hero_titulo`, `hero_subtitulo`, `hero_imagen_url`, `hero_cta_texto`, `hero_cta_url`
- **Footer & Contacto**: `footer_descripcion`, `whatsapp_numero`, `email_contacto`, `direccion_publica`, `horario_atencion`, `instagram_url`, `facebook_url`, `tiktok_url`, `footer_newsletter`, `footer_links_json`
- **SEO**: `meta_titulo`, `meta_descripcion`

**Campos ELIMINADOS de la propuesta original**: `color_primario`, `color_fondo`, `color_texto`, `layout_template`. Razón: los colores son fijos en el CSS de cada plantilla. Cada negocio elige una plantilla — no personaliza colores individualmente.

## Migración 014

Crea las tres piezas anteriores. Archivo de referencia: `docs/catalogo-landing/plan-catalogo-react.md` sección 3.

## Módulo `publico/` — F4 ✅

Sin autenticación — decorador `@Publica()` en cada endpoint (usa `APP_GUARD` global con `JwtGuardia` + detector de metadata `ES_PUBLICA`). Archivos: `publico.modulo.ts`, `publico.controlador.ts`, `publico.servicio.ts`, 3 DTOs.

```
GET /pub/:slug              → ConfigPublicaDto (NegocioConfig completa — excluye auditoría)
GET /pub/:slug/productos    → ProductoPublico[] (visible_catalogo=true, precio NOT NULL, con imagenes[] y etiquetas[])
GET /pub/:slug/categorias   → CategoriaPublicaDto[] (etiquetas con ≥1 producto visible)
```

Decisión impl: `resolverNegocioId(slug)` privado — los 3 endpoints lo reusan para obtener `negocioId` desde `NegocioPaginaConfig`. Productos ordenados por `orden_catalogo ASC NULLS LAST` usando `literal()` de Sequelize (único modo para NULLS LAST en PG).

## Contrato TypeScript (F5 ✅)

Después de F5 los tipos en `lib/types.ts` reflejan exactamente lo que devuelve el backend:

```typescript
// NegocioConfig — SIN colorPrimario/Fondo/Texto/layoutTemplate (fijos en CSS)
// ProductoPublico — SIN marcaNombre/imagenPrincipal/imagenes:string[]
// ProductoPublico — CON sku, ordenCatalogo, imagenes: ImagenProducto[]
interface ImagenProducto { url: string; alt: string|null; esPrincipal: boolean; orden: number; }
```

Patrón THEME: colores del template viven en `templates/catalogo-grilla/theme.ts` como constante `THEME = { primary: '#366758', bg: '#fbf9f5', text: '#1b1c1a' }`. Todos los componentes importan THEME en vez de recibir colores como prop. Extensible: cada plantilla futura tendrá su propio `theme.ts`.

`app/[slug]/page.tsx` ya no tiene `switch(layoutTemplate)` — siempre renderiza `CatalogoGrilla`. La selección de plantilla se hará en una fase futura cuando haya más de una.

## Panel Angular "Mi Página" — F6 ✅ (rediseño premium 2026-06-16)

Ruta en panel: `/dashboard/mi-pagina`. Vista split-pane full-height (sin scroll de página).

**Diseño actual (post-premium-refactor)**:
- **Izquierda** (`form-col`, 38% del ancho): tab nav custom + formulario reactivo con 4 secciones.
- **Derecha** (`preview-col`, `flex:1`): browser mockup que llena toda la altura disponible.

**Tabs actuales** (custom nav — sin MatTabsModule, con `@switch (tabActiva())`):
| Key | Ícono | Contenido |
|---|---|---|
| `identidad` | `storefront` | Nombre público, slug, logo, favicon, toggle publicar |
| `contacto` | `contact_phone` | WhatsApp, email, dirección, horario, redes sociales, desc. footer |
| `seo` | `search` | Meta título, meta descripción (contador 0/160) |
| `plantilla` | `palette` | Selección visual: Grilla vs Esencias (cards con mini-preview CSS-only) |

**Layout CSS — cadena full-height**:
```css
/* panel.component.css */
.view-area.view-mi-pagina { overflow: hidden; display: flex; flex-direction: column; padding: 0; }

/* mi-pagina.component.css */
:host { flex:1; display:flex; flex-direction:column; min-height:0; overflow:hidden; }
.shell { flex:1; display:flex; flex-direction:column; min-height:0; }
.body  { flex:1; min-height:0; display:flex; overflow:hidden; }
.form-col   { min-width:340px; width:38%; max-width:440px; flex-shrink:0; display:flex; flex-direction:column; }
.preview-col { flex:1; min-height:0; overflow:hidden; display:flex; flex-direction:column; }
.browser-wrap { flex:1; min-height:0; display:flex; flex-direction:column; }  /* ← clave */
.browser-frame { flex:1; display:flex; flex-direction:column; overflow:hidden; }
.preview-products-placeholder { flex:1; } /* ← crece entre navbar y footer */
```

**Gotcha resuelto — espacio vacío en preview**: La causa era `preview-col-inner` con `max-width: 480px; margin: 0 auto` dentro de un `flex:1` de ~930px. Dejaba ~225px en blanco a cada lado. Fix: eliminar el max-width, convertir a full flex column. Ver también [[venta-inventario-frontend-diseno]].

**Browser mockup**: pestañas del navegador (`browser-tabs`) + viewport (`browser-frame`). El viewport muestra navbar del catálogo → placeholder productos → footer con 3 columnas. Todo reactivo via signal `previewData` (actualizado en `form.valueChanges`). Spotlight amarillo con `box-shadow: 0 0 0 2px #facc15` al pulsar el ícono de ojo de cada campo.

**Selector de plantilla**: dos cards CSS-only que renderizan una miniatura en ~84px de alto sin imágenes — solo divs con colores hardcodeados de cada plantilla.

Backend nuevos en `NegocioControlador`:
```
GET  /negocio/pagina-config  @Permisos('CATALOGO_MI_PAGINA_VER')
PATCH /negocio/pagina-config @Permisos('CATALOGO_MI_PAGINA_EDITAR')
```

`obtenerPaginaConfig(negocioId)`: si no existe el registro → crea uno vacío con `activo=false`. Evita 404 en primer uso.

Migración 015: permisos `CATALOGO_MI_PAGINA_VER` + `CATALOGO_MI_PAGINA_EDITAR` → asignados a SUPERADMIN.

Migración 016: permiso `CATALOGO_MI_PAGINA_SLUG_EDITAR` — solo SUPERADMIN. El slug es sensible: cambiarlo rompe URLs publicadas y SEO. Sin este permiso el campo se muestra bloqueado (readonly con ícono `pi-lock` + badge "Requiere permiso especial").

### UX de spotlight — "ojito" en cada campo

Cada label del formulario tiene un botón ojo (`pi pi-eye`) que al pulsarlo llama `destacar(seccionId)`. El método activa `seccionActiva` signal durante 2.5s, luego se resetea.

Comportamiento visual en la preview:

- El elemento específico del campo recibe `box-shadow` amarillo glow: `0 0 0 2px #facc15, 0 0 12px rgba(250,204,21,0.5)`
- El resto de secciones se oscurece con `opacity-20`
- Usa `[style.boxShadow]` (no clases Tailwind) porque Tailwind no soporta valores CSS custom en ring/shadow

IDs granulares usados (campo → ID → elemento en preview):

| Campo | ID | Elemento |
|---|---|---|
| Nombre público | `navbar-nombre` | `<span>` navbar + `<span>` footer col 1 |
| URL logo | `navbar-logo` | `<img>` navbar |
| Slug | — (sin ojito) | — |
| URL favicon | `tab-favicon` | `<img>` o placeholder div en tab bar |
| Título principal | `hero-titulo` | `<h2>` dentro del hero |
| Subtítulo | `hero-subtitulo` | `<p>` dentro del hero |
| URL imagen hero | `hero-imagen` | div hero entero (es el background) |
| Texto botón CTA | `hero-cta` | `<button>` CTA |
| URL botón CTA | `hero-cta` | `<button>` CTA |
| Contacto (4 campos) | `footer-contacto` | columna footer contacto |
| Redes (3 campos) | `footer-siguenos` | columna footer síguenos |
| Desc. footer | `footer-marca-desc` | columna footer marca |
| Newsletter | `footer-newsletter` | strip newsletter |
| Título SEO | — (sin ojito) | — |
| Descripción SEO | — (sin ojito) | — |

El botón CTA aparece temporalmente aunque esté vacío cuando se activa `hero-cta` (muestra placeholder "Texto del botón CTA"), así el usuario ve dónde quedaría.

El nombre público aparece en dos lugares (navbar + footer col 1). Ambos reciben el glow simultáneamente cuando `seccionActiva() === 'navbar-nombre'`. La columna footer no se oscurece en ese caso.

## Permisos — Plan 2 (cliente intermedio)

El catálogo es una funcionalidad exclusiva del **Plan 2**. Todos los features nuevos se protegen con permisos. Sección `CATALOGO`.

| Permiso | Descripción |
|---|---|
| `CATALOGO_VER` | Ver sección "Mi Página" en el panel |
| `CATALOGO_CONFIG_EDITAR` | Editar identidad visual (logo, nombre, hero) |
| `CATALOGO_FOOTER_EDITAR` | Editar WhatsApp, email, dirección, redes |
| `CATALOGO_PLANTILLA_ELEGIR` | Cambiar la plantilla del catálogo (reservado — plantilla fija por ahora) |
| `CATALOGO_PRODUCTO_PRECIO_EDITAR` | Editar precio y precio_oferta |
| `CATALOGO_PRODUCTO_VISIBILIDAD` | Marcar productos visibles en catálogo |
| `CATALOGO_PRODUCTO_IMAGEN_VER` | Ver galería de imágenes |
| `CATALOGO_PRODUCTO_IMAGEN_SUBIR` | Subir imágenes a productos |
| `CATALOGO_PRODUCTO_IMAGEN_ELIMINAR` | Eliminar imágenes de productos |
| `CATALOGO_MI_PAGINA_VER` | Ver pantalla "Mi Página" (migración 015) |
| `CATALOGO_MI_PAGINA_EDITAR` | Editar configuración completa de la página (migración 015) |
| `CATALOGO_MI_PAGINA_SLUG_EDITAR` | Editar slug — solo SUPERADMIN; campo bloqueado sin este permiso (migración 016) |

Fase **F4.5**: migración SQL que inserta estos permisos y los asigna al rol Plan 2.

## Fases del proyecto

| Fase | Tarea | Estado |
|---|---|---|
| F1 | Crear repo + estructura Next.js 14 | ✅ 2026-05-17 |
| F2 | Plantilla `catalogo-grilla` con mock data | ✅ 2026-05-17 |
| F3 | Migración SQL 014 + entidades NestJS | ✅ 2026-05-17 |
| F4 | Módulo `publico/` en NestJS | ✅ 2026-05-17 |
| F4.5 | Migración SQL permisos CATALOGO | ✅ incluido en 014 |
| F5 | Conectar Next.js con endpoints reales | ✅ 2026-05-17 |
| F6 | Panel Angular "Mi Página" (split-pane + live preview) | ✅ 2026-05-17 |
| F7 | Modal "Catálogo" producto (precios + visibilidad + galería imágenes) | ✅ 2026-05-18 |
| F8 | Deploy producción — backend Render + catálogo Vercel | En progreso 2026-05-29 |

## Archivos de referencia

- `docs/catalogo-landing/plan-catalogo-react.md` — plan completo (actualizado con F6, NegocioConfig limpia sin colores)
- `docs/catalogo-landing/plan-catalogo-react.html` — plan navegable (incluye sección F6 split-pane mockup)
- `docs/catalogo-landing/comparativo-config-catalogo.html` — comparativo visual: campos actuales vs propuesta aprobada (colores eliminados)
- `docs/catalogo-landing/mockup-lullaby-soft.html` — mockup HTML de la primera plantilla
- `docs/auditoria-bd/plan-auditoria-schema.html` — plan de auditoría: schema producción vs setup-completo.sql (6 queries SQL + checklist timestamps)
- `docs/auditoria-bd/plan-auditoria-schema.md` — versión MD del plan de auditoría
- `stitch/stitch_catalogo_de_productos/lullaby_soft_cat_logo_premium_suave/` — referencia visual

## Decisiones de diseño

- **Colores fijos por plantilla**: Eliminados `color_primario`, `color_fondo`, `color_texto` de la BD. Los colores son hardcodeados en el CSS de cada plantilla. Cada negocio elige plantilla, no personaliza colores. Garantiza coherencia visual.
- **Sin precio = no aparece**: endpoint `/pub` no retorna productos sin `precio`. El panel admin muestra `visibleCatalogo=true` correctamente, pero si `precio IS NULL` el producto igual queda excluido del catálogo público — filtro explícito `precio: { [Op.not]: null }` en `publico.servicio.ts:47`. Confusión frecuente: producto "Visible" en panel que no aparece en catálogo → revisar columna Precio primero.
- **PostgreSQL NUMERIC → string en JSON**: columnas `NUMERIC(10,2)` (precio, precioOferta) llegan al frontend Next.js como `string`, no `number`. Usar `Number(valor).toFixed(2)` antes de formatear. Bug inicial: `producto.precioOferta.toFixed is not a function` en `ProductCard.tsx:117` — fixed 2026-05-18.
- **Cambio de negocio no recargaba "Mi Página"** (bug sesión 5): dos bugs coordinados:
  - **Frontend**: `MiPaginaComponent.ngOnInit()` llamaba `cargar()` una vez — no suscrito al signal de contexto. Fix: `toObservable(negocioCtx.negocioActivo).pipe(filter(n => n !== null), switchMap(() => servicio.obtener()))` — recarga automática al cambiar negocio.
  - **Backend**: `negocio.controlador.ts` usaba `req.user.negocioId` en `obtenerMiNegocio`, `actualizarMiNegocio`, `obtenerPaginaConfig`, `actualizarPaginaConfig` — bypasseaba el `TenantInterceptor`. Fix: reemplazar por `@NegocioActual()` que lee `request.negocioId` (ya procesado por el interceptor).
- **Regla derivada**: Cualquier controller que deba respetar el contexto de negocio DEBE usar `@NegocioActual()`, nunca `req.user.negocioId`. El `TenantInterceptor` setea `request.negocioId` desde header `X-Negocio-Context` para SUPERADMIN — si el controller ignora esto, el contexto no funciona.
- **Etiquetas como categorías en v1**: sidebar de filtros usa etiquetas, sin tabla `categoria_producto`.
- **WhatsApp FAB condicional**: solo si `config.whatsappNumero != null`.
- **Imagen placeholder SVG**: genérico con el color de la plantilla (no del negocio).
- **PaginaNoDisponible (reemplazó mock fallback)**: `app/[slug]/page.tsx` ya no usa MOCK_CONFIG/MOCK_PRODUCTOS. Si la API falla o el catálogo está inactivo (`activo=false` → backend retorna 404), se muestra `<PaginaNoDisponible />` con ícono de candado. El mock solo servía para desarrollar; en producción es confuso.
- **Dark mode desactivado**: se removió el bloque `@media (prefers-color-scheme: dark)` del globals.css para que el branding prevalezca siempre.
- **`CatalogoGrilla` es `'use client'`**: maneja estado de filtro con `useState` + `useMemo`. Subcomponentes sin estado son server components.
- **F6 split-pane implementado**: panel Angular "Mi Página" con 4 tabs (Identidad, Hero, Contacto, SEO) a la izquierda + preview vivo a la derecha. Signal `previewData` actualizada con `form.valueChanges`. Permisos: `CATALOGO_MI_PAGINA_VER` / `CATALOGO_MI_PAGINA_EDITAR` (migración 015).
- **THEME constante por plantilla**: cada template tiene su `theme.ts` con colores hardcodeados. Los componentes importan THEME, no reciben colores como prop del config del negocio.
- **Favicon en Next.js App Router**: `generateMetadata({ icons: { icon: faviconUrl } })` NO funciona si existe `app/favicon.ico` físico — el archivo toma prioridad absoluta. Solución: eliminar `app/favicon.ico`. El favicon dinámico pasa a funcionar desde `generateMetadata`.
- **Slug sensible → permiso especial**: cambiar el slug rompe URLs publicadas y posicionamiento SEO. El campo se protege con `CATALOGO_MI_PAGINA_SLUG_EDITAR` (migración 016), asignado solo a SUPERADMIN. Sin el permiso, el campo aparece bloqueado (readonly) con badge de aviso.
- **Toggle "Publicar página web"**: renombrado desde "Catálogo activo" para que sea más claro para el usuario final.
- **`w-fit` para glow en elementos inline**: usar `w-fit` (no `inline-block`) en `<h2>`/`<p>` dentro del hero para que el box-shadow abrace solo el texto pero los elementos sigan en su propio bloque (no fluyan inline).
- **PrimeNG Tooltip**: siempre usar `pTooltip="texto" tooltipPosition="top"` + importar `TooltipModule`. Nunca `title=""` — produce tooltip nativo del browser sin estilos.
- **Reset de estilos en botones Angular**: botones `<button type="button">` tienen estilos browser por defecto (borde, fondo gris). Reset obligatorio: `bg-transparent border-0 p-0 focus:outline-none` vía Tailwind.
- **Dimming con prefijo**: cuando el spotlight es granular (ej. `hero-titulo`), el `opacity-20` del padre debe usar `!seccionActiva()?.startsWith('hero')` en lugar de `seccionActiva() !== 'hero'`.
- **Upsert en pagina-config**: `obtenerPaginaConfig` crea registro vacío (`activo=false`) si no existe — evita 404 en el primer uso del panel.
- **StorageServicio: Cloudinary por defecto en producción (2026-05-24)**: Bug crítico — imágenes subidas en Railway se guardaban con URL `http://localhost:13200/uploads/...` porque `StorageServicio` usaba `process.env.STORAGE_PROVIDER ?? 'local'` como proveedor. Si la variable no se leía en el primer deploy, el fallback era `'local'`. Fix: invertir la lógica a `process.env.STORAGE_PROVIDER === 'local' ? 'local' : 'cloudinary'`. Cloudinary es el default; local solo si se pide explícitamente. Evita depender de `NODE_ENV`. **Regla derivada**: en producción, nunca confiar en que una variable de entorno esté seteada — asumir que el default es el comportamiento seguro/productivo. **Regla local**: agregar `STORAGE_PROVIDER=local` en `.env` para desarrollo, así el disco local sigue funcionando. Credenciales en uso: `CLOUDINARY_CLOUD_NAME=dovxgcpc5`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` — configuradas en Railway. Error 500 intermedio fue causado por credenciales incorrectas en Railway (copy-paste erróneo). Verificar siempre en la consola de Cloudinary que `cloud_name` coincida con el mostrado en la URL del dashboard.
- **StorageServicio separado por negocio (2026-05-19)**: `subirImagen(file, negocioId)` — firma cambiada. Cloudinary usa `folder: 'inventario-ventas/{negocioId}/productos'` (Cloudinary crea las carpetas automáticamente al primer upload). Local usa `uploads/negocio-{id}/productos/`. Controller `multerStorage` lee `req['negocioId']` (seteado por TenantInterceptor) para crear la carpeta física local. Si se agrega otro controller que use StorageServicio, DEBE pasar `negocioId`. Imágenes ya subidas quedan en la ruta antigua — solo afecta nuevos uploads.
- **F7 — Modal "Catálogo" en lugar de dialog de galería simple**: La decisión de diseño fue crear un modal unificado `CatalogoProductoDialog` que consolida precio + precioOferta + visibleCatalogo + ordenCatalogo + galería de imágenes. Esos campos se **movieron** del dialog de edición al nuevo modal. El edit dialog queda solo con datos básicos. Botón `pi-globe` en la tabla abre el modal.
- **Storage local/Cloudinary con abstracción**: `StorageServicio` (`backend/src/storage/`) lee `STORAGE_PROVIDER=local|cloudinary`. Local: multer diskStorage → `uploads/productos/` (git-ignored), sirve vía `ServeStaticModule` en `serveRoot: '/uploads'`. Cloudinary: `cloudinary.uploader.upload(file.path)` → `secure_url`. URL almacenada es siempre absoluta (`BACKEND_URL` + path relativo para local).
- **Multer gotcha — `@BelongsTo` no es `CreationOptional`**: en la entidad `ProductoImagen`, el campo `declare producto: Producto` de la relación no tiene `CreationOptional<>`, lo que hace que `imagenModelo.create({...})` falle en TypeScript. Fix: `as any` en el create. No modificar la entidad — es un límite de Sequelize-TypeScript.
- **`FormData` upload a través de `HttpBaseService`**: `HttpBaseService.post()` acepta `unknown` como payload. Al pasar `FormData`, Angular's `HttpClient` setea automáticamente `Content-Type: multipart/form-data; boundary=...`. No se debe setear el header manualmente (lo rompería). `esPrincipal` se pasa como string `'true'`/`'false'` en el FormData — el DTO lo transforma con `@Transform`.
- **Bug `esPrincipal` FormData boolean — `enableImplicitConversion: true` convierte `'false'` a `true`**: La ValidationPipe de NestJS tiene `enableImplicitConversion: true` en `transformOptions`. Esto hace que el string `'false'` (enviado por FormData) sea convertido a `Boolean('false')` = `true` (string no vacío → truthy) ANTES de que corra el `@Transform`. Resultado: checkbox desmarcado → imagen guardada como principal. **Fix**: en el servicio Angular (`producto-imagenes.servicio.ts`), solo enviar el campo cuando es `true`: `if (esPrincipal === true) fd.append('esPrincipal', 'true')`. Si no se envía, el backend recibe `undefined` y `dto.esPrincipal ?? false` = `false`. **Regla general**: para campos boolean en FormData, nunca enviar el valor negativo — omitirlo y confiar en el `?? false` del backend.
- **Parpadeo al marcar imagen principal — actualización optimista (2026-05-23)**: `marcarComoPrincipal()` llamaba `cargarImagenes()` en el `next`. Eso ejecutaba: `cargandoImagenes.set(true)` → HTTP GET → `imagenes.set(nuevaArray)` — tres operaciones que causaban re-render completo, flash visual y reset del scroll de la galería. Fix: reemplazar `this.cargarImagenes()` por `this.imagenes.update(imgs => imgs.map(img => ({ ...img, esPrincipal: img.id === imagen.id })))`. La operación es determinista (una principal, el resto no), no necesita round-trip. Angular reutiliza los nodos DOM del `@for (track img.id)` y actualiza solo los bindings que cambiaron. Regla: si el resultado de una mutación es predecible, actualizar el signal localmente en lugar de recargar del servidor.
- **`StorageServicio.eliminarImagen(url)` — borrado real en Cloudinary/local (2026-05-23)**: el backend solo hacía soft delete en BD (`eliminado = true`) pero la imagen quedaba en Cloudinary consumiendo espacio. Fix: nuevo método `eliminarImagen(url: string)` en `StorageServicio`. Para Cloudinary: extrae `public_id` con regex `url.match(/\/upload\/(?:v\d+\/)?(.+)\.[^.]+$/)` y llama `cloudinary.uploader.destroy(publicId)`. Para local: extrae ruta desde la URL y llama `fs.unlinkSync()`. El método está en try/catch silencioso — si Cloudinary falla el soft delete en BD igual se completa. Llamado desde `productos.servicio.ts::eliminarImagen()` justo antes del `imagen.update({ eliminado: true })`. En local se puede probar: subir imagen → ver archivo en `backend/uploads/` → eliminar desde el modal → verificar que desaparece del disco.
- **`@nestjs/serve-static` no disponible en Docker**: el backend corre en Docker (`/app/`). Instalar un paquete con `npm install` en el host (`backend/`) no lo pone disponible en el contenedor. Fix: `docker compose -f docker-compose.dev.yml exec backend npm install @nestjs/serve-static`. Regla general: cualquier paquete nuevo en backend debe instalarse dentro del contenedor o hacer rebuild de la imagen.

## F7 — Modal "Catálogo" de producto ✅

Botón `pi-globe` en la tabla de productos (desktop + mobile) abre `CatalogoProductoDialog` — modal Angular con dos secciones verticales:

**Sección A — Precios y visibilidad** (movida desde el dialog de edición)
- `precio`, `precioOferta` — gated `CATALOGO_PRODUCTO_PRECIO_EDITAR`
- `visibleCatalogo` (toggle), `ordenCatalogo` — gated `CATALOGO_PRODUCTO_VISIBILIDAD`
- Guarda con `PATCH /inventario/productos/:id` (endpoint existente, solo envía los 4 campos)

**Sección B — Galería de imágenes** — gated `CATALOGO_PRODUCTO_IMAGEN_VER`
- Lista thumbnails 80×80 con badge "Principal" y botones estrella/trash
- Upload: `<input file>` + alt + checkbox principal → `POST /inventario/productos/:id/imagenes`
- Marcar principal actualiza otras imágenes (backend desmarca las demás)

**Archivos creados en esta fase:**

| Archivo | Descripción |
|---|---|
| `backend/src/storage/storage.servicio.ts` | Abstracción local/Cloudinary |
| `backend/src/storage/storage.modulo.ts` | Módulo NestJS del storage |
| `backend/src/inventario/productos/dto/agregar-imagen.dto.ts` | DTO subida de imagen |
| `backend/src/inventario/productos/dto/actualizar-imagen.dto.ts` | DTO actualización imagen |
| `frontend/src/app/core/services/producto-imagenes.servicio.ts` | Servicio Angular CRUD imágenes |
| `frontend/src/app/features/inventario/productos-lista/catalogo-producto-dialog/` | Componente modal catálogo |

**Variables de entorno nuevas:**
```
STORAGE_PROVIDER=local       # o cloudinary
BACKEND_URL=http://localhost:3200
CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=
```

**Endpoints nuevos:**
```
GET    /inventario/productos/:id/imagenes            CATALOGO_PRODUCTO_IMAGEN_VER
POST   /inventario/productos/:id/imagenes            CATALOGO_PRODUCTO_IMAGEN_SUBIR  (multipart)
PATCH  /inventario/productos/:id/imagenes/:imagenId  CATALOGO_PRODUCTO_IMAGEN_SUBIR
DELETE /inventario/productos/:id/imagenes/:imagenId  CATALOGO_PRODUCTO_IMAGEN_ELIMINAR
```

## Plantilla `catalogo-esencias` ✅ 2026-05-24

Estética de perfumería / colonias de lujo. Referencia visual: imagen de e-commerce orgánico con paleta cálida, sidebar de filtros y tipografía serif.

### Paleta y tipografía

| Token | Valor | Uso |
|---|---|---|
| `bg` | `#f7f3ee` | Fondo general (ivory cálido) |
| `surface` | `#f0ebe2` | Sidebar, placeholder imagen |
| `border` | `#ddd5c6` | Bordes suaves |
| `text` | `#1a1209` | Texto principal (espresso) |
| `textMuted` | `#7a6a58` | Texto secundario, labels |
| `primary` | `#b8975c` | Botones, accents (dorado) |
| `primaryDark` | `#8a6a40` | Hover del primario |

Tipografía: `Playfair Display` como `--font-display` (headings) + `Nunito Sans` (cuerpo). Playfair cargada en `app/layout.tsx` junto a las existentes.

### Layout diferencial vs grilla

| Aspecto | `catalogo-grilla` | `catalogo-esencias` |
|---|---|---|
| Filtros | Chips en navbar, scroll horizontal | Sidebar izquierda (lg+) + pills mobile |
| Imagen producto | Aspect 4:3 (horizontal) | Aspect 3:4 (vertical, más elegante) |
| Placeholder | SVG geométrico color primario | SVG frasco abstracto en `surface` |
| Card borders | `rounded-2xl shadow-sm` | Sin borde ni sombra — gap del grid separa |
| Precio oferta | Badge fluorescente "Oferta" | Texto tachado discreto |
| Hero | Solo desktop (`hidden md:block`) | Visible desktop + mobile (altura responsiva) |
| Hero overlay | No tiene | Caja ivory `bg-[bg]/90 backdrop-blur` esquina inferior izquierda |

### Componentes plantilla `catalogo-esencias`

| Componente | Tipo | Responsabilidad |
|---|---|---|
| `index.tsx` | client | Orquestador, estado filtro, layout sidebar+grid |
| `Navbar.tsx` | client | Logo serif, links de categoría underline activo, link WhatsApp |
| `HeroBanner.tsx` | server | Hero full-width, caja overlay ivory con badge + título + CTA |
| `FilterSidebar.tsx` | client | Sidebar desktop 224px sticky + pills mobile; props: etiquetas, categoriaActiva, onCategoriaChange |
| `ProductGrid.tsx` | server | Grid 2/3 cols, skeleton con aspect 3:4, mensaje vacío |
| `ProductCard.tsx` | server | Imagen 3:4, etiqueta en color, nombre Playfair, precio discreto, btn dorado |
| `Footer.tsx` | client | 4 columnas (marca + tienda + contacto + newsletter), copyright |
| `WhatsAppFab.tsx` | client | FAB dorado (primary) en lugar del verde de grilla |
| `theme.ts` | — | Constante THEME con 7 tokens de color |

### Mock data de esencias

En `lib/mock-data.ts`: `MOCK_CONFIG_ESENCIAS` (slug `esencias-demo`, `plantilla: 'esencias'`), `MOCK_CATEGORIAS_ESENCIAS` (Cítrico, Floral, Amaderado, Cálido), `MOCK_PRODUCTOS_ESENCIAS` (6 fragancias con nombres reales: Aura Essentielle, Santalum Nº3, Oud & Fig, etc.).

## Componentes plantilla `catalogo-grilla`

| Componente | Tipo | Responsabilidad |
|---|---|---|
| `index.tsx` | client | Orquestador, estado filtro categoría |
| `Navbar.tsx` | client | Logo, chips de categoría, sticky |
| `HeroBanner.tsx` | server | Fondo color primario, decoración SVG geométrica |
| `ProductGrid.tsx` | server | Grid 1/2/3 cols, estado vacío, skeleton |
| `ProductCard.tsx` | server | Imagen/placeholder, precio+oferta, btn WhatsApp |
| `Footer.tsx` | client | Contacto, redes, newsletter form |
| `WhatsAppFab.tsx` | client | Botón flotante, solo si `whatsappNumero != null` |

- **`plantilla` en NegocioConfig (2026-05-24)**: campo `plantilla: string | null` agregado a la interfaz TypeScript y al `ConfigPublicaDto`. El router en `page.tsx` hace switch por valor string. Default: `CatalogoGrilla` si el campo es `null` o desconocido. Extensible: agregar nueva plantilla = crear carpeta + importar + agregar `else if`. Pendiente: campo `plantilla` en la tabla SQL `negocio_pagina_config` (aún no tiene migración — el campo existe en TypeScript pero no en BD).
- **`catalogo-esencias`: sidebar izquierda en lugar de chips de navbar**: `FilterSidebar` es un componente separado que en desktop (lg+) renderiza el sidebar y en mobile renderiza pills horizontales. El `index.tsx` lo ubica al lado del `ProductGrid` con `flex-row gap-12`. Mantiene la misma interfaz de props que la lógica de filtrado del grilla (`categoriaActiva`, `onCategoriaChange`).
- **Playfair Display como `--font-display`**: nombre de variable genérico (no `--font-playfair`) para que otras plantillas futuras puedan usar otro serif sin cambiar los componentes que consumen `var(--font-display)`.
- **WhatsAppFab dorado en esencias**: para coherencia con la paleta, el FAB usa `THEME.primary` (#b8975c) en lugar del verde WhatsApp (#25d366). La identidad de marca prima sobre la identidad de la app de mensajería.

## F8 — Deploy producción (en progreso 2026-05-29)

### Infraestructura elegida

| Servicio | Plataforma | URL |
|---|---|---|
| Backend NestJS | Render (Web Service, Docker) | `inventario-multisede-backend.onrender.com` |
| Catálogo Next.js | Vercel | `catalogo.adifnex.site` |
| Base de datos | PostgreSQL en Render | (privado) |

**Railway → Render**: Railway trial se agotó (~$4.54 créditos). Render free tier elegido como reemplazo. Mismo Dockerfile de producción — multi-stage, `npm ci --omit=dev`, `CMD ["node", "dist/main.js"]`.

### Configuración Render

- **Language**: Docker (NO Node — `.npmrc` con `legacy-peer-deps=true` se copia automático en Docker; con Node mode el comportamiento es impredecible)
- **Branch**: `main`
- **Region**: Oregon (US West) — ya tenía 3 servicios ahí
- **PORT**: NO agregar manualmente — Render lo inyecta, y `main.ts` ya lee `process.env.PORT || 3000`

**Variables de entorno en Render (backend):**

```env
DB_HOST=<render postgres host>
DB_NAME=postgres
DB_PASS=<password>
DB_PORT=5432
DB_SCHEMA=inventario-ventas
DB_SSL=true
DB_USER=<usuario>
JWT_SECRETO=<mínimo 64 chars, generar nuevo>
JWT_EXPIRACION=15m
JWT_REFRESH_EXPIRACION=30d
BACKEND_URL=https://inventario-multisede-backend.onrender.com
STORAGE_PROVIDER=cloudinary
CLOUDINARY_CLOUD_NAME=dovxgcpc5
CLOUDINARY_API_KEY=<de consola Cloudinary>
CLOUDINARY_API_SECRET=<de consola Cloudinary>
```

### Gotcha Vercel — NEXT_PUBLIC_* requiere redeploy

`NEXT_PUBLIC_API_URL` se inyecta en **build time**, no en runtime. Cambiar el valor en Vercel → Environment Variables no tiene efecto hasta el próximo deploy.

**Proceso correcto:** cambiar variable → Deployments → Redeploy (sin cache existente).

**Valor correcto:**

```env
NEXT_PUBLIC_API_URL=https://inventario-multisede-backend.onrender.com/api
```

Con `/api` al final, sin trailing slash.

### Gotcha Render free tier — cold start

Free tier duerme después de inactividad. Primera request tarda 30–50 segundos en despertar. Si Vercel llama al backend en cold start, el fetch puede timeout → `PaginaNoDisponible`. Solución para producción real: upgrade a Starter ($7/mes) o servicio externo de health check cada 10 min.

### Diagnóstico `PaginaNoDisponible`

El componente `PaginaNoDisponible` aparece ante CUALQUIER error del try/catch en `app/[slug]/page.tsx:45` — no solo `activo=false`. Checklist de diagnóstico:

1. ¿Backend responde? → abrir `https://<backend>/api/pub/<slug>` directo en browser
2. ¿Variable correcta en Vercel? → re-ingresar `NEXT_PUBLIC_API_URL` (es Sensitive, no se puede leer) → redeploy
3. ¿Cold start? → despertar el backend manualmente → recargar catálogo inmediatamente

## Refactor 2026-06-16 — Hero + Newsletter eliminados completamente

**Decisión**: Hero y Newsletter no se usarán en el catálogo en esta primera versión. Se eliminaron de toda la plataforma (DB, backend, frontend Angular, catálogo React).

### Qué se eliminó

| Capa | Qué se borró |
|---|---|
| **DB** | Tabla `pagina_hero_imagen` completa · columnas `hero_titulo`, `hero_subtitulo`, `hero_cta_texto`, `hero_cta_url` (de `negocio_pagina_config`) · columna `footer_newsletter` · columna `max_imagenes_hero` (de `negocio`) · 2 índices hero |
| **Backend** | Entidad `pagina-hero-imagen.entidad.ts` eliminada · 4 endpoints hero (POST/DELETE/PATCH orden/PATCH principal) del controlador · métodos hero en el servicio · `PaginaHeroImagen` removido del módulo · campos hero/newsletter de DTOs · `ImagenHeroPublicaDto` de config-publica.dto.ts |
| **Frontend Angular** | Tab "Hero" completo del componente `mi-pagina` · toggle Newsletter · preview hero y preview newsletter · signal `imagenesHero` + `maxImagenesHero` · métodos `onHeroImagenSeleccionada`, `eliminarImagenHero`, `onDragDrop`, `marcarPrincipal` · interfaz `ImagenHero` del servicio · métodos de imagen hero del servicio · tarjeta "Max. imágenes hero" en `mi-negocio-vista` |
| **Catálogo React** | Archivos `HeroBanner.tsx` de ambas plantillas · interfaz `ImagenHero` de `types.ts` · campos `heroTitulo/Subtitulo/CtaTexto/CtaUrl/imagenesHero/footerNewsletter` de `NegocioConfig` · mock data limpio |

### Migración

`database/migrations/009-eliminar-hero-newsletter.sql` — idempotente, con `DROP TABLE IF EXISTS` y `DROP COLUMN IF EXISTS`. Orden: 1) eliminar `pagina_hero_imagen` (tiene FK), 2) columnas hero de `negocio_pagina_config`, 3) `max_imagenes_hero` de `negocio`.

### Reglas derivadas

- **SEO metadata fallback**: `app/[slug]/page.tsx` usaba `config.heroSubtitulo` como fallback para `description`. Cambiado a `config.nombrePublico`.
- **Plantillas sin hero**: ambas plantillas (`catalogo-grilla` y `catalogo-esencias`) arrancan directamente en la Navbar → Productos. No hay sección de bienvenida grande.
- **`NegocioConfig` más limpia**: la interfaz ahora refleja exactamente lo que existe en BD. Sin campos muertos.

### Panel "Mi Página" — tabs restantes

Ahora solo quedan 4 tabs: **Identidad** · **Contacto** · **SEO** · **Plantilla**.

---

## Refactor 2026-06-30 — Unificación de plantillas + modelo de "Temas"

**Contexto**: las plantillas habían crecido a **7 tipos** (`grilla`, `esencias`, `bebe`, `dama`, `mujer`, `accesorios`, `standard`), cada una en su carpeta `templates/catalogo-*` con su propio `index.tsx`, `Navbar`, `FilterSidebar`, `ProductGrid`, `ProductCard`, `Footer`, `WhatsAppFab`, `theme.ts`, `temaContext.tsx`. La estructura/layout era **idéntica** entre ellas; lo único que cambiaba era **color, tipografía y formato del card**. → ~95% de código duplicado + un `if/else` gigante en `app/[slug]/page.tsx`. Tocar algo obligaba a editar 7 carpetas.

**Decisión (con Christian)**: un **solo catálogo** manejado por un **registro de presets**. Modelo conceptual **"Tema curado + color libre"**:
- Un **tema** empaqueta **fuente + formato de card FIJOS** (su personalidad). NO se combinan libremente.
- El negocio solo elige **color** (4 paletas por tema).
- Se descartó el desacople total (color × fuente × card sueltos): generaría combinaciones incoherentes y explosión de QA (4×4×4=64 combos vs 7 temas × 4 colores). 
- **String `config.plantilla` sin cambios** (`catalogo-X[-color]`) → **cero cambios en backend/BD**. Retrocompatible.

### Arquitectura nueva en `catalogo-react`

| Pieza | Archivo | Rol |
|---|---|---|
| Tipos | `lib/plantillas/tipos.ts` | `Tema` (11 colores), `ConfigCard`, `Fuentes`, `PlantillaResuelta`, `Preset` |
| Registro | `lib/plantillas/registro.ts` | `PRESETS` por prefijo + `resolverPlantilla(plantilla) → {tema, fuentes, card}`. Paletas copiadas exactas de los viejos `theme.ts`. Resolver genérico: matchea prefijo (más largo primero) y resuelve sufijo de color; fallback = grilla |
| Contexto | `components/catalogo/PlantillaContext.tsx` | `PlantillaProvider` + hooks `usePlantilla/useTema/useFuentes/useCard` (reemplaza los 7 `temaContext`) |
| Shell | `components/catalogo/Catalogo.tsx` | Único; resuelve plantilla, envuelve en provider, renderiza banda bienvenida + sidebar + grid + footer + fab |
| Componentes | `components/catalogo/{Navbar,FilterSidebar,ProductGrid,Footer,WhatsAppFab}.tsx` | Únicos (basados en standard, usan `fuentes.titulo` para headings) |
| Card | `components/catalogo/ProductCard.tsx` | **Una sola card parametrizada** por `ConfigCard` |
| Detalle | `lib/tema-catalogo.ts` | `resolverTemaDetalle()` reescrito para reusar `resolverPlantilla` (antes reimportaba 5 `theme.ts`) |

`app/[slug]/page.tsx` quedó en **un solo `<Catalogo>`** sin imports ni `if/else`. Las **7 carpetas `templates/` se eliminaron**.

### 4 formatos de card (enum `FormatoCard`)

| Formato | Aspecto | Quién | Rasgos |
|---|---|---|---|
| `estandar` | 4:3 | grilla, standard, dama, mujer | borde, swatches, badge oferta, WhatsApp verde |
| `redondeado` | 4:3 | bebe | `rounded-3xl` + sombra, WhatsApp verde |
| `alto` | 3:4 | esencias | sin borde, sin swatches, WhatsApp color primary, oferta texto |
| `cuadrado` | 1:1 | accesorios | badge índice 01/02, título UPPERCASE, swatches inline, **sin WhatsApp**, oferta texto |

Flags de `ConfigCard`: `swatches`, `indice`, `whatsapp: 'verde'|'primary'|'oculto'`, `ofertaEstilo: 'badge'|'texto'`, `tituloUppercase`, `tituloFuente`.

### Fuentes por tema (CSS vars en `app/layout.tsx`)

| Tema | Título | Cuerpo |
|---|---|---|
| grilla, bebe, standard | Quicksand | Nunito Sans |
| esencias | Playfair Display (serif) | Nunito Sans |
| dama, mujer | Cormorant Garamond (serif) | Nunito Sans |
| accesorios | Jost (gallery) | Jost |

### Colores — Grilla y Esencias ahora con 4 variantes (antes 1)

- **Grilla**: Esmeralda (default `catalogo-grilla`), Azul, Grafito, Vino.
- **Esencias**: Sepia (default `catalogo-esencias`), Carbón, Rosa, Oliva.
- Las paletas faltantes de Esencias/Grilla (eran tema único) se completaron a las 11 claves de `Tema`.

### Admin Angular "Mi Página" — reencuadre a "Temas"

- Renombrado **"Plantillas"/"Diseño del catálogo" → "Temas"**; tab `Plantilla` → `Tema`; "Color del diseño" → "Color del tema".
- **Unificado el modelo**: las 7 ahora viven en el array `familias`. Se eliminó todo el código especial de Bebé (`variantesBebe`, `varianteBebe*`, `esFamiliaBebe`) y las 3 tarjetas hardcodeadas (Grilla/Esencias/Bebé). Un solo `@for` las renderiza con sus swatches.
- **Grilla y Esencias ahora muestran selector de color** (antes a un solo color).
- `FamiliaPlantilla` extendida con `fuenteTitulo` (CSS) + `formato`.

### Vista previa con productos ficticios

Antes el preview mostraba un placeholder `[ Grid de productos ]`. Ahora muestra **3 productos ficticios** (`productosEjemplo`) con la **fuente real + formato + color** del tema activo, para ver de verdad cómo se ve. Helpers: `temaActivo()`, `previewVariante()`, `aspectoPreview()`. Las **fuentes Google reales** (Quicksand, Nunito Sans, Playfair, Cormorant, Jost) se cargan vía `<link>` en `frontend/src/index.html`. El nombre del navbar del preview también usa la fuente del tema.

**Gotcha — imagen del mock invisible**: la imagen-placeholder usaba `previewVariante().soft`, que es **el mismo color que el fondo del cuerpo** (`previewTema().bodyBg = soft`) → la imagen no se veía (peor en formatos sin fondo de card: Esencias/Accesorios → preview "vacío"). **Fix**: la imagen usa `surface` (color distinto) + `box-shadow: inset 0 0 0 1px` + un **ícono SVG de foto** (marco + montaña) en color `primaryDark`. Regla derivada: en mockups, nunca pintar el placeholder con el mismo color del contenedor; usar color contrastante + ícono representativo.

**Verificación**: `catalogo-react` `tsc --noEmit` OK + runtime de `resolverPlantilla` (todos los prefijos/colores/fallback). `frontend` `ng build` OK.

---

## Links relacionados

- [[venta-inventario]] — hub del proyecto
- [[venta-inventario-inventario]] — marcas, etiquetas, productos (entidades base)
- [[venta-inventario-rbac]] — sistema de permisos existente (donde se añadirán los CATALOGO_*)
- [[modelos-negocio/saas-planes]] — planes de suscripción (Plan 1 básico vs Plan 2 intermedio)

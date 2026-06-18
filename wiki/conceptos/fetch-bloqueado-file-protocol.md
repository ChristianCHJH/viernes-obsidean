---
titulo: fetch bloqueado bajo file:// — patrón script-global
tipo: concepto
tags: [web, javascript, estatico, cors, file-protocol, gotcha]
fecha_creacion: 2026-06-18
fecha_actualizacion: 2026-06-18
estado: activo
---

# `fetch` bloqueado bajo `file://` — patrón script-global

## El problema

En una web 100% estática que se abre con **doble clic** (protocolo `file://`, sin servidor), los
navegadores (Chrome/Edge) **bloquean por CORS** `fetch()`, `XMLHttpRequest` y los **ES modules**
(`import`). Por eso no se pueden cargar archivos `.json` de datos locales con `fetch`: el `index.html`
abierto directamente falla silenciosamente o tira error CORS.

## La solución — datos como `.js` que asignan a un global

Los **`<script src>` clásicos SÍ cargan archivos locales bajo `file://`** (a diferencia de
fetch/XHR/modules). Entonces, en vez de datos `.json` + `fetch`, se sirven como `.js` que asignan a un
objeto global:

```js
// data/clase-01.js
window.LESSONS = window.LESSONS || {};
window.LESSONS["m01-l01"] = { /* …datos… */ };
```

Y el HTML los carga con `<script src="data/clase-01.js"></script>`. Para carga dinámica (no saber de
antemano qué archivo), se inyecta el script en runtime:

```js
const s = document.createElement("script");
s.src = file; s.onload = () => resolve(window.LESSONS[id]); document.head.appendChild(s);
```

Esto **mantiene la separación contenido/lógica** (los datos siguen siendo "datos", solo cambia el
envoltorio) y permite que el proyecto ande sin servidor.

## Cuándo aplica

- Apps/herramientas estáticas que deben abrirse con doble clic, sin `python -m http.server` ni Live
  Server, ni deploy.
- Si en el futuro se sirve con un servidor estático, se puede volver a `.json` + `fetch` sin tocar la
  lógica de render.

## Otras implicancias del modo `file://`

- Nada de ES modules (`type="module"` + `import`): usar IIFE + globals (`window.X`).
- `localStorage` **sí** funciona en `file://` → sirve para persistencia (con Exportar/Importar JSON
  para versionar, porque el navegador no puede escribir al disco/repo).

## Visto en

- [[proyectos/profesor-ingles]] — curso de inglés estático; `window.LESSONS` + `window.ROADMAP`.

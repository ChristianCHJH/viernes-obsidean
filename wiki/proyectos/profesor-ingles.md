---
titulo: Profesor Inglés (teacher-inglish)
tipo: proyecto
tags: [educacion, ingles, estatico, vanilla-js, web-speech-api, autodidacta]
fecha_creacion: 2026-06-18
fecha_actualizacion: 2026-06-18
estado: activo
---

# Profesor Inglés (teacher-inglish)

Curso de inglés **autodidacta, adaptativo y mes a mes** para Christian (alumno único). Funciona como
un **repositorio de clases interactivas**: Claude actúa de profesor, diseña el contenido mes a mes
(A1→C2, marco CEFR), y al final de cada mes evalúa qué reforzar. Punto de partida: nivel 0 +
mini-diagnóstico en la Clase 1.

Ubicación: `c:\Christian\Christian Personal\teacher-inglish`

## Stack (atípico para Christian — a propósito)

- **100% estático**: HTML + CSS + JavaScript **vanilla**. Sin framework, sin build, sin servidor.
  Se abre con **doble clic** (probado Chrome/Edge). Nada de Angular/NestJS aquí.
- **Cero dependencias / cero CDN.** Comentarios en español, variables en inglés.
- **Voz**: Web Speech API — `SpeechSynthesis` (escuchar modelo) + `SpeechRecognition` (pronunciación).
- **Persistencia (Opción C)**: `localStorage` automático + botones Exportar/Importar JSON (el progreso
  se versiona a mano en `progress/` porque el navegador no puede escribir al repo).

## Decisión técnica clave — datos como `.js`, no `.json`

El contenido de cada clase **son datos separados del motor** (principio rector). Idealmente serían
`.json` + `fetch()`, pero al abrir con `file://` los navegadores **bloquean `fetch` por CORS**.
Solución: cada clase es un `.js` que asigna a `window.LESSONS[id] = {...}` y se carga con
`<script src>` (los script tags clásicos sí cargan local bajo `file://`). El roadmap igual:
`window.ROADMAP`. Si algún día se sirve con servidor estático, se puede migrar a `.json` sin tocar
el motor. Ver [[conceptos/fetch-bloqueado-file-protocol]].

## Diseño visual (dirección CERRADA por Christian)

Base blanca dominante, mucho espacio en blanco, **sin colores saturados**. Un único acento desaturado
= **verde salvia**, usado con moderación solo para estados (correcto/incorrecto, foco, progreso).
Accesible: foco visible, navegable por teclado, contraste. Tokens `--spa-*` en `engine/styles.css`.
Construido con la skill `ui-ux-pro-max` (su CLI Python no corría en la PC; se usó la guía + checklist).

## Arquitectura — motor vs contenido

```
teacher-inglish/
├── index.html / lesson.html / dashboard.html
├── engine/   styles.css · storage.js · lesson-engine.js · exercises.js · speech.js · qa-banner.js
├── curriculum/roadmap.js          (window.ROADMAP, mapa A1→C2 + Mes 1)
├── lessons/month-01/              lesson-01.js (con mini-diagnóstico) · lesson-02.js · exam.js
└── progress/                      JSON exportados por el usuario
```

- **Paquete de interacción**: navegación por pasos/slides · corrección inmediata con explicación ·
  banner flotante de Q&A predefinido por clase · pronunciación permisiva→estricta por nivel CEFR.
- **7 tipos de ejercicio**: multiple_choice, fill_blank, matching, word_order, listening,
  pronunciation, short_writing. Cada ejercicio lleva campo `topic` (obligatorio) → alimenta el
  etiquetado de errores y el **repaso espaciado**.
- **Examen** etiqueta cada error por tema; **dashboard** agrega por topic y ordena por debilidad.

## Estado — 5 hitos completos (validados 2026-06-18: "de momento todo bien")

| Hito | Estado |
|------|--------|
| 1. Motor base (índice + render por pasos + localStorage) | ✅ |
| 2. Clase 1 con mini-diagnóstico (12 preguntas) | ✅ |
| 3. Clase 2 normal A1 (todos los tipos, audio, pronunciación, banner) | ✅ |
| 4. Examen Mes 1 con etiquetado de errores por tema | ✅ |
| 5. Dashboard de progreso | ✅ |

4 commits hechos (`feat:`/`content:`). Sintaxis JS verificada con `node --check`.

## Pendiente

- Clases **03–25** del Mes 1 — mismo molde que `lesson-02.js`, se generan en lote cuando Christian
  valide las 2 muestras. En `roadmap.js` están con `available: false`.
- Meses siguientes (A2→C2) y la lógica de planificación mensual adaptativa post-examen.

## Documentos fuente en el repo

`PLAN.md` (el qué/por qué del producto — fuente de verdad), `CLAUDE.md` (cómo construir),
`README.md` (uso, respaldo de progreso, limitación de voz).

## Relacionadas

- [[proyectos/landing-page-christina]] — otro proyecto HTML estático de Christian
- [[personas/christian-jara]]

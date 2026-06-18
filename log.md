# Log de operaciones — Viernes

Registro cronológico de todas las ingestiones, actualizaciones y operaciones del wiki.

---

## [2026-06-17] ingest | Negocios lista — rediseño premium + patrón view-mint documentado

Páginas creadas:
- `wiki/proyectos/venta-inventario-frontend-negocios.md` — rediseño premium de negocios-lista, KPI signals, nivel-badge, mobile rows, acciones y permisos, gotcha Tailwind JIT

Páginas actualizadas:
- `wiki/proyectos/venta-inventario-frontend-diseno.md` — patrón `view-mint` (fondo mint + scroll de página), regla anti-doble-fondo/doble-padding, tabla comparativa vs `view-productos`

Contexto: skill `/premium-refactor` aplicado a `negocios-lista`. Bugs resueltos: (1) Tailwind JIT no genera clases `md:hidden` en archivos nuevos → usar media queries CSS custom. (2) Doble fondo visible → `view-area.view-mint` pone background, `:host` sin background; `view-area` con `padding:0`, `:host` con padding propio. Paso 2 (reemplazo original + limpieza v2) ejecutado en la misma sesión.

---

## [2026-05-15] ingest | Remotion — skill instalado + exploración de tecnología

Páginas creadas:
- `tecnologias/remotion.md` — nueva tecnología: video creation con React/TypeScript

Páginas actualizadas:
- `tecnologias/claude-code-skills.md` — sección `remotion-best-practices` skill agregada

Contexto: Christian instaló `npx skills add remotion-dev/skills`. Skill cubre: animaciones con `useCurrentFrame()`, sequencing, audio/video/GIFs, 3D (Three.js), captions, FFmpeg, Voiceover (ElevenLabs), mapas (MapLibre). No hay proyectos activos con Remotion aún — exploración inicial.

---

## [2026-05-09] ingest | Venta e Inventario — roles en tabla usuarios + indicador permisos-por-rol + bug Sequelize BelongsToMany alias

Páginas actualizadas:

- `wiki/proyectos/venta-inventario.md` — estado actual actualizado (columna Rol + indicador asignadoPorRol)
- `wiki/proyectos/venta-inventario-rbac.md` — dos secciones nuevas: indicador "Desde rol" y bug Sequelize BelongsToMany/raw

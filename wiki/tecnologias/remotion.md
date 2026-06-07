---
titulo: Remotion — Video creation in React
tipo: tecnologia
tags: [remotion, react, video, animacion, typescript]
fecha_creacion: 2026-05-15
fecha_actualizacion: 2026-05-15
estado: activo
---

# Remotion

Librería para crear videos programáticamente con React y TypeScript. Los componentes React se convierten en frames de video. El render usa Puppeteer/Chrome internamente.

## Cuándo usarla

- Generar videos dinámicos o parametrizados (ej: video resumen por usuario)
- Animaciones complejas que serían imposibles en video editores tradicionales
- Videos con datos: gráficas, subtítulos automáticos, waveforms
- Automatizar producción de contenido en masa

## Conceptos clave

### Animación

```tsx
const frame = useCurrentFrame();           // frame actual (0-based)
const { fps, durationInFrames } = useVideoConfig();

const opacity = interpolate(frame, [0, fps * 2], [0, 1], {
  extrapolateRight: "clamp",
  easing: Easing.bezier(0.16, 1, 0.3, 1),
});
```

- **PROHIBIDO**: CSS `transition`, CSS `animation`, clases Tailwind de animación → no renderizan correctamente
- **Correcto**: siempre `useCurrentFrame()` + `interpolate()`

### Sequencing

```tsx
<AbsoluteFill>
  <Sequence from={0}>                          {/* desde frame 0 */}
    <Background />
  </Sequence>
  <Sequence from={fps} durationInFrames={fps * 2} layout="none">
    <Titulo />                                 {/* aparece al segundo, dura 2s */}
  </Sequence>
</AbsoluteFill>
```

### Composición (Root.tsx)

```tsx
<Composition
  id="MiVideo"
  component={MiComposicion}
  durationInFrames={150}
  fps={30}
  width={1920}
  height={1080}
/>
```

## Assets

- Ubicar en `/public/` del proyecto
- Referenciar con `staticFile("archivo.mp4")`
- Soporta URLs remotas directamente

## Comandos esenciales

```bash
# Crear proyecto nuevo
npx create-video@latest --yes --blank --no-tailwind mi-video

# Preview en Remotion Studio
npx remotion studio

# Render un frame (sanity check)
npx remotion still MiComposicion --scale=0.25 --frame=30
```

## Skill instalado

`remotion-best-practices` — instalado vía `npx skills add remotion-dev/skills`.
Provee reglas detalladas para: audio, video, GIFs, 3D, captions, FFmpeg, voiceover, mapas.

Ver [[tecnologias/claude-code-skills]] para detalles de instalación.

## Estado en proyectos de Christian

- Exploración inicial (2026-05-15) — sin proyectos activos con Remotion aún
- Skill instalado y disponible en Claude Code para cuando se necesite

---
titulo: Claude Code Skills — Ecosystem de Extensiones
tipo: tecnologia
tags: [claude-code, ai, productividad, caveman, tokens, napkin, memoria]
fecha_creacion: 2026-05-06
fecha_actualizacion: 2026-05-15
estado: activo
---

# Claude Code Skills

Sistema de extensiones para Claude Code que amplían el comportamiento del agente. Se instalan globalmente y quedan disponibles en todos los proyectos.

## Instalación de skills

```powershell
# Instalar un skill globalmente (auto-detect agent, sin prompts)
npx skills add <owner>/<repo> --yes --global

# Se instalan en:
# ~\.agents\skills\<nombre-skill>\SKILL.md
# Y se symlinkan automáticamente a Claude Code
```

**Importante:** `npx skills add` instala los skills pero **NO** los hooks JS. Para hooks completos (stats, badges) se necesita el instalador oficial (`install.ps1`).

---

## Skills instalados: Caveman Ecosystem

Fuente: [github.com/JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman)

### Propósito

Reduce tokens de output ~75% haciendo que Claude responda como cavernícola: sin relleno, sin cortesías, solo lo técnico. Mismo resultado, menos palabras.

> "why use many token when few do trick"

### Skills disponibles

| Skill | Comando | Función |
|-------|---------|---------|
| caveman | `/caveman` | Activa modo comprimido (lite / full / ultra / wenyan) |
| caveman-commit | `/caveman-commit` | Mensajes de commit Conventional Commits ≤50 chars |
| caveman-review | `/caveman-review` | Reviews de PR en una línea por comentario |
| caveman-compress | `/caveman-compress ARCHIVO` | Comprime CLAUDE.md o notas (~46% menos tokens de entrada) |
| caveman-stats | `/caveman-stats` | Stats de tokens ahorrados (requiere hooks JS del instalador completo) |
| caveman-help | `/caveman-help` | Referencia rápida de todos los comandos |
| cavecrew | `/cavecrew` | Subagentes comprimidos — investigator / builder / reviewer |
| compress | `/compress` | Alias de caveman-compress |

### Niveles de intensidad

| Nivel | Descripción |
|-------|-------------|
| `lite` | Sin relleno ni hedging. Artículos intactos. Profesional pero sin fluff |
| `full` | Sin artículos, fragmentos OK, sinónimos cortos. Caveman clásico (default) |
| `ultra` | Máxima compresión. Abreviaturas (DB/auth/req/res/fn), flechas para causalidad |
| `wenyan` | Chino clásico literario — máxima compresión posible |

### Benchmarks reales (Claude API)

| Tarea | Normal | Caveman | Ahorro |
|-------|--------|---------|--------|
| Explicar re-render React | 1180 tok | 159 tok | 87% |
| Fix auth middleware | 704 tok | 121 tok | 83% |
| Arquitectura micro vs monolito | 446 tok | 310 tok | 30% |
| **Promedio** | **1214 tok** | **294 tok** | **65%** |

---

## Configuración aplicada en Claude Code

### settings.json (`~/.claude/settings.json`)

```json
{
  "effortLevel": "medium",
  "theme": "dark",
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "Write-Output '{\"hookSpecificOutput\":{\"hookEventName\":\"SessionStart\",\"additionalContext\":\"CAVEMAN MODE ACTIVE. Speak terse like smart caveman. Drop articles, filler, pleasantries, hedging. Fragments OK. Short synonyms. Technical terms exact. Code blocks unchanged. Pattern: [thing] [action] [reason]. Off only if user says: stop caveman / normal mode.\"}}'",
            "shell": "powershell",
            "statusMessage": "Activating caveman mode..."
          }
        ]
      }
    ]
  }
}
```

**Qué hace el hook:** Al iniciar cada sesión, inyecta las instrucciones caveman como `additionalContext` vía `SessionStart` hook. Claude recibe el modo automáticamente sin necesidad de escribir `/caveman`.

**Para desactivar en una sesión:** escribir "stop caveman" o "normal mode".

### Skills path

```
~\.agents\skills\
├── caveman\SKILL.md
├── caveman-commit\SKILL.md
├── caveman-compress\SKILL.md
├── caveman-help\SKILL.md
├── caveman-review\SKILL.md
├── caveman-stats\SKILL.md
├── cavecrew\SKILL.md
└── compress\SKILL.md
```

---

---

## Skill: Napkin — Memoria de errores por repo

Fuente: [github.com/blader/napkin](https://github.com/blader/napkin)

### Propósito

Mantiene un runbook vivo en `.claude/napkin.md` por repositorio. El modelo acumula en él: errores propios, correcciones del usuario, patrones que funcionan, gotchas del stack. En sesión 3-5 el agente empieza a prevenir errores antes de que el usuario los corrija.

> "Baby continual learning in a markdown file."

### Instalación

```bash
# Instalar globalmente (una vez)
git clone https://github.com/blader/napkin.git ~/.claude/skills/napkin
```

Se instala en `~/.claude/skills/napkin/` — disponible en todos los proyectos.

### Cómo funciona

| Acción | Quién | Cuándo |
|--------|-------|--------|
| **Lectura** del napkin | SessionStart hook (automático) | Al inicio de cada sesión |
| **Escritura** al napkin | El modelo | Durante el trabajo, al detectar algo reutilizable |

El napkin **no es un log cronológico** — es un runbook curado. Cada entrada tiene:
- Fecha `[YYYY-MM-DD]`
- Título corto del patrón
- Línea `Do instead:` con la acción concreta

### Estructura del napkin

```markdown
# Napkin Runbook

## Execution & Validation (Highest Priority)
## Shell & Command Reliability
## Domain Behavior Guardrails
## User Directives
```

Máx. 10 entradas por categoría. El modelo re-prioriza y elimina entradas de bajo valor en cada lectura.

### Ubicación por proyecto

```
<proyecto>/
└── .claude/
    └── napkin.md   ← uno por repo, gitignoreable o commitable
```

---

## Configuración aplicada en Claude Code

### settings.json (`~/.claude/settings.json`)

```json
{
  "effortLevel": "medium",
  "theme": "dark",
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "Write-Output '{\"hookSpecificOutput\":{\"hookEventName\":\"SessionStart\",\"additionalContext\":\"CAVEMAN MODE ACTIVE. Speak terse like smart caveman. Drop articles, filler, pleasantries, hedging. Fragments OK. Short synonyms. Technical terms exact. Code blocks unchanged. Pattern: [thing] [action] [reason]. Off only if user says: stop caveman / normal mode.\"}}'",
            "shell": "powershell",
            "statusMessage": "Activating caveman mode..."
          },
          {
            "type": "command",
            "command": "$f='.claude/napkin.md'; if(Test-Path $f){ $c=Get-Content $f -Raw; [Console]::OutputEncoding=[System.Text.Encoding]::UTF8; Write-Output (@{hookSpecificOutput=@{hookEventName='SessionStart';additionalContext=\"NAPKIN RUNBOOK (apply silently):`n$c\"}}|ConvertTo-Json -Compress) }",
            "shell": "powershell",
            "statusMessage": "Loading napkin..."
          }
        ]
      }
    ]
  }
}
```

**Qué hace cada hook:**
1. **Caveman hook** — inyecta instrucciones de modo comprimido. Auto-activa sin `/caveman`.
2. **Napkin hook** — si existe `.claude/napkin.md` en el directorio actual, inyecta su contenido como contexto. Si no existe, no hace nada.

**Para desactivar caveman en una sesión:** escribir "stop caveman" o "normal mode".

### Skills path

```
~\.claude\skills\
├── caveman\SKILL.md
├── caveman-commit\SKILL.md
├── caveman-compress\SKILL.md
├── caveman-help\SKILL.md
├── caveman-review\SKILL.md
├── caveman-stats\SKILL.md
├── cavecrew\SKILL.md
├── compress\SKILL.md
└── napkin\SKILL.md       ← nuevo
```

---

## Skill: remotion-best-practices — Videos con React

Fuente: [github.com/remotion-dev/skills](https://github.com/remotion-dev/skills)

### Propósito

Inyecta conocimiento experto de Remotion (librería para crear videos programáticamente con React/TypeScript). Activa cuando se trabaja con código Remotion.

### Instalación

```bash
npx skills add remotion-dev/skills
# Instala en: <proyecto>\.agents\skills\remotion-best-practices\
# Symlink automático a Claude Code
```

### Qué cubre

| Área | Detalle |
|------|---------|
| **Core** | `useCurrentFrame()`, `interpolate()`, `<Sequence>`, `<Composition>` |
| **Prohibiciones** | CSS transitions/animations y Tailwind animation classes — rompen el render |
| **Media** | `<Audio>`, `<Video>`, `<Img>`, GIFs, Lottie, videos transparentes |
| **Texto** | Animaciones tipográficas, Google Fonts, fuentes locales, captions/SRT |
| **Avanzado** | Audio visualization, 3D (Three.js/R3F), Voiceover (ElevenLabs), FFmpeg, detección de silencio |
| **Datos** | Videos parametrizados con Zod, duración/dimensiones dinámicas |
| **Mapas** | MapLibre para rutas animadas y flyovers |

---

## Pendiente / Limitaciones conocidas

- `/caveman-stats` **no funciona** — requiere `hooks/caveman-stats.js` del instalador completo (`install.ps1`). No se instaló al usar `npx skills add`.
- Badge de statusline `[CAVEMAN] ⛏ 12.4k` tampoco activo por la misma razón.
- Para instalar los hooks completos: `irm https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.ps1 | iex`
- El napkin hook usa `shell: "powershell"` — en sistemas sin PowerShell habría que adaptar el comando.

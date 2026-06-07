---
titulo: Proyecto SPA — Base del Stack Angular + Node
tipo: proyecto
tags: [spa, fullstack, angular, nodejs, arquitectura, base, ai-workflow]
fecha_creacion: 2026-05-06
fecha_actualizacion: 2026-05-06
estado: activo
---

# Proyecto SPA

Proyecto SPA fullstack que parece ser la **base arquitectónica** reutilizada en múltiples proyectos de Christian.

## Descripción

SPA (Single Page Application) con Angular + Node.js. Lo que lo distingue de otros proyectos es la presencia de:
1. Carpeta `documentacion/` con ADR (Architecture Decision Records)
2. Carpeta `ai-workflow/` — integración de flujos con IA
3. Es el proyecto cuyo nombre interno (`ProyectoSpaBk`) aparece en múltiples otros proyectos, sugiriendo que es el template origen

## Stack

- **Backend**: Node.js (`proyecto_spa_bk/` con `package.json`)
- **Frontend**: Angular 18.x (`proyecto_spa_ng/` con `angular.json`)
- **Documentación**: ADR en `documentacion/README.md`
- **IA**: `ai-workflow/`

## Estructura

```
proyecto-spa/
├── documentacion/
│   └── README.md  ← Architecture Decision Records
├── proyecto_spa_bk/   ← Backend
│   └── package.json
├── proyecto_spa_ng/   ← Frontend
│   ├── package.json
│   └── angular.json
└── ai-workflow/       ← Workflows con IA
```

## Por qué es relevante

Este proyecto es probablemente el **más maduro y documentado** del portafolio. El hecho de que:
- Tenga ADR (decisiones de arquitectura documentadas)
- Tenga un módulo de `ai-workflow`
- Su nombre aparezca en otros 3+ proyectos como template base

...indica que Christian lo usa como punto de referencia o boilerplate.

## Preguntas abiertas

- ¿Qué decisiones están documentadas en los ADR?
- ¿Qué hace exactamente `ai-workflow`? ¿Integración con Claude, OpenAI?
- ¿Es el proyecto activo más importante actualmente?

## Links relacionados

- [[ecommerce]] — mismo template base
- [[autenticacion-prototipo]] — mismo template base
- [[venta-inventario]] — mismo template base

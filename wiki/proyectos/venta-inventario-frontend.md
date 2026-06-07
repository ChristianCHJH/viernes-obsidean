---
titulo: Venta e Inventario — Frontend Angular
tipo: proyecto
tags: [inventario, angular, angular-material, tailwind, adifnex]
fecha_creacion: 2026-05-07
fecha_actualizacion: 2026-06-07
estado: activo
---

# Venta e Inventario — Frontend Angular

Sub-página de [[venta-inventario]].

## Estado actual

Angular 18.2 + Angular Material 18.2 (migración PrimeNG completada 2026-06-03).
Design system Adifnex (`--p-*`, paleta verde `#14532D` + naranja `#EA580C`).
Sidebar blanco/verde v2 (promovido 2026-06-06, sesión 43).

## Stack

| Capa | Tecnología |
|------|-----------|
| Framework | Angular 18.2 (standalone components, Signals) |
| UI Components | Angular Material 18.2 |
| Estilos | Tailwind CSS + tokens `--p-*` (Adifnex) + `--spa-*` (shell) |
| ORM frontend | HttpBaseService con desempaquetado centralizado |
| Iconos | Material Icons (ligatures) |
| Tipografía | DM Sans (Google Fonts) |

## Sub-páginas

- [[venta-inventario-frontend-migracion-material]] — Migración PrimeNG → Angular Material (7 fases, completado 2026-06-03)
- [[venta-inventario-frontend-patrones]] — HTTP envelope, tenant reload, toObservable+switchMap, guard contexto
- [[venta-inventario-frontend-permisos-rbac]] — Permisos UI, nivel inline, secciones unificadas, reasignación negocio
- [[venta-inventario-frontend-gotchas]] — Gotchas Angular/CSS/Material (colección completa)
- [[venta-inventario-frontend-productos]] — Marcas/Etiquetas, bugs lista productos, UX rediseño, paginación
- [[venta-inventario-frontend-registro]] — Registro 2 pasos, guards, bugs responsive
- [[venta-inventario-frontend-productos-premium]] — Modal 4 tabs, design system Adifnex, SKU automático, eliminar
- [[venta-inventario-frontend-productos-premium-ux]] — UX mobile, bottom sheets, card compact `prod-row`, paginación fix
- [[venta-inventario-frontend-diseno]] — Tokens globales `--p-*`, sidebar v2, top bar, negative margins, CSS budget
- [[venta-inventario-frontend-sedes]] — Sedes lista premium, drawer stock, toggle custom, select nativo

## Páginas relacionadas

- [[venta-inventario]] — Hub principal del proyecto
- [[venta-inventario-testing]] — Jest + Playwright
- [[venta-inventario-catalogo]] — Catálogo público Next.js
- [[venta-inventario-rbac]] — RBAC backend + SQL seed permisos

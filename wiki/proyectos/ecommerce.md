---
titulo: Ecommerce
tipo: proyecto
tags: [ecommerce, fullstack, angular, nodejs, spa]
fecha_creacion: 2026-05-06
fecha_actualizacion: 2026-05-06
estado: activo
---

# Ecommerce

Proyecto de tienda en línea fullstack con Angular y Node.js.

## Descripción

SPA (Single Page Application) de comercio electrónico. La nomenclatura interna (`proyecto_spa_bk` y `proyecto_spa_ng`) sugiere que es una de las primeras versiones del stack SPA que Christian ha desarrollado y refinado en múltiples proyectos.

## Stack

- **Backend**: Node.js (`proyecto_spa_bk/` con `package.json`)
- **Frontend**: Angular 18.x (`proyecto_spa_ng/` con `angular.json`)

## Estructura

```
ecommerce/
├── proyecto_spa_bk/    ← Backend Node.js
│   └── package.json
└── proyecto_spa_ng/    ← Frontend Angular
    ├── package.json
    └── angular.json
```

## Notas

- El nombre interno `ProyectoSpaBk` en el README de Angular sugiere que este es el template base que se reutiliza en varios proyectos (ver también [[proyecto-spa]], [[autenticacion-prototipo]], [[venta-inventario]])
- Angular CLI v18.2.19

## Links relacionados

- [[proyecto-spa]] — versión más evolucionada / base del mismo stack
- [[comercio-centro-lima]] — proyecto de comercio similar
- [[autenticacion-prototipo]] — comparte la misma nomenclatura interna

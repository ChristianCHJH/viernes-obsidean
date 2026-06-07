---
titulo: Stack Principal de Christian
tipo: tecnologia
tags: [angular, nestjs, nodejs, typescript, postgresql, tailwind, primeng]
fecha_creacion: 2026-05-06
fecha_actualizacion: 2026-05-06
estado: activo
---

# Stack Principal

El stack que Christian usa por defecto en todos sus proyectos fullstack.

## Backend

| Tecnología | Versión | Rol |
|-----------|---------|-----|
| Node.js | 20.x | Runtime |
| NestJS | 10.x | Framework |
| TypeScript | 5.x | Lenguaje |
| TypeORM | latest | ORM |
| PostgreSQL | latest | Base de datos |

## Frontend

| Tecnología | Versión | Rol |
|-----------|---------|-----|
| Angular | 17+ | Framework |
| TypeScript | 5.x | Lenguaje |
| Tailwind CSS | latest | Estilos |
| PrimeNG | latest | Componentes UI |

## Mobile

| Tecnología | Rol |
|-----------|-----|
| Kotlin | Lenguaje Android |
| Android nativo | Plataforma |

## Automatización / DevOps

| Tecnología | Rol |
|-----------|-----|
| n8n | Workflows de automatización |
| Playwright | Web scraping |
| Docker | Contenedores (n8n, Playwright) |

## Decisiones de diseño establecidas

### PostgreSQL
- IDs: `BIGINT GENERATED ALWAYS AS IDENTITY` (nunca SERIAL)
- Columnas de auditoría obligatorias en toda tabla: `usuario_creacion`, `usuario_actualizacion`, `fecha_creacion`, `fecha_actualizacion`, `estado`, `eliminado`
- Soft delete siempre: nunca `DELETE` físico

### NestJS
- Respuestas HTTP estandarizadas: `{ success, statusCode, message, data, meta? }`
- `TransformInterceptor` global para éxito
- `HttpExceptionFilter` global para errores

### Angular
- Standalone components por defecto (Angular 17+)
- Sintaxis moderna: `@if`, `@for`, `@switch` (nunca `*ngIf`, `*ngFor`)
- `OnPush` change detection
- `async` pipe en templates
- Sin `any` en TypeScript

## Proyectos que usan este stack

- [[comercio-centro-lima]]
- [[ecommerce]]
- [[museo]]
- [[metropolitano]]
- [[soporte]]
- [[venta-inventario]]
- [[proyecto-spa]]
- [[autenticacion-prototipo]]

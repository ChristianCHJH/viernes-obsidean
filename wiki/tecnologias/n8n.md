---
titulo: n8n — Automatización de Workflows
tipo: tecnologia
tags: [n8n, automatizacion, workflows, no-code, docker]
fecha_creacion: 2026-05-06
fecha_actualizacion: 2026-05-06
estado: activo
---

# n8n

Plataforma de automatización de flujos de trabajo (workflow automation). Christian la usa como capa de orquestación en proyectos que requieren automatización de procesos.

## Qué es

n8n es una herramienta de automatización visual (similar a Zapier/Make) que se puede auto-alojar. Permite conectar servicios, APIs y bases de datos mediante nodos visuales.

## Cómo lo usa Christian

- Desplegado en Docker
- Separado del backend principal como módulo independiente
- Maneja la lógica de automatización que no pertenece al core de la app

## Proyectos donde aparece

- [[soporte]] — automatización de tickets (asignación, notificaciones, escalamiento)
- [[super-mercado]] — orquestación del scraping de precios

## Casos de uso observados

1. **Tiquetera de soporte**: Flujos de asignación automática, notificaciones, escalamiento
2. **Scraping**: Coordinación de workers Playwright para extraer precios

## Patrón arquitectónico

Christian separa n8n del backend en su propia carpeta/servicio. Esto sugiere que lo trata como una herramienta externa que complementa el backend, no como parte del core.

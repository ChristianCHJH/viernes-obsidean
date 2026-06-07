---
titulo: Comparador de Precios de Supermercados Peruanos
tipo: proyecto
tags: [scraping, comparador-precios, python, fastapi, postgresql, n8n, playwright, angular]
fecha_creacion: 2026-05-06
fecha_actualizacion: 2026-05-06
estado: activo
---

# Comparador de Precios de Supermercados Peruanos

Sistema automatizado para monitorear y comparar precios en supermercados peruanos.

## Descripción

Plataforma de scraping e inteligencia de precios que monitora múltiples supermercados peruanos de forma automatizada. Es el proyecto con el stack más diferente al resto — usa Python en el backend en lugar de Node.js.

## Supermercados cubiertos

- Metro
- Wong
- Plaza Vea
- Tottus
- Vega
- Makro

## Stack

- **Backend**: Python 3.11 + FastAPI + SQLAlchemy 2.0
- **Base de datos**: PostgreSQL
- **Automatización**: n8n (Docker)
- **Scraping**: Playwright (Docker)
- **Frontend**: Angular (`frontend-angular/` con `angular.json`)

## Estructura

```
super_mercado/
└── comparador-precios/
    ├── README.md
    └── frontend-angular/
        ├── package.json
        └── angular.json
```

## Arquitectura notable

Este proyecto es **único** en el portafolio de Christian porque:
1. Usa Python (FastAPI) en lugar de Node.js/NestJS
2. Usa Docker extensamente (n8n y Playwright dockerizados)
3. El scraping con Playwright está separado del backend principal

## Modelo de negocio potencial

- Herramienta para consumidores que quieren comprar más barato
- Datos de precios como servicio (B2B) para retailers
- Inteligencia competitiva para supermercados

## Preguntas abiertas

- ¿Tiene mecanismo anti-detección para el scraping?
- ¿Con qué frecuencia se actualizan los precios?
- ¿Es un proyecto personal o tiene cliente?

## Links relacionados

- [[soporte]] — también usa n8n
- [[tecnologias/n8n]] — automatización
- [[tecnologias/playwright]] — scraping
- [[modelos-negocio/datos-como-servicio]] — modelo potencial

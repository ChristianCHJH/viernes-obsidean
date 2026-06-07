---
titulo: Soporte — Tiquetera con Automatización
tipo: proyecto
tags: [soporte, tiquetera, helpdesk, fullstack, angular, nodejs, n8n, automatizacion]
fecha_creacion: 2026-05-06
fecha_actualizacion: 2026-05-06
estado: activo
---

# Soporte — Tiquetera con Automatización

Sistema de tiquetera de soporte técnico con automatización de flujos mediante n8n.

## Descripción

Plataforma de gestión de tickets de soporte. Lo que lo distingue es la integración con **n8n** para automatizar flujos de trabajo: asignación automática, notificaciones, escalamiento, etc.

## Stack

- **Backend**: Node.js (`backend/` con `package.json`)
- **Frontend**: Angular (`frontend/` con `angular.json`)
- **Automatización**: n8n (`n8n/` con `README.md`)

## Estructura

```
Soporte/
├── backend/
│   └── package.json
├── frontend/
│   ├── package.json
│   └── angular.json
└── n8n/
    └── README.md  ← "Workflows de n8n — Tiquetera de Soporte"
```

## Funcionalidades identificadas

- Sistema de tickets de soporte
- Workflows de automatización con n8n
- Gestión desde interfaz web n8n

## Arquitectura notable

La separación del módulo n8n como carpeta independiente indica una arquitectura donde la lógica de automatización vive aparte del backend principal. Esto permite escalar los flujos sin tocar el core de la app.

## Links relacionados

- [[super-mercado]] — también usa n8n para automatización
- [[tecnologias/n8n]] — tool de automatización usada aquí
- [[modelos-negocio/saas-b2b]] — posible modelo de negocio

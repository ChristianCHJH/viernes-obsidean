---
titulo: Adifnex — Modelo SaaS Comercial
tipo: modelo-negocio
tags: [adifnex, saas, ecommerce, catalogo, peru, pricing, lanzamiento]
fecha_creacion: 2026-05-29
fecha_actualizacion: 2026-05-31
estado: activo
---

# Adifnex — Modelo SaaS Comercial

Hub de decisiones comerciales, estrategia de producto y pendientes de lanzamiento.

## Decisión de enfoque

**Opción elegida: B — Storefront / Catálogo público**

El diferenciador no es el inventario (eso lo tiene Alegra/Bsale). El diferenciador es que Adifnex da a cada negocio una **tienda pública con URL propia** integrada con su inventario. Nadie en Perú tiene eso en un solo producto accesible para PyMEs.

Técnicamente el sistema ya soporta ambos (gestión interna + catálogo público) — el pitch de venta se enfoca en el catálogo.

## Propuesta de valor

> "Tu tienda online profesional con catálogo siempre actualizado — sin pagar un desarrollador."

Problema real del cliente: muestran productos por WhatsApp o Instagram sin precio claro, sin URL propia, sin imagen profesional.

## Mercado

- **Inicial:** Perú
- **Futuro:** LatAm
- **Segmento:** PyMEs con productos físicos — tiendas de ropa, perfumerías, ferreterías, bazares, artesanías

## Pricing definido (2026-05-29)

Referencia de mercado: Tiendanube Perú desde S/49/mes.

| Plan | Precio mensual | Precio anual (20% dto) | Incluye |
|---|---|---|---|
| **Inicio** | S/29 | S/278 | 1 sede, 3 usuarios, 1 plantilla |
| **Crecer** | S/59 | S/566 | 3 sedes, 10 usuarios, todas las plantillas |
| **Pro** | S/99 | S/950 | Sedes ilimitadas, usuarios ilimitados, dominio propio* |

*dominio propio: gap técnico pendiente, anunciar como "próximamente"

**Plan destacado:** Crecer (S/59) — es el ancla visual en la pricing page.

### Estrategia de adquisición — primeros 10 clientes (meta: julio 2026)

- **Modelo:** Trial con tarjeta registrada desde el inicio (no freemium)
- Tarjeta requerida evita usuarios fantasma y convierte mejor

| Código | Beneficio | Uso |
|---|---|---|
| `ADIFNEX1` | 2 meses gratis | Primeros 10 clientes — exclusividad |
| `JULIO2026` | 50% primeros 3 meses | Lanzamiento público |
| `REFERIDO` | 1 mes gratis | Al referir un negocio (activar cuando haya 3+ clientes) |

**Por qué 2 meses > 1 mes:** Con 2 meses el cliente ya tiene su catálogo publicado y compartido. Cancelar duele más que si apenas está configurando.

### Descuento anual
20% de descuento pagando anual = 2 meses gratis. Mostrar precio anual primero en la landing.

## Competencia

| Competidor | Falla en |
|---|---|
| Shopify / Tiendanube | Caro, sin inventario multi-sede, no pensado para Perú |
| Alegra / Bsale | Solo gestión interna, sin storefront público |
| Instagram / WhatsApp | No es tienda, sin URL propia, no escala |
| Web a medida | Miles de soles + meses + dependencia del técnico |

## Stack técnico ya construido

- Multi-tenancy shared schema (row-level isolation)
- RBAC 3 capas + tiers de plan (`nivel_plan` en JWT)
- Catálogo Next.js por slug (`/[slug]`) — 2 plantillas activas
- Panel Angular para gestión
- App Android (Kotlin)
- Backend NestJS + PostgreSQL

Ver [[proyectos/venta-inventario]] para detalle técnico.

## Gaps para lanzar (roadmap comercial)

### P0 — Bloqueantes para primer cliente pagante
- [x] Página de registro público (autoservicio) ✅ 2026-05-31
- [ ] Integración de pagos (Culqi o Stripe)
- [ ] Activar `fecha_vencimiento` (cortar acceso si no paga)

### P1 — Para crecer
- [x] Landing page de ventas con pricing público ✅ 2026-05-31
- [ ] Onboarding guiado post-registro — secuencia guiada post-registro: nombre negocio → primer producto → compartir link
- [ ] Dominio personalizado por negocio

### P2 — Diferenciadores futuros
- [ ] Más plantillas de catálogo
- [ ] Analytics para el negocio (visitas a su catálogo)
- [ ] Integración con WhatsApp (compartir link del catálogo)

## Sistema de diseño — Paleta de marca

Definida 2026-05-31:

| Token | Valor | Uso |
|---|---|---|
| Primary text | `#16A34A` | Headings, textos principales |
| Primary dark | `#064E3B` | Sidebar Angular, fondos oscuros |
| Primary medium | `#15803D` | Links, badges, bordes |
| Primary light | `#DCFCE7` | Fondos suaves, hover |
| Surface | `#F0FDF4` | Fondo secciones claras |
| Accent CTA | `#EA580C` | Botones CTA principales |
| Accent hover | `#C2410C` | Hover botones CTA |

**Decisión:** Verde esmeralda + naranja. Diferencia de toda la competencia (azul). Verde = crecimiento/dinero para PyMEs peruanas. Naranja = calidez/acción.

**Anti-referencia:** `#022C22` (forest muy oscuro) — demasiado pesado, reemplazado por `#064E3B`.

## Arquitectura de la landing

- **Proyecto:** `catalogo/` (Next.js) — ruta `/`
- **Ruta catálogos:** `/[slug]` — mismo proyecto
- **CTA apunta a:** `app.adifnex.com/registro` (Angular frontend)
- **Imágenes:** `catalogo/public/img/` — fotos reales generadas con Gemini
- **Pasarela recomendada:** Culqi (Perú nativo, soporta recurrencia, self-service)

## Pendientes de decisión

- [x] Pricing definitivo: S/29 / S/59 / S/99 (definido 2026-05-29)
- [x] Modelo de adquisición: trial con tarjeta + códigos de descuento
- [x] Paleta de marca: verde esmeralda `#16A34A` + naranja `#EA580C` (definido 2026-05-31)
- [ ] ¿Pasarela de pago: Culqi (recomendado) o Stripe?
- [ ] Nombre de dominio público del producto
- [ ] ¿Adifnex es el nombre final de cara al cliente?

## Log de sesiones

### 2026-05-29 — Sesión 1 (estrategia inicial + pricing)
- Pricing definido: S/29 / S/59 / S/99 (referencia: Tiendanube desde S/49)
- Descuento anual: 20% (2 meses gratis)
- Códigos: ADIFNEX1 (2 meses gratis), JULIO2026 (50% x3 meses), REFERIDO (1 mes)
- Meta: 5-10 clientes pagando antes de julio 2026

### 2026-05-31 — Sesión 2 (landing page + sistema de diseño)
- Landing construida en `catalogo/app/page.tsx` (Next.js) — 8 secciones completas
- Paleta de marca definida: verde `#16A34A` + naranja `#EA580C`
- Angular y Next.js migrados de azul marino/teal a verde esmeralda
- Imágenes generadas con Gemini: fotos reales de dueños de tienda limeños (no stock genérico)
- Prompts Gemini documentados: especificar "NO traditional/indigenous clothing", "Lima urbano moderno"
- Pasarela recomendada: Culqi (vs Niubiz: no soporta recurrencia nativa)
- Onboarding definido conceptualmente: 3 pasos post-registro
- P1 landing marcado ✅

### 2026-05-29 — Sesión 1 — original
- Decisión: enfoque en catálogo público (opción B)
- Pricing pensado en soles: S/15 / S/19 / S/29
- Mercado inicial: Perú
- Se creó `.agents/product-marketing.md` con contexto base
- Gaps identificados: registro, pagos, landing

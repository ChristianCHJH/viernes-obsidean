---
titulo: Paquete de Marketing Adifnex — Playbook de las 43 skills
tipo: sintesis
tags: [marketing, adifnex, growth, lanzamiento, playbook, skills]
fecha_creacion: 2026-06-22
fecha_actualizacion: 2026-06-22
estado: activo
---

# Paquete de Marketing Adifnex — Playbook

Índice operativo del sistema de marketing de Viernes. Conecta las **43 marketing skills** (instaladas desde `coreyhaines31/marketingskills` en `.claude/skills/`) con el estado real de [[modelos-negocio/adifnex-saas|Adifnex]] y el agente [[../.claude/agents/marketing-asesor|marketing-asesor]].

**Cómo se usa:** invoca al agente `marketing-asesor` o, si trabajas directo, lanza la skill puntual con la tool `Skill`. El contexto de producto vive en `.agents/product-marketing.md` y casi todas las skills lo leen automáticamente.

## Momento actual del producto

- **Etapa:** pre-lanzamiento. **0 clientes pagantes.** Meta: primeros 10 en Perú (objetivo ~julio 2026).
- **Diferenciador:** tienda pública con URL propia + inventario integrado, para PyMEs LatAm.
- **Modelo:** SaaS. Planes S/29 / S/59 / S/99. Ancla = "Crecer" S/59.
- **Cobro actual (manual e intencional):** el cliente paga por **Yape directo a Christian**. Flujo = catálogo/landing → **WhatsApp** → cierre conversado → cobro Yape → activación manual. Pagos automáticos (Culqi/Stripe) y registro autoservicio = diferidos, NO bloquean cobrar.
- **Implicación de marketing:** el CTA de todo (landing, copy, outreach) es **"Escríbeme por WhatsApp"**, no "regístrate" ni "paga aquí". El canal de conversión es la conversación, no un checkout.

## Secuencia recomendada (no hacer todo a la vez)

| Fase | Foco | Skills clave |
|---|---|---|
| **0. Cimientos** | Mensaje y prueba de que el problema duele | `product-marketing`, `customer-research`, `marketing-psychology`, `competitors` |
| **1. Convertir** | Landing + pricing page que cierren al primer cliente | `copywriting`, `cro`, `pricing`, `landing`/`launch`, `signup`, `popups` |
| **2. Atraer (barato)** | Tráfico orgánico antes que pago | `seo-audit`, `ai-seo`, `content-strategy`, `social`, `directory-submissions`, `community-marketing` |
| **3. Outbound directo** | Buscar los primeros 10 a mano | `prospecting`, `cold-email`, `co-marketing`, `marketing-ideas` |
| **4. Retener y referir** | Que el cliente publique y no cancele | `onboarding`, `churn-prevention`, `referrals`, `emails` |
| **5. Medir y escalar** | Solo cuando 1–4 funcionan | `analytics`, `ab-testing`, `ads`, `ad-creative`, `aso` |

Regla de oro para esta etapa: **los primeros 10 clientes se consiguen a mano (Fase 3) + una landing que empuje a WhatsApp (Fase 1)**, no con ads. Posponer Fase 5 hasta tener señal de conversión.

**Funnel real hoy (manual):** Catálogo/landing → clic en **WhatsApp** → Christian conversa y cierra → cliente paga por **Yape** → Christian activa la cuenta a mano. Optimizar (`cro`, `copywriting`) para maximizar el clic a WhatsApp, no para un checkout. El `pricing` se muestra para anclar valor, pero la acción es escribir, no pagar en el sitio.

## Catálogo completo de skills por etapa

### Posicionamiento y mensaje
- `product-marketing` — mantiene `.agents/product-marketing.md`, el perfil canónico.
- `customer-research` — entrevistas, jobs-to-be-done, lenguaje del cliente.
- `marketing-psychology` — sesgos y palancas de decisión.
- `copywriting` / `copy-editing` — escribir y pulir copy de páginas.
- `competitors` / `competitor-profiling` — mapear Tiendanube, Shopify, Bsale, Instagram.

### Adquisición orgánica
- `seo-audit`, `ai-seo`, `programmatic-seo` — SEO técnico y de contenido.
- `content-strategy`, `social` — plan editorial y redes.
- `site-architecture`, `schema` — estructura y datos estructurados del sitio.
- `community-marketing`, `directory-submissions` — comunidades y directorios.
- `aso` — App Store (relevante para la app Android cuando se publique).

### Adquisición pago / outbound
- `ads`, `ad-creative` — campañas y creatividades pagas.
- `cold-email`, `prospecting` — outbound directo a PyMEs.
- `co-marketing`, `marketing-ideas` — alianzas y tácticas creativas.

### Conversión
- `cro`, `popups`, `paywalls`, `signup` — optimización del embudo de registro.
- `pricing` — tabla de planes, anclaje, descuentos (`ADIFNEX1`, `JULIO2026`).
- `lead-magnets`, `free-tools` — imanes de captación.
- `launch` — checklist y narrativa de lanzamiento.

### Activación y retención
- `onboarding` — que el negocio publique su catálogo el día 1.
- `churn-prevention`, `referrals` — retención y crecimiento orgánico.
- `emails`, `sms` — ciclos de vida y nurturing.
- `sales-enablement`, `revops` — soporte a ventas y operación de ingresos.

### Medición y producción
- `analytics`, `ab-testing` — instrumentación y experimentos.
- `image`, `video`, `remotion-best-practices` — assets visuales y video.

## Planes vivos que alimentan al marketing

Lo que marketing debe reflejar siempre (links a las fuentes técnicas):

- [[../proyectos/venta-inventario-catalogo|Catálogo público]] — plantillas como diferenciador visual (grilla mint, esencias/lujo). El pitch vende "diseño profesional sin desarrollador".
- [[../proyectos/venta-inventario-variantes-categorias|Variantes y categorías]] — Talla/Color + filtros/facetas en el catálogo (desde 2026-06-17). Argumento de venta: catálogo navegable y filtrable, no solo una grilla plana.
- [[../proyectos/venta-inventario-frontend-productos-premium|Productos premium]] — UX de gestión que respalda el "sin técnicos".
- [[modelos-negocio/adifnex-saas|Modelo SaaS]] — pricing, códigos, roadmap P0/P1 de lanzamiento.

Cuando uno de estos planes avance, actualizar `.agents/product-marketing.md` y esta página.

## Relacionado
- [[modelos-negocio/adifnex-saas]]
- [[../proyectos/venta-inventario]]

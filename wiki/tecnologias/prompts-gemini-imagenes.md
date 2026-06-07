---
titulo: Prompts Gemini — Generación de imágenes para Adifnex
tipo: tecnologia
tags: [gemini, prompts, imagenes, adifnex, landing]
fecha_creacion: 2026-05-31
fecha_actualizacion: 2026-05-31
estado: activo
---

# Prompts Gemini — Imágenes Adifnex Landing

Prompts validados para generar imágenes fotorrealistas de dueños de tienda peruanos.
Aprendizajes documentados para reutilizar en futuras campañas.

---

## Reglas universales (aplicar a TODOS los prompts)

- `NO traditional or indigenous clothing` — evita estereotipo andino
- `NO rustic or artisan products` — evita ambiente no urbano
- `NO food products / NO grocery items` — especificar tipo de negocio
- `NO logos or brand names visible` — evita marcas inventadas
- `Lima Peru` + `modern urban` — anclar contexto geográfico
- `Photorealistic` o `candid style` — evita look de stock genérico
- Edad: `25-35` — dueño joven, no mayor de 40

---

## Prompts validados

### Hero — Mockup catálogo en dispositivo
```
Product mockup of a clean online product catalog website displayed on a
modern MacBook Pro and iPhone 15. The website header shows the brand name
"Adifnex" in green color. Product grid with items typical of a Peruvian
small business (clothing, accessories). Prices shown in Peruvian soles
format "S/ 99.00". Buttons say "Ver detalle" in green (#059669).
NO "add to cart" buttons. NO invented brand names.
Warm wooden desk background. Professional flat lay photography.
High quality, photorealistic. No text watermarks.
The website must show exactly the text 'Adifnex' as the brand name, nothing else.
```
**Resultado:** ✅ Aceptable. Revisar que no aparezca nombre inventado.

---

### Problema 1 — Dueño respondiendo WhatsApp (bandeja múltiple)
```
Realistic photo of a stressed Latin American man aged 25-35,
wearing casual modern clothing (plain white t-shirt), standing in a modern
clothing and shoe store in Lima Peru with metal shelving racks of t-shirts,
jeans and sneakers visible in background. He holds a smartphone
toward the camera showing the WhatsApp inbox screen with many
unread conversations. Chat list shows messages in Spanish like
"¿Tienen talla M?", "¿Cuánto cuesta?", "¿Hay en negro?",
"¿Hacen delivery?", "¿Tienen stock?" with green unread badges
showing numbers 3, 5, 12. WhatsApp green header clearly visible
on phone screen. Man has frustrated overwhelmed expression,
other hand on his head. Phone screen must be fully readable.
Bright fluorescent store lighting.
NO food products. NO grocery shelves. NO traditional clothing.
Photorealistic, candid style.
```
**Aprendizaje:** pedir bandeja de entrada (inbox list), no una sola conversación. Muestra el caos de múltiples clientes.

---

### Problema 2 — Sin URL propia / Instagram
```
Realistic photo of a small clothing store owner in Lima Peru showing
their products on an Instagram profile page on a smartphone. The phone
screen shows a generic Instagram grid with product photos but no clear
price or purchase option. Owner looks confused. Natural warm lighting.
No real brand names visible on screen. No text overlays.
```

---

### Problema 3 — Inventario en Excel
```
Realistic photo of a stressed Latin American man or woman aged 25-35,
wearing casual modern clothing (plain t-shirt or hoodie), sitting at
a desk inside a small clothing or shoe store in Lima Peru.
Open laptop on desk showing a messy Excel spreadsheet with product
names, sizes and stock numbers. Clothing items, shoe boxes or
folded t-shirts scattered around the desk.
Tired and frustrated facial expression.
Bright fluorescent store lighting. Modern laptop, no visible brand name.
Spreadsheet visible on screen but no readable text required.
NO traditional clothing. NO indigenous clothing. NO food products.
NO grocery items. NO rustic environment.
Photorealistic candid style.
```

---

### Cómo funciona Paso 1 — Persona registrándose (con screenshot real)
```
Realistic photo of a young Latin American woman or man aged 26-30,
wearing casual modern clothing (plain t-shirt or light shirt),
sitting at a desk inside their small clothing or shoe store in Lima Peru.
They are typing on a MacBook laptop. The laptop screen clearly shows
a web registration form with fields for username, email, phone and
password, dark left panel and white right panel with a form card.
Clothing racks and folded clothes visible in background.
Natural window light, modern clean store environment.
Focused and motivated expression.
NO logos or brand names on clothing or walls.
NO traditional clothing. Photorealistic style.
Use the attached image as the exact laptop screen content.
```
**Nota:** adjuntar screenshot de `localhost:4200/registro` al enviar a Gemini.

---

### Cómo funciona Paso 2 — Subiendo productos (con screenshot real)
```
Realistic photo of a young Latin American woman or man aged 26-30,
wearing casual modern clothing (plain t-shirt), sitting at a desk
inside their small clothing or shoe store in Lima Peru.
They are typing on a MacBook laptop. The laptop screen clearly shows
a product management dashboard with a modal form titled
"Nuevo producto" with fields for name, SKU, description and stock.
White sidebar navigation visible on left, product table in background.
Clothing racks and folded clothes visible in store background.
Natural window light, modern clean environment.
Focused and engaged expression.
NO logos or brand names on clothing or walls.
NO traditional clothing. Photorealistic style.
Use the attached image as the exact laptop screen content.
```
**Nota:** adjuntar screenshot del modal "Nuevo producto" del panel Angular.

---

### Cómo funciona Paso 3 — Compartiendo link (con screenshot catálogo)
```
Realistic photo of a young Latin American woman or man aged 26-30,
wearing casual modern clothing (plain t-shirt), standing inside
their small clothing or shoe store in Lima Peru.
They are holding a smartphone toward camera showing a clean online
product catalog webpage with product grid and prices in soles S/.
They have a happy satisfied expression, showing the phone proudly.
Clothing racks visible in background. Bright natural lighting.
Modern iPhone, screen fully visible and readable.
NO logos or brand names on clothing or walls.
NO traditional clothing. Photorealistic candid style.
Use the attached image as the exact phone screen content.
```
**Nota:** adjuntar screenshot de `localhost:3010/[slug]`.

---

### CTA Final — Dueño exitoso
```
Realistic photo of a happy and confident young Latin American
woman or man aged 26-32, wearing casual modern clothing
(plain t-shirt or light shirt), standing proudly in front of
their well-organized small clothing or shoe store in Lima Peru.
Big genuine smile. Arms crossed or thumbs up gesture.
Metal shelving with organized clothing and sneakers visible behind.
Bright clean fluorescent lighting. Modern urban store environment.
NO traditional clothing. NO indigenous clothing.
NO brand names or logos visible. No text overlays.
Professional photography, slight wide angle.
```

---

## Técnica: reemplazar pantalla en foto existente

Cuando Gemini no usa el screenshot adjunto como pantalla:

```
Edit this photo: replace the laptop screen content exactly with
the attached screenshot image. Keep everything else identical —
the person, the store, the lighting, the angle. Only change
the laptop screen to show the attached image exactly as provided.
```

Alternativa más rápida: usar Canva "reemplazar imagen en pantalla" — 2 minutos.

---

## Archivos generados — ubicación

```
catalogo/public/img/
├── Mockup catálogo en dispositivo.png   ← Hero
├── Dueño respondiendo WhatsApp.png      ← Problema 1
├── Sin URL propia.png                   ← Problema 2
├── Inventario en Excel.png              ← Problema 3
└── Registro.png                         ← Paso 01
```

Pendientes: Paso 02 (subir productos), Paso 03 (compartir link), CTA final.

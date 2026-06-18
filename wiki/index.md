# Índice Maestro — Viernes

Catálogo de todas las páginas del wiki. Actualizar en cada ingestión.

Última actualización: 2026-06-07 (restructure frontend)

---

## Proyectos (13)

| Página | Descripción | Estado |
| ------ | ----------- | ------ |
| [[proyectos/comercio-centro-lima]] | Sistema de comercio para Lima, fullstack Angular + Node | activo |
| [[proyectos/driver-acompanamiento]] | App Android Kotlin para conductores/acompañamiento | activo |
| [[proyectos/ecommerce]] | Tienda en línea SPA, base del template ProyectoSpa | activo |
| [[proyectos/metropolitano]] | Sistema de monitoreo de flota GPS — fue prueba técnica | archivado |
| [[proyectos/museo]] | Aplicación web para gestión de museo | activo |
| [[proyectos/soporte]] | Tiquetera de soporte con automatización n8n | activo |
| [[proyectos/super-mercado]] | Comparador de precios supermercados peruanos (Python + n8n) | activo |
| [[proyectos/venta-inventario]] | Inventario multi-sede con app Android — hub | activo |
| [[proyectos/venta-inventario-auth]] | Auth, JWT, refresh token, BIGINT fix | activo |
| [[proyectos/venta-inventario-multitenancy]] | Multi-tenancy, tabla negocio, tiers | activo |
| [[proyectos/venta-inventario-inventario]] | Módulos, tablas, endpoints, flujos | activo |
| [[proyectos/venta-inventario-rbac]] | RBAC 3 capas, permisos, roles, SQL seed | activo |
| [[proyectos/venta-inventario-android]] | App Adifnex, MVVM, TokenAuthenticator | activo |
| [[proyectos/venta-inventario-frontend]] | Hub Angular — stack, estado, links a sub-páginas | activo |
| [[proyectos/venta-inventario-frontend-migracion-material]] | Migración PrimeNG → Angular Material (7 fases, completado 2026-06-03) | activo |
| [[proyectos/venta-inventario-frontend-patrones]] | Patrones HTTP envelope, tenant reload, toObservable+switchMap, guard contexto | activo |
| [[proyectos/venta-inventario-frontend-permisos-rbac]] | Permisos UI, nivel inline, secciones unificadas, reasignación negocio | activo |
| [[proyectos/venta-inventario-frontend-gotchas]] | Gotchas Angular/CSS/Material — colección completa | activo |
| [[proyectos/venta-inventario-frontend-productos]] | Marcas/Etiquetas, bugs lista productos, UX rediseño, paginación | activo |
| [[proyectos/venta-inventario-frontend-registro]] | Registro 2 pasos, guards, bugs responsive | activo |
| [[proyectos/venta-inventario-frontend-productos-premium]] | Modal 4 tabs, design system Adifnex, SKU automático, eliminar | activo |
| [[proyectos/venta-inventario-frontend-productos-premium-ux]] | UX mobile, bottom sheets, card compact prod-row, paginación fix | activo |
| [[proyectos/venta-inventario-frontend-diseno]] | Tokens globales --p-*, sidebar v2, top bar, negative margins, CSS budget | activo |
| [[proyectos/venta-inventario-frontend-sedes]] | Sedes lista premium, drawer stock, toggle custom, select nativo | activo |
| [[proyectos/venta-inventario-testing]] | Jest 114 tests backend, mocks Sequelize, Playwright plan | activo |
| [[proyectos/venta-inventario-catalogo]] | Catálogo público Next.js, plantillas, permisos Plan 2, migración 014 | activo |
| [[proyectos/venta-inventario-variantes-categorias]] | Variantes (Talla/Color), categorías, atributos — 10 tablas, módulos backend, filtro catálogo | activo |
| [[proyectos/proyecto-spa]] | Base arquitectónica SPA con ADR y ai-workflow | activo |
| [[proyectos/autenticacion-prototipo]] | Prototipo de sistema de autenticación JWT | archivado |
| [[proyectos/landing-page-christina]] | Landing page HTML estático para Christina | activo |

---

## Modelos de Negocio (1)

| Página | Descripción | Estado |
| ------ | ----------- | ------ |
| [[modelos-negocio/adifnex-saas]] | Adifnex como SaaS comercial — estrategia, pricing, gaps, pendientes | activo |

---

## Tecnologías (4)

| Página | Descripción |
| ------ | ----------- |
| [[tecnologias/stack-principal]] | Stack completo: Angular + NestJS + PostgreSQL + Tailwind + PrimeNG |
| [[tecnologias/n8n]] | Plataforma de automatización de workflows |
| [[tecnologias/patron-autenticacion-jwt]] | Patrón JWT + Refresh Tokens + RBAC — usado en todos los proyectos |
| [[tecnologias/claude-code-skills]] | Ecosystem de skills para Claude Code: caveman, napkin, remotion-best-practices |
| [[tecnologias/remotion]] | Librería para crear videos con React/TypeScript — skill instalado 2026-05-15 |
| [[tecnologias/railway]] | PaaS Railway — archivado (trial agotado 2026-05-29) |
| [[tecnologias/render]] | PaaS Render — sucesor de Railway, Docker deploy, gotchas free tier cold start |
| [[tecnologias/prompts-gemini-imagenes]] | Prompts validados para generar imágenes fotorrealistas con Gemini — contexto PyME Lima |

---

## Conceptos (0)

*Pendiente. Se crearán al identificar patrones y decisiones recurrentes.*

---

## Personas (1)

| Página | Descripción |
| ------ | ----------- |
| [[personas/christian-jara]] | Propietario — desarrollador fullstack en Perú |

---

## Síntesis (0)

*Pendiente. Se crearán a partir de análisis y preguntas de Christian.*

---

## Notas del wiki

- Total de páginas: 38
- Proyectos con ingestión profunda completa: 2 (venta-inventario, autenticacion-prototipo)
- venta-inventario refactorizado a hub + 6 sub-páginas atómicas (2026-05-07)
- `venta-inventario-frontend.md` actualizado 2026-05-08: regla permisos permanente, vista unificada Secciones+Permisos, bugs `pi-slash` y password autocomplete
- `venta-inventario-rbac.md` actualizado 2026-05-11: nivelMinimo inline, cadena migraciones 008-010, consolidación secciones, guard mi-negocio, script diagnóstico
- `venta-inventario-frontend.md` actualizado 2026-05-11: selector inline nivel + forkJoin bulk, bug double-unwrap servicios inventario
- Proyectos con ingestión pendiente: 9
- `venta-inventario-frontend.md` actualizado 2026-05-30 (sesión 17): guards `guardiaNegocio`, flujo registro 2 pasos frontend, `DatosRegistroNegocio`, componente `negocio-registro`.
- `venta-inventario-testing.md` actualizado 2026-05-12 (ronda 2): 3 bugs raíz resueltos — race condition `esperarCarga()`, `NegocioContextoServicio` in-memory, `waitForResponse` invertido
- `NegocioContextoServicio` ahora persiste a localStorage (mejora UX + tests). `global-setup.ts` pre-carga negocio en storageState.
- `venta-inventario-testing.md` actualizado 2026-05-12 (ronda 3): 6 fixes — tilde `Sí` confirms, race condition inline beforeEach, selector `.spa-sidebar`. 7 tests pendientes de diagnóstico.
- `venta-inventario-inventario.md` actualizado 2026-05-14: MarcasModulo, EtiquetasModulo, tablas `marca`/`etiqueta`/`producto_etiqueta`, endpoints marcas y etiquetas.
- `venta-inventario-frontend.md` actualizado 2026-05-14: componentes marcas-lista y etiquetas-lista, bug `etiquetas undefined` (círculos grises), bug SQL ON CONFLICT → WHERE NOT EXISTS.
- `venta-inventario-testing.md` actualizado 2026-05-14: 3 nuevas suites backend + 5 frontend + 3 specs E2E pendientes (marcas, etiquetas, productos-lista).
- `venta-inventario-frontend.md` actualizado 2026-05-14 (sesión 2): rediseño tabla productos (paginación, sorting, scroll), bugs display:flex en td, mobile paginación, Sequelize bulkCreate ignoreDuplicates, p-multiSelect strings.
- `venta-inventario-testing.md` actualizado 2026-05-15 (ronda 4): fix `productos.page.ts` — locator SKU incorrecto + campo disabled por modo automático. Regla derivada documentada.
- Próxima acción: correr suite productos CRUD para validar los 5 tests. Pendiente: 7 tests ronda 3 + pruebas marcas/etiquetas/productos-lista.
- `venta-inventario-auth.md` actualizado 2026-05-30 (sesión 17): separación registro usuario/negocio, `negocioId` nullable, nuevo endpoint `registrar-negocio`, token verificación no nullificado, campo teléfono, re-registro con token expirado.
- `venta-inventario-auth.md` actualizado 2026-05-30: migración 020 (3 cols verificación), flujo registro autoservicio, endpoints `POST /registro` y `GET /verificar`, patrón bootstrap circular ref.
- `venta-inventario-correo.md` actualizado 2026-05-30: método `enviarVerificacionCorreo`, flujo fire&forget, pendiente reset password endpoint.
- `venta-inventario-testing.md` actualizado 2026-05-30 (sesión 18): +13 backend (autenticacion+correo) + 14 frontend (registro/verificar/negocio-registro/guardia). Totales: ~183 backend / ~200 frontend. Gotchas: RouterLink + provideRouter, transacción Sequelize vía modelo, nodemailer mock, runInInjectionContext para guards.
- `venta-inventario-catalogo.md` creado 2026-05-15: plan catálogo público Next.js, gap analysis, migración 014, 9 permisos Plan 2, 8+1 fases de implementación.
- `venta-inventario-catalogo.md` actualizado 2026-05-17 (sesión 3): F4+F5+F6 completadas. Módulo `publico/` NestJS, tipos TypeScript alineados (THEME pattern), panel Angular "Mi Página" split-pane. Pendientes: F7 galería imágenes, F8 deploy.
- `venta-inventario-catalogo.md` actualizado 2026-05-18 (sesión 4): 2 gotchas documentados — precio null excluye producto del catálogo aunque aparezca "Visible" en panel; NUMERIC de PostgreSQL llega como string a Next.js → usar Number() antes de toFixed.
- `venta-inventario-catalogo.md` actualizado 2026-05-18 (sesión 7): F7 ✅ — modal "Catálogo" unificado (precios + visibilidad + galería imágenes). StorageServicio local/Cloudinary. 4 endpoints imagen. Campos catálogo movidos del edit dialog. Gotchas: Sequelize BelongsTo sin CreationOptional, FormData sin Content-Type manual.
- `plan-catalogo-react.md` + `plan-catalogo-react.html` actualizados 2026-05-18 (sesión 8): F7 marcada ✅, sección 11 "Suite de pruebas" completa — 9 backend unit + 7 frontend service + 11 frontend component + 10 E2E nuevos mapeados con archivos destino exactos.
- `venta-inventario-catalogo.md` actualizado 2026-05-18 (sesión 5): bug contexto negocio en Mi Página — 2 causas (frontend: ngOnInit sin reactivo; backend: req.user.negocioId bypaseaba TenantInterceptor).
- `venta-inventario-catalogo.md` actualizado 2026-05-18 (sesión 6): spotlight granular (IDs hero-titulo/navbar-nombre/tab-favicon etc.), permiso slug (migración 016), PaginaNoDisponible, favicon Next.js gotcha, patrones `w-fit`/`pTooltip`/reset botón.
- `venta-inventario-frontend.md` actualizado 2026-05-18: patrón `toObservable + switchMap` para recarga HTTP reactiva al cambio de contexto de negocio.
- `venta-inventario-rbac.md` actualizado 2026-05-18: regla crítica `@NegocioActual()` vs `req.user.negocioId` — cómo funciona `TenantInterceptor` y qué controladores deben usar el decorador.
- `venta-inventario-catalogo.md` actualizado 2026-05-23 (sesión 11): 2 fixes post-F7 — parpadeo galería (actualización optimista signal) + StorageServicio.eliminarImagen() borra de Cloudinary/disco.
- `venta-inventario-catalogo.md` actualizado 2026-05-19: StorageServicio separado por negocio — Cloudinary `inventario-ventas/{id}/productos`, local `uploads/negocio-{id}/productos/`.
- `venta-inventario-testing.md` actualizado 2026-05-19 (sesión 9): +76 tests — 4 archivos backend nuevos/ampliados, 6 archivos frontend nuevos. E2E: PO mi-pagina.page.ts + 2 specs catálogo. Total ~170 backend / ~186 frontend / ~96 E2E.
- `venta-inventario-catalogo.md` actualizado 2026-05-24 (sesión 12): fix StorageServicio — Cloudinary por defecto en producción (lógica invertida), credenciales Railway, regla `STORAGE_PROVIDER=local` en desarrollo.
- `tecnologias/railway.md` creado 2026-05-24: gotcha variables morado = pendientes sin deploy, variables de entorno requeridas, cómo verificar credenciales Cloudinary.
- `venta-inventario-catalogo.md` actualizado 2026-05-24 (sesión 13): segunda plantilla `catalogo-esencias` — paleta dorada/ivory, Playfair Display, sidebar filtros, imagen 3:4. Campo `plantilla` en NegocioConfig. Router en page.tsx. Pendiente: migración SQL para campo `plantilla` en BD.
- `venta-inventario-auth.md` actualizado 2026-05-31 (sesión 19): flujo "olvidé mi contraseña" — migración 023, 2 nuevos endpoints, patrón anti-enumeración, componentes Angular, validador cross-field, fix URL correo.
- `venta-inventario-correo.md` actualizado 2026-05-31: `enviarRecuperacionContrasena` marcado implementado, URL corregida a `/restablecer-contrasena`, sección flujo reset password.
- `venta-inventario-testing.md` actualizado 2026-05-31 (sesión 20): +5 backend unit (solicitarRecuperacion + restablecerContrasena) + 2 E2E nuevos. Totales: ~188 backend / ~200 frontend. Patrón anti-enumeración documentado.
- `venta-inventario-auth.md` actualizado 2026-05-31 (sesión 21): validación correo verificado en login — 403 con `codigo` tipado, `ExcepcionHttpFiltro` propagación de `codigo`, frontend redirige a `/registro/pendiente` o muestra error inline.
- `venta-inventario-testing.md` actualizado 2026-05-31 (sesión 21): `AutenticacionServicio` ahora 25 tests. +1 E2E spec `correo-no-verificado.spec.ts` (crea usuario no verificado vía API → intenta login → verifica redirect).
- `venta-inventario-frontend.md` actualizado 2026-06-02 (sesión 22): plan migración PrimeNG → Angular Material 18 — tabla equivalencias completa, 7 fases, archivos a crear, notas críticas.
- `venta-inventario-frontend.md` actualizado 2026-06-02 (sesión 23): Fase 0 marcada ✅ — desinstalado PrimeNG, instalado Material 18, 3 archivos creados, angular.json/styles.css actualizados.
- `venta-inventario-testing.md` actualizado 2026-06-02 (sesión 23): +4 tests pendientes para `SnackBarServicio` + `ConfirmDialogComponent`, patrones mock `MatSnackBar` y `MAT_DIALOG_DATA`.
- `venta-inventario-testing.md` actualizado 2026-06-02 (sesión 24): +3 tests pendientes Fase 1 — `entrada.component.spec.ts` (toggle mat-icon, FormControl sin pInputText) + `inicio-sesion.component.spec.ts` (mat-checkbox formControlName).
- `venta-inventario-frontend.md` actualizado 2026-06-02 (sesión 26): Fase 3 marcada ✅ — panel.component y dashboard.component migrados (mat-icon, animate-pulse skeletons, matRipple, Material icon names).
- `venta-inventario-frontend.md` actualizado 2026-06-02 (sesión 27): Fase 4 marcada ✅ — 10 componentes CRUD admin migrados.
- `venta-inventario-frontend.md` actualizado 2026-06-03 (sesión 28): Fase 5 marcada ✅ — 7 componentes inventario migrados; `secciones-permiso-lista` legacy eliminado. Solo `mi-pagina` (Fase 6) pendiente.
- `venta-inventario-testing.md` actualizado 2026-06-03 (sesión 28): sprint Fase 5 — +4 tests pendientes. Total ~216 frontend. Grandes: usuarios-lista (3 p-dialog→modal-overlay), permisos-lista (TS reescrito), mi-negocio-vista (p-tag→Tailwind). 6 componentes solo iconos.
- `venta-inventario-frontend.md` actualizado 2026-06-03 (sesión 29): Fase 6 ✅ — `mi-pagina` migrado. Migración PrimeNG → Angular Material **100% completa** (Fases 0-6).
- `venta-inventario-frontend.md` actualizado 2026-06-03 (sesión 30): 4 gotchas nuevos — Material Icons vs Symbols, CSS scoped vs Tailwind md:hidden, mat-icon hereda font-size, Docker rebuild. UX refactoring productos-lista documentado.
- `venta-inventario-frontend.md` actualizado 2026-06-06 (sesión 43): sidebar v2 promovido — `barra-lateral-v2/` eliminado, código consolidado en `barra-lateral/` (selector `spa-sidebar`). Dark navy v1 eliminado definitivamente.
- `venta-inventario-frontend.md` actualizado 2026-06-06 (sesión 42): naranja `#EA580C` activado (logo+avatar sidebar, CTA btn, summary badge); bug fix dropdown perfil (`stopPropagation` faltante con `@HostListener('document:click')`).
- `venta-inventario-frontend.md` actualizado 2026-06-06 (sesión 41): `barra-lateral-v2` — diseño white/green en paralelo, workspace switcher custom (no mat-select), gotcha dropdown en sidebars angostos.
- `venta-inventario-frontend.md` actualizado 2026-06-06 (sesión 39): regla estándar tablas full-height — patrón 3 pasos (view-VISTA class + CSS flex/overflow + :host flex:1), cadena flex documentada, negative margins marcados como inaplicables para tablas.
- `venta-inventario-frontend.md` actualizado 2026-06-06 (sesión 37): productos-premium sesión de refinamiento — modal crear nuevo producto (signals+Subject debounce SKU), eliminar con ConfirmDialog, toasts en 4 métodos, fix z-index cdk-overlay-container (10000), "Guardar cambios" condicional por tab, click fuera no cierra modal. Preview estirada 380px/170px.
- `venta-inventario-frontend.md` actualizado 2026-06-06 (sesión 36): 3 mejoras UX mobile modal productos-premium — (1) preview como bottom sheet, (2) modal altura fija `height: min(88vh,720px)`, (3) formulario "Registrar movimiento" como bottom sheet con auto-cierre. Causa raíz scroll horizontal documentada (doble padding). 2 reglas nuevas.
- `venta-inventario-frontend.md` actualizado 2026-06-04 (sesión 31): mejoras UI productos-lista (gradiente, pill, row hover accent, action buttons tipados, modal border-top); componente `productos-premium` desde cero (design system indigo/violet, KPI bar, gradiente tabla header, pulse-dot, avatares dinámicos, DM Sans); trick negative margins :host. Fase 6 migración corregida a ✅.
- `venta-inventario-testing.md` actualizado 2026-06-03 (sesión 29): sprint Fase 6 — +5 tests pendientes para `mi-pagina.component.spec.ts`. Total ~221 frontend estimado.

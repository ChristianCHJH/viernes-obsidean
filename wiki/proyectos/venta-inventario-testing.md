---
titulo: Venta e Inventario — Estrategia de Pruebas
tipo: proyecto
tags: [testing, jest, playwright, nestjs, angular, unit-tests, venta-inventario]
fecha_creacion: 2026-05-12
fecha_actualizacion: 2026-06-06 (sesión 38)
estado: activo
---

# Venta e Inventario — Estrategia de Pruebas

Sub-página del hub [[venta-inventario]]. Documenta la estrategia, decisiones y estado de implementación del sistema de pruebas automatizadas.

---

## Contexto

Hasta 2026-05-12 el proyecto no tenía ningún test. Al modificar un servicio, errores en otros módulos se detectaban tarde (en producción). El script `test` en `backend/package.json` era un `echo` vacío.

---

## Estrategia adoptada — 3 fases

```
Pirámide de pruebas:
         /    E2E (Playwright)    \     ← Fase 3 — flujos críticos
        / ─────────────────────── \
       /  Integración (Jest)       \    ← Fase 2 (futuro)
      / ─────────────────────────── \
     /   Unitarias (Jest)            \  ← Fase 1 ✅ COMPLETADA
    / ─────────────────────────────── \
```

### Fase 1 — Unit tests backend (COMPLETADA 2026-05-12)

- **Tool**: Jest + ts-jest + @nestjs/testing
- **Scope**: todos los servicios NestJS (15 servicios)
- **Total**: 114 tests, 15 suites — 100% verde
- **Comando**: `npm test` | `npm run test:cov` (desde `backend/`)

### Fase 2 — Unit tests frontend (COMPLETADA 2026-05-11)

- **Tool**: Jest + jest-preset-angular 16 (migración desde Karma)
- **Scope**: 14 servicios Angular — 2 de estado local + 12 HTTP
- **Total**: 110 tests, 15 suites — 100% verde, 4.8s
- **Comando**: `npm test` | `npm run test:cov` (desde `frontend/`)
- **No incluye**: componentes visuales, templates HTML

### Fase 3 — E2E Playwright (IMPLEMENTADA 2026-05-12 — ajuste en curso)

- **Tool**: Playwright 1.60, Chromium
- **Estructura**: `e2e/` en raíz del monorepo (package.json independiente)
- **Total**: 19 specs, ~71 tests
- **Comando**: `npm test` | `npm run test:headed` | `npm run test:ui` (desde `e2e/`)
- **Estado**: infraestructura creada, 13 pasados, 51 fallando por selectores PrimeNG — ajuste en curso

---

## Configuración backend (Fase 1)

### Dependencias instaladas

```bash
npm install --save-dev jest @types/jest @nestjs/testing ts-jest --legacy-peer-deps
```

> Se requiere `--legacy-peer-deps` por conflicto entre `@nestjs/testing@^11` y `@nestjs/common@^10`.

### jest.config.ts

```typescript
import type { Config } from 'jest';

const config: Config = {
  moduleFileExtensions: ['js', 'json', 'ts'],
  rootDir: 'src',
  testRegex: '.*\\.spec\\.ts$',
  transform: { '^.+\\.(t|j)s$': ['ts-jest', { diagnostics: false }] },
  collectCoverageFrom: ['**/*.(t|j)s'],
  coverageDirectory: '../coverage',
  testEnvironment: 'node',
};
export default config;
```

> `diagnostics: false` deshabilita el type-checking de ts-jest. Necesario porque algunos servicios del proyecto tienen errores de TypeScript strict (ej. Sequelize `null` en campos opcionales) que bloquearían la ejecución de los tests aunque el runtime funcione.

### Scripts en package.json

```json
"test": "jest",
"test:watch": "jest --watch",
"test:cov": "jest --coverage"
```

---

## Patrón de mocks — Sequelize NestJS

### Modelos (@InjectModel)

```typescript
import { getModelToken } from '@nestjs/sequelize';

const mockModelo = {
  findOne: jest.fn(),
  findAll: jest.fn(),
  findByPk: jest.fn(),
  create: jest.fn(),
};

// En el TestingModule:
{ provide: getModelToken(MiEntidad), useValue: mockModelo }
```

### Conexión (@InjectConnection — para transacciones)

```typescript
import { getConnectionToken } from '@nestjs/sequelize';

const mockSequelize = {
  transaction: jest.fn().mockImplementation((cb) => cb({})),
  query: jest.fn(),
};

{ provide: getConnectionToken(), useValue: mockSequelize }
```

### Patrón de objeto fake (instancia)

```typescript
const entidadFake = (overrides = {}) => ({
  id: 1,
  campo: 'valor',
  eliminado: false,
  update: jest.fn().mockResolvedValue(undefined),
  save: jest.fn().mockResolvedValue(undefined),
  ...overrides,
});
```

---

## Lección aprendida — `update()` vs `save()`

El patrón más común en los servicios de este proyecto **NO es** `objeto.update({ campo: valor })`. Es:

```typescript
// Patrón real en la mayoría de servicios:
objeto.campo = valor;
return objeto.save();
```

Esto significa que los asserts correctos son:

```typescript
// ✅ Correcto
expect(objeto.campo).toBe(valor);
expect(objeto.save).toHaveBeenCalled();

// ❌ Incorrecto (update no se llama en estos casos)
expect(objeto.update).toHaveBeenCalledWith({ campo: valor });
```

**Servicios que SÍ usan `.update()`**: NegocioServicio, SeccionesPermisoServicio (actualizar), NegocioPermisosServicio (cambiarEstado), ProductosServicio.

---

## Estado de specs por servicio (Fase 1)

| Servicio | Tests | Método más complejo cubierto |
|---|---|---|
| AutenticacionServicio | 25 | `obtenerPermisosEfectivos`, correo no verificado (2 códigos) |
| RefreshTokensServicio | 6 | `validarVigencia` (expirado, revocado) |
| LogSesionesServicio | 5 | `obtenerPorUsuario` (límite default) |
| UsuariosServicio | 8 | `crear` (bcrypt + validarCorreoUnico) |
| RolesServicio | 7 | `listarNoAsignadosPorUsuario` |
| PermisosServicio | 7 | CRUD + soft delete |
| NegocioPermisosServicio | 7 | `asignar` (conflict), `revocar` |
| RolesPermisosServicio | 5 | `asignarPermisosARol` (upsert) |
| UsuariosRolesServicio | 7 | `vincularRol` (crea o reactiva) |
| UsuariosPermisosServicio | 6 | `vincularPermisos` (insertado/activado) |
| SeccionesPermisoServicio | 8 | `listarConPermisosPorUsuario` |
| StockServicio | 11 | `registrarMovimiento` (INGRESO/SALIDA + tx) |
| ProductosServicio | 9 | `crear` (SKU duplicado), `actualizar` |
| SedesServicio | 8 | CRUD + soft delete |
| NegocioServicio | 9 | `obtenerMiNegocio` (usa `findOne` no `findByPk`) |
| **Total** | **114** | |

---

## Estado de specs por servicio (Fase 2 — Frontend)

| Servicio | Tests | Tipo |
|---|---|---|
| NegocioContextoServicio | 8 | Estado local (signals) |
| PermisosUsuarioServicio | 13 | Estado local (signals) |
| NegocioServicio | 9 | HTTP |
| NegocioPermisosServicio | 6 | HTTP |
| PermisosServicio | 5 | HTTP |
| RolesServicio | 6 | HTTP |
| RolesPermisosServicio | 5 | HTTP |
| SeccionesPermisoServicio | 6 | HTTP |
| UsuariosServicio | 8 | HTTP |
| UsuariosPermisosServicio | 6 | HTTP |
| UsuariosRolesServicio | 6 | HTTP |
| ProductosInventarioServicio | 8 | HTTP |
| SedesInventarioServicio | 5 | HTTP |
| StockInventarioServicio | 9 | HTTP |
| **Total** | **110** | |

### Configuración frontend (Fase 2)

- `jest.config.ts` + `setup-jest.ts` creados en raíz de `frontend/`
- `tsconfig.spec.json` actualizado: `types: ["jest"]` (sin jasmine)
- Karma/Jasmine desplazado — `npm test` ahora corre Jest

### Nota branch coverage

Servicios con branch coverage baja (~25-50%): `roles.servicio.ts`, `secciones-permiso.servicio.ts`, `usuarios-roles.servicio.ts`. Las ramas sin cubrir son variantes alternativas del envelope (`data`, `roles`, `items`) que el backend nunca emite en producción.

---

## Configuración E2E (Fase 3)

### Estructura e2e/

```text
e2e/
├── package.json                 ← proyecto independiente
├── playwright.config.ts         ← 2 proyectos: autenticacion (sin storageState) + app (con storageState)
├── global-setup.ts              ← POST /api/autenticacion/login → guarda auth-state.json en localStorage
├── fixtures/
│   ├── datos-prueba.json        ← credenciales dev, datos de prueba con timestamp
│   └── auth-state.json          ← generado por global-setup (no commitear)
├── pages/                       ← Page Object Model (10 clases)
└── tests/
    ├── 01-autenticacion/        ← sin storageState (prueban login desde cero)
    ├── 02-usuarios/
    ├── 03-roles/
    ├── 04-productos/
    ├── 05-sedes/
    ├── 06-stock/
    ├── 07-negocios/
    ├── 08-secciones-permisos/
    ├── 09-dashboard/
    └── 10-acceso-denegado/
```

### Credenciales dev

```json
{ "correo": "admin@museo.local", "contrasena": "admin123" }
```

### URLs

- Frontend: `http://localhost:4200`
- Backend: `http://localhost:3200` (prefijo `/api`)
- Docker expone ambos al host — Playwright corre en la máquina, no dentro de Docker

### Cómo funciona el storageState

`global-setup.ts` llama directamente al backend REST, obtiene el JWT y lo guarda en `fixtures/auth-state.json` como `localStorage['spa.auth.tokens']` con la estructura:

```json
{
  "tokenAcceso": "...",
  "tokenRefresco": "...",
  "tipoToken": "Bearer",
  "expiraEn": "..."
}
```

Playwright inyecta este localStorage antes de que Angular cargue. El guard `guardiaAutenticacion` llama `estaAutenticado()` que solo lee `localStorage['spa.auth.tokens'].tokenAcceso` — no valida expiración ni hace API call.

### Bug crítico resuelto — envelope `datos` vs `data`

El backend retorna `{ exito, datos }` (no `{ data }`). El `global-setup.ts` original usaba `cuerpo.data ?? cuerpo` — siempre caía al fallback `cuerpo` completo, y `cuerpo.tokenAcceso` es undefined. El token nunca se guardaba. Todos los tests del proyecto `app` se quedaban en la pantalla de login.

**Fix**: `cuerpo.datos ?? cuerpo.data ?? cuerpo`

---

## Bugs resueltos — sesión 2026-05-12 (ronda 2)

Estado antes: 39 pasando / 25 fallando. Causa raíz: 3 bugs distintos.

### Bug 1 — Race condition en `esperarCarga()`

`waitForFunction(!document.querySelector('.animate-pulse'))` resolvía inmediatamente si Angular aún no había montado nada (DOM vacío = sin `.animate-pulse`). Los tests leían la pantalla antes de que el componente renderizara.

**Fix en todos los page objects**:

```typescript
async esperarCarga(): Promise<void> {
  await this.page.waitForSelector('.page-title', { timeout: 15000 }); // confirma Angular montado
  await this.page.waitForFunction(
    () => !document.querySelector('.animate-pulse'),
    { timeout: 15000 }
  );
}
```

`.page-title` está en el header del panel y siempre existe una vez que Angular inicializa el `PanelComponent`.

### Bug 2 — `NegocioContextoServicio` in-memory: productos siempre en `sinContexto()`

`NegocioContextoServicio._negocioActivo` era un `signal(null)` puro — no persistía entre sesiones. Cada test iniciaba con contexto nulo. Resultado:

- `ProductosListaComponent.sinContexto()` = true → mostraba "Selecciona un negocio" en vez de la tabla
- El interceptor no enviaba el header `X-Negocio-Context` → crear producto fallaba en el backend

**Fix doble**:

1. `NegocioContextoServicio` ahora persiste a `localStorage['spa.negocio.contexto']`:

```typescript
private readonly _negocioActivo = signal<NegocioContexto | null>(this.leerDeStorage());
// establecer() → escribe a localStorage; limpiar() → borra de localStorage
```

2. `global-setup.ts` ahora busca el negocio del usuario tras login:

```typescript
// Intenta GET /api/negocio/mi-negocio con el JWT
// Fallback: GET /api/negocio → primer negocio activo
// Guarda: localStorage['spa.negocio.contexto'] = { id, nombre }
```

**Efecto colateral positivo**: en producción, el usuario ya no pierde el contexto de negocio al refrescar la página (mejora de UX real).

### Bug 3 — `waitForResponse` bloqueante antes de la acción

En `02-usuarios/usuarios-crud.spec.ts`:

```typescript
// ❌ MALO — bloquea 10s (timeout) antes de llamar crearUsuario
await page.waitForResponse(..., { timeout: 10000 }).catch(() => null);
await usuariosPage.crearUsuario(nombre, correo, 'Test1234!');

// ✅ CORRECTO — listener activo, acción, luego esperar resultado
const respuesta = page.waitForResponse(...);
await usuariosPage.crearUsuario(nombre, correo, 'Test1234!');
await respuesta.catch(() => null);
```

Causaba que "crear usuario" tardara 16s (timeout completo antes de empezar).

---

## Bugs resueltos — sesión 2026-05-12 (ronda 3)

Estado antes: 58 pasando / 13 fallando. 6 fixes aplicados. 7 aún pendientes de diagnóstico con errores reales.

### Bug 1 — `acceptLabel` sin tilde: `'Si'` vs `'Sí'`

Los confirm dialogs de usuarios, roles y negocios usaban `acceptLabel: 'Si, activar'` / `'Si, desactivar'` (sin tilde). Los tests buscan `button:has-text("Sí")` (con tilde). Playwright hace match exacto de texto — no matcheaba.

**Archivos corregidos**:
- `usuarios-lista.component.ts`: `'Si, activar'` → `'Sí, activar'`, `'Si, desactivar'` → `'Sí, desactivar'`
- `roles-lista.component.ts`: mismo fix
- `negocios-lista.component.ts`: mismo fix

**Tests esperados**: `usuarios-crud:63`, `usuarios-crud:90`, `roles-crud:53`, `negocios-crud:50`

> Nota: `sedes-lista.component.ts` ya tenía tilde correcta (`'Sí, eliminar'`), no requirió fix.

### Bug 2 — Race condition en `beforeEach` inline de specs sin page object

`secciones-permisos.spec.ts` y `dashboard.spec.ts` usaban `beforeEach` directo con `waitForFunction(!animate-pulse)` sin el `waitForSelector('.page-title')` previo. El fix de ronda 2 se aplicó solo a los page objects — estos specs inline quedaron sin corregir.

**Fix aplicado**:
```typescript
// Antes (buggy):
await page.waitForFunction(() => !document.querySelector('.animate-pulse'), { timeout: 15000 });

// Después:
await page.waitForSelector('.page-title', { timeout: 15000 });
await page.waitForFunction(() => !document.querySelector('.animate-pulse'), { timeout: 15000 });
```

`dashboard.spec.ts` también usaba `waitForLoadState('networkidle')` — reemplazado por el patrón correcto (networkidle puede nunca resolver si hay polling).

**Tests esperados**: `secciones-permisos:48`, `dashboard:14`

### Bug 3 — Selector `spa-sidebar` (host element) vs `.spa-sidebar` (inner div)

`dashboard.spec.ts:14` buscaba el host element de Angular `<spa-sidebar>`. En Angular con Emulated ViewEncapsulation, el host element puede tener dimensiones inconsistentes en Playwright. El inner div tiene class `.spa-sidebar` con CSS real.

**Fix**: `page.locator('spa-sidebar')` → `page.locator('.spa-sidebar')`

### Tests pendientes de diagnóstico (7)

Requieren ejecutar Playwright con output verbose para ver el error exacto:

| Test | Duración | Hipótesis |
|---|---|---|
| `usuarios-crud:38` editar usuario | 14.4s | `crearUsuario` no cierra dialog (validación falla) o timing en reload de lista |
| `usuarios-permisos:36` reset password | 30.1s (timeout global) | `btnReset.click()` cuelga — posible elemento cubierto por otro |
| `sedes-crud:56` soft-delete | 4.2s | Confirm funciona (tilde OK) pero lista no se recarga o ruta no filtra eliminados |
| `movimientos:13` INGRESO | 15.9s | Dropdown PrimeNG no interactúa correctamente con `p-dropdown[formcontrolname]` |
| `movimientos:54` SALIDA | 15.3s | Mismo |
| `carga-inicial:11` | 18.1s | Navegación sede-stock o selectores del form en esa vista |
| `transferencia:6` | 10.5s | Mismo + `.p-selectbutton TRANSFERENCIA` selector |

**Hallazgos técnicos útiles (para diagnóstico futuro)**:

- `PermisosUsuarioServicio` es in-memory puro (signal, no localStorage) — se inicializa en `PanelComponent.ngOnInit()` leyendo el JWT payload. `PermisoDirectiva` usa `effect()` reactivo.
- Template `usuarios-lista` tiene 3 `<p-dialog>`: historial/negocio, formulario principal, y confirmDialog. Al buscar `.p-dialog` puede dar strict mode violation si múltiples están visibles.
- La acción `click()` en Playwright tiene timeout = test global timeout (30s). Si cuelga 30s, el elemento existe (isVisible=true) pero no es clickable (cubierto, pointer-events, o animando).

---

## Qué cubre E2E (alcance correcto)

E2E = flujos que un QA humano haría en staging. No es exhaustivo — es la red de seguridad de los flujos críticos.

| Flujo | Ejemplo |
|---|---|
| Crear registro | Agregar sección → aparece en lista |
| Editar/actualizar | Cambiar nombre → se guarda |
| Habilitar/deshabilitar | Toggle estado → refleja en tabla |
| Eliminar (soft delete) | Eliminar → desaparece de lista activa |
| Búsqueda/filtros | Buscar producto → tabla filtra |
| Movimiento de stock | Registrar ingreso → stock actualizado |

**Lo que NO entra en E2E**: casos edge raros, toda combinación de datos posible, lógica de negocio pura (ya cubierta en unit tests).

---

## Pruebas pendientes — módulos Marcas, Etiquetas y fix Productos (2026-05-14)

Generadas al implementar marcas, etiquetas y corregir el bug de `etiquetas undefined` en la lista de productos.

### Unit backend

| Suite | Tests | Archivo |
|---|---|---|
| `productos.servicio.spec.ts` | `listar()` con etiquetas incluidas + marcaNombre; crear() SKU auto; conflicto SKU; soft-delete | `backend/src/inventario/productos/productos.servicio.spec.ts` |
| `marcas.servicio.spec.ts` | listar() filtra negocioId; crear() ConflictException; actualizar(); eliminar() soft-delete | `backend/src/inventario/marcas/marcas.servicio.spec.ts` |
| `etiquetas.servicio.spec.ts` | listar(); crear() ConflictException; actualizar() nombre+color; eliminar() soft-delete | `backend/src/inventario/etiquetas/etiquetas.servicio.spec.ts` |

### Unit frontend

| Suite | Tests | Archivo |
|---|---|---|
| `marcas-inventario.servicio.spec.ts` | GET listar; POST crear; PATCH actualizar; DELETE eliminar | `frontend/src/app/core/services/marcas-inventario.servicio.spec.ts` |
| `etiquetas-inventario.servicio.spec.ts` | GET listar; POST crear; PATCH actualizar; DELETE eliminar | `frontend/src/app/core/services/etiquetas-inventario.servicio.spec.ts` |
| `marcas-lista.component.spec.ts` | render tabla; abrir dialog; editar; confirmar eliminar; error HTTP toast | `frontend/src/app/features/inventario/marcas-lista/marcas-lista.component.spec.ts` |
| `etiquetas-lista.component.spec.ts` | render con color dot; picker actualiza campo; crear; eliminar; error toast | `frontend/src/app/features/inventario/etiquetas-lista/etiquetas-lista.component.spec.ts` |
| `productos-lista.component.spec.ts` | render todos los productos con `etiquetas=undefined`; chips con color; `—` sin etiquetas; marcaNombre visible | `frontend/src/app/features/inventario/productos-lista/productos-lista.component.spec.ts` |

### E2E Playwright

| Spec | Flujo | Archivo |
|---|---|---|
| `06-marcas.spec.ts` | crear → ver en lista → editar → eliminar; sin permiso → no aparece en sidebar | `e2e/tests/06-marcas.spec.ts` |
| `07-etiquetas.spec.ts` | crear con color → chip visible → editar → eliminar; sin permiso → no aparece | `e2e/tests/07-etiquetas.spec.ts` |
| `08-productos-lista.spec.ts` | crear producto con etiqueta → lista muestra chip y marcaNombre | `e2e/tests/08-productos-lista.spec.ts` |

---

## Bugs resueltos — sesión 2026-05-15 (ronda 4 — productos.page.ts)

Causa raíz: la tabla de productos fue rediseñada (columnas nuevas, SKU fusionado con nombre, SKU auto-generado por defecto) pero el page object no se actualizó.

### Bug 1 — Locator `inputSku` con placeholder inexistente

El page object buscaba `input[placeholder="Ej: ACE-LAV-001"]`. Tras el rediseño del formulario, el campo SKU tiene `placeholder="Se generará automáticamente"` siempre. El locator nunca matcheaba → timeout en todos los tests que llaman `crearProducto`.

**Fix**: `page.locator('input[placeholder="Ej: ACE-LAV-001"]')` → `page.locator('.p-dialog input.sku-field')`

### Bug 2 — Campo SKU `disabled` por defecto (modo automático)

El formulario tiene un toggle `skuAutomatico = signal(true)`. Al abrir el dialog, el componente llama `formulario.controls.sku.disable()`. Aunque el locator funcionara, `fill()` en un input disabled no tiene efecto en Playwright.

El checkbox "Ingresar manualmente" habilita el campo manualmente — el page object no lo consideraba.

**Fix en `crearProducto()`**:

```typescript
if (sku) {
  // habilitar modo manual primero
  const chkManual = this.page.locator('.p-dialog .sku-toggle input[type="checkbox"]');
  await chkManual.check();
  await this.inputSku.waitFor({ state: 'visible' });
  await this.inputSku.fill(sku);
}
```

El parámetro `sku` también se volvió opcional (`sku?: string`). Si no se pasa SKU, el sistema genera uno automáticamente y los tests de búsqueda por SKU no aplican.

### Regla derivada

> Cuando se rediseña un formulario con campos opcionales/condicionales (disabled/enabled por toggle), revisar todos los page objects que interactúan con esos campos. Un input disabled no lanza error al ser llenado — falla silenciosamente o con timeout.

---

## Sprint 2026-05-19 — Suite completa (catálogo + imágenes + componentes)

Implementación masiva de todos los tests pendientes del backlog del catálogo. Sin regresiones en tests pre-existentes.

### Nuevos tests backend (+29)

| Archivo | Tests | Qué cubre |
|---|---|---|
| `storage.servicio.spec.ts` (nuevo) | 4 | `subirImagen(file, negocioId)` — Cloudinary folder por negocio, URL local por negocio, error Cloudinary |
| `publico.servicio.spec.ts` (nuevo) | 8 | `obtenerConfig`, `obtenerProductos`, `obtenerCategorias` — NotFoundException + filtros visibilidad |
| `productos.servicio.spec.ts` (ampliado) | +5 | Suite `Imágenes de producto`: listarImagenes, agregarImagen (reset esPrincipal), actualizarImagen, eliminarImagen soft-delete |
| `negocio.servicio.spec.ts` (ampliado) | +3 | Suite `PaginaConfig`: lazy-create con slug default, retornar existente, actualizar campos |

**Total backend acumulado: ~170 tests** (141 base + 29 nuevos)

### Nuevos tests frontend (+47)

| Archivo | Tests | Qué cubre |
|---|---|---|
| `mi-pagina-catalogo.servicio.spec.ts` (nuevo) | 5 | GET obtener, PATCH actualizar, propagación de errores 403/401 |
| `producto-imagenes.servicio.spec.ts` (nuevo) | 6 | listar, subir (FormData + esPrincipal), actualizar, eliminar, error 500 |
| `marcas-lista.component.spec.ts` (nuevo) | 7 | Render tabla, abrirCrear, abrirEditar con datos, guardar editar recarga, error toast |
| `etiquetas-lista.component.spec.ts` (nuevo) | 8 | Render color dot, seleccionarColor actualiza signal, crear/editar, error toast |
| `productos-lista.component.spec.ts` (nuevo) | 7 | Render con etiquetas=undefined, chips con color, `—` sin etiquetas, marcaNombre |
| `catalogo-producto-dialog.component.spec.ts` (nuevo) | 14 | ngOnChanges carga imágenes, guardarPrecios emite guardado, subirImagen con/sin archivo, eliminarImagen, onCerrar |

**Total frontend acumulado: ~186 tests** (~139 base + 47 nuevos)

### Nuevos E2E (requieren backend + frontend corriendo)

| Archivo | Tests | Qué cubre |
|---|---|---|
| `e2e/pages/mi-pagina.page.ts` (nuevo PO) | — | Locators: tabs, nombrePublico, toggle activo, guardar |
| `e2e/tests/13-catalogo/catalogo-mi-pagina.spec.ts` | 5 | Formulario con 4 tabs, editar nombrePublico, activar catálogo |
| `e2e/tests/13-catalogo/catalogo-imagenes.spec.ts` | 4 | Botón pi-globe en tabla, dialog abre, campos precio/visibilidad |
| `e2e/pages/productos.page.ts` (ampliado) | — | +`abrirModalCatalogo(sku)`, `cerrarModalCatalogo()` |

**Total E2E acumulado: ~96 tests** (87 base + ~9 nuevos)

### Regresiones pre-existentes (NO causadas por estos cambios)

`sedes.servicio.spec.ts` y `usuarios.servicio.spec.ts` fallan con `NullInjectorError: NegocioRepository`. Verificado con git stash — existían antes. Pendiente de diagnóstico.

### Nota — StorageServicio cambio de firma (2026-05-19)

`subirImagen` ahora recibe `negocioId: number` como segundo parámetro. El controlador `multerStorage` lee `req['negocioId']` para crear la carpeta dinámica. Si se agrega otro controlador que use StorageServicio, debe pasar negocioId.

---

## Sprint 2026-05-30 — Auth registro/verificación/negocio + correo

Tests para los nuevos flujos de registro público autoservicio, verificación de correo y registro en dos pasos (usuario → negocio).

### Backend (+13 tests)

| Archivo | Tests nuevos | Qué cubre |
|---|---|---|
| `autenticacion.servicio.spec.ts` (ampliado) | +11 | `registrar()` ×4, `verificarCorreo()` ×4, `registrarNegocio()` ×3 |
| `correo.servicio.spec.ts` (nuevo) | 2 | `enviarVerificacionCorreo()` — sendMail llamado + fire&forget silencioso |

**Total backend acumulado: ~183 tests**

#### Detalle `registrar()` (4 tests)

- Correo ya verificado → ConflictException
- Correo sin verificar → refresca token, reenvía correo (flujo re-registro)
- NombreUsuario ya en uso → ConflictException
- Happy path → crea usuario en transacción, asigna rol Administrador, envía correo fire&forget, retorna `{ verificacionPendiente: true }`

#### Detalle `verificarCorreo()` (4 tests)

- Token no encontrado → BadRequestException
- Cuenta ya verificada → BadRequestException (mensaje "ya fue verificada")
- Token expirado → BadRequestException (mensaje "expirado")
- Happy path → `update({ correoVerificado: true })` + retorna tokens

#### Detalle `registrarNegocio()` (3 tests)

- Usuario no encontrado → BadRequestException
- Usuario ya tiene negocio (`negocioId != null`) → ConflictException
- Happy path → crea negocio en transacción, vincula usuario, retorna tokens actualizados

### Patrón mock — transacciones Sequelize vía modelo

Los métodos `registrar()` y `registrarNegocio()` usan `this.usuarioModelo.sequelize!.transaction()`. El mock va directamente en el objeto del modelo:

```typescript
const mockTransaction = {
  commit: jest.fn().mockResolvedValue(undefined),
  rollback: jest.fn().mockResolvedValue(undefined),
};

const mockUsuarioModelo = {
  findOne: jest.fn(),
  create: jest.fn(),
  sequelize: { transaction: jest.fn().mockResolvedValue(mockTransaction) },
};
```

> Diferente al patrón anterior con `getConnectionToken()` — aquí la transacción sale del modelo, no de la conexión inyectada.

### Patrón mock — nodemailer

```typescript
jest.mock('nodemailer');
// En beforeEach, ANTES de compilar el módulo:
const sendMailMock = jest.fn().mockResolvedValue({ messageId: 'ok' });
(nodemailer.createTransport as jest.Mock).mockReturnValue({ sendMail: sendMailMock });
```

El transporter se crea en el constructor de `CorreoServicio`, por eso el mock debe aplicarse antes de `Test.createTestingModule().compile()`.

### Frontend (+14 tests)

| Archivo | Tests | Qué cubre |
|---|---|---|
| `registro.component.spec.ts` (nuevo) | 5 | Form inválido no llama; payload correcto; signal `enviando`; navega a `/registro/pendiente`; error HTTP → `errorGeneral` |
| `verificar-correo.component.spec.ts` (nuevo) | 4 | Token en queryParam → llama verificarCorreo; éxito → navega dashboard; error genérico → estado `'error'`; "ya verificada" → estado `'ya-verificado'` |
| `negocio-registro.component.spec.ts` (nuevo) | 3 | Submit → llama registrarNegocio; éxito → inicializarPermisos + navega; 409 → mensaje "ya tiene un negocio" |
| `negocio.guardia.spec.ts` (nuevo) | 2 | `negocioId != null` → true; `negocioId == null` → redirige a `/negocio/nuevo` |

**Total frontend acumulado: ~200 tests**

### Gotcha — RouterLink en componentes standalone requiere `provideRouter([])`

`RegistroComponent` y `VerificarCorreoComponent` importan `RouterLink` directamente en sus `imports: [...]`. Al hacer `imports: [ComponenteX]` en TestBed, Angular instancia `RouterLink`, que en su constructor suscribe a `Router.events`. Un mock simple de Router no tiene `events` Observable → TypeError.

```typescript
// ❌ FALLA — Router mock sin events
{ provide: Router, useValue: { navigate: jest.fn() } }

// ✅ CORRECTO — provideRouter([]) + spy sobre el router real
provideRouter([]),
// después de compileComponents():
const router = TestBed.inject(Router);
jest.spyOn(router, 'navigate').mockResolvedValue(true);
```

`NO_ERRORS_SCHEMA` **no ayuda aquí** porque el componente importa `RouterLink` explícitamente, no solo lo usa en el template.

### Patrón guard funcional con `TestBed.runInInjectionContext`

```typescript
TestBed.configureTestingModule({
  providers: [
    { provide: AutenticacionServicio, useValue: mockServicio },
    { provide: Router, useValue: mockRouter },
  ],
});
const result = TestBed.runInInjectionContext(() =>
  guardiaNegocio({} as ActivatedRouteSnapshot, {} as RouterStateSnapshot)
);
```

`runInInjectionContext` permite llamar guards funcionales (que usan `inject()`) sin necesidad de un módulo completo.

---

## Sprint 2026-05-31 — Tests recuperación de contraseña

Tests para el flujo "Olvidé mi contraseña" — `solicitarRecuperacion` y `restablecerContrasena`.

### Backend (+5 tests en `autenticacion.servicio.spec.ts`)

| Método | Tests | Casos |
|---|---|---|
| `solicitarRecuperacion()` | 2 | Correo existe → guarda token + llama correoServicio; correo no existe → retorna OK silencioso |
| `restablecerContrasena()` | 3 | Token válido → actualiza hash + limpia token; token expirado → BadRequestException; token inexistente → BadRequestException |

**Total backend acumulado: ~188 tests** (23 en autenticacion suite — 100% verde)

Cambio menor al spec existente: `mockCorreoServicio` ampliado con `enviarRecuperacionContrasena: jest.fn()`.

### E2E Playwright (+2 specs en `e2e/tests/01-autenticacion/recuperar-contrasena.spec.ts`)

| Test | Flujo |
|---|---|
| Flujo completo | Login → click "Olvide mi contraseña" → formulario email → enviar → aparece `<h1>Revisa tu correo</h1>` |
| Sin token → redirect | `GET /restablecer-contrasena` (sin `?token=`) → Angular `ngOnInit` detecta ausencia de token → navega a `/recuperar-contrasena` |

### Patrón anti-enumeración en tests

`solicitarRecuperacion()` siempre devuelve 200 con el mismo mensaje, exista o no el correo. En el test:
- Caso correo existe: `findOne` retorna usuario → `update` llamado + `enviarRecuperacionContrasena` llamado
- Caso correo no existe: `findOne` retorna `null` → ni `update` ni `enviarRecuperacionContrasena` se llaman — pero el resultado sigue teniendo `mensaje`

Esto verifica que no hay fuga de información ("timing-safe" a nivel de respuesta HTTP).

---

## Sprint 2026-06-02 — Tests Fase 0 migración Angular Material

Tests derivados de la Fase 0: `SnackBarServicio` y `ConfirmDialogComponent`.

### Frontend unit (+4 tests pendientes)

| Archivo | Tests | Qué cubre |
|---|---|---|
| `snack-bar.servicio.spec.ts` (nuevo) | 2 | `exito()` → `MatSnackBar.open()` con `panelClass: ['snack-exito']`; `error()` usa duración 5000ms por defecto |
| `confirm-dialog.component.spec.ts` (nuevo) | 2 | Clic "Confirmar" → `dialogRef.close(true)`; clic "Cancelar" → `dialogRef.close(false)` |

#### Patrón mock para SnackBarServicio

```typescript
const snackBarSpy = { open: jest.fn() };

TestBed.configureTestingModule({
  providers: [
    SnackBarServicio,
    { provide: MatSnackBar, useValue: snackBarSpy },
  ],
});
```

#### Patrón mock para ConfirmDialogComponent

```typescript
const dialogRefSpy = { close: jest.fn() };

TestBed.configureTestingModule({
  imports: [ConfirmDialogComponent],
  providers: [
    { provide: MAT_DIALOG_DATA, useValue: { titulo: 'T', mensaje: 'M' } },
    { provide: MatDialogRef, useValue: dialogRefSpy },
  ],
});
```

**Total frontend acumulado estimado: ~204 tests** (pending — no ejecutados aún)

---

## Sprint 2026-06-02 — Tests Fase 1 migración Angular Material (autenticación)

Tests derivados de la migración de los 6 componentes de autenticación + `spa-input` + `spa-button`.

### Frontend unit (+3 tests pendientes)

| Archivo | Tests | Qué cubre |
|---|---|---|
| `entrada.component.spec.ts` (nuevo o existente) | 2 | Toggle contraseña: icono `visibility_off` cuando `contrasenaVisible=true`; input funcional con `FormControl` sin `pInputText` |
| `inicio-sesion.component.spec.ts` (nuevo o existente) | 1 | `mat-checkbox` bindea `formControlName="recordarme"` — cambiar checkbox actualiza valor en el form |

**Total frontend acumulado estimado: ~207 tests** (pending — no ejecutados aún)

---

## Sprint 2026-06-02 — Tests Fase 2 migración Angular Material (barra-lateral)

Tests derivados de la migración de `barra-lateral` a Angular Material.

### Frontend unit (+3 tests pendientes)

| Archivo | Tests | Qué cubre |
|---|---|---|
| `barra-lateral.component.spec.ts` | 1 | `elementosFiltrados()` oculta ítems sin permiso — usa mock de `PermisosUsuarioServicio.tienePermiso()` |
| `barra-lateral.component.spec.ts` | 1 | `seleccionarElemento()` emite `seleccionElemento` output con la clave del elemento |
| `barra-lateral.component.spec.ts` | 1 | `seleccionarNegocioContextoPorId(null)` llama `negocioContexto.limpiar()` |

Archivo destino: `frontend/src/app/shared/components/barra-lateral/barra-lateral.component.spec.ts`

**Total frontend acumulado estimado: ~210 tests** (pending — no ejecutados aún)

---

---

## Sprint 2026-06-02 — Tests Fase 4 migración Angular Material (CRUD admin)

Tests derivados de la migración de los 10 componentes CRUD administración.

### Frontend unit (+2 tests pendientes)

| Archivo | Tests | Qué cubre |
|---|---|---|
| `permisos-lista.component.spec.ts` | 1 | Actualizar mocks: `MessageService`/`ConfirmationService` → `MatSnackBar` + `MatDialog`; verificar que `confirmarCambioEstado` abre `ConfirmDialogComponent` vía `dialog.open()` |
| `mi-negocio-vista.component.spec.ts` | 1 | Mock `MatSnackBar`; verificar que el skeleton usa `animate-pulse` (no `p-skeleton`); `etiquetaPlan(1)` retorna 'Basico'; `colorPlan(2)` retorna clase Tailwind green |

Archivos destino:
- `frontend/src/app/features/permisos/permisos-lista/permisos-lista.component.spec.ts`
- `frontend/src/app/features/mi-negocio/mi-negocio-vista/mi-negocio-vista.component.spec.ts`

**Nota**: componentes solo-iconos (roles-lista, negocios-lista, secciones-con-permisos-lista, usuario-acceso, rol-acceso, negocio-permisos-modal) no generan tests nuevos — el cambio fue puramente visual.

**Total frontend acumulado estimado: ~212 tests** (pending — no ejecutados aún)

## Sprint 2026-06-03 — Tests Fase 5 migración Angular Material (inventario)

Tests derivados de la migración de los 7 componentes inventario.

### Frontend unit (+4 tests pendientes)

| Archivo | Tests | Qué cubre |
|---|---|---|
| `marcas-lista.component.spec.ts` | 1 | Actualizar mocks: `ConfirmationService` → `MatDialog`; verificar que eliminar abre `ConfirmDialogComponent` |
| `etiquetas-lista.component.spec.ts` | 1 | Igual que marcas-lista |
| `productos-lista.component.spec.ts` | 1 | Actualizar providers: reemplazar `MessageService`/`ConfirmationService` por `SnackBarServicio` mock + `MatDialog` mock |
| `catalogo-producto-dialog.component.spec.ts` | 1 | Actualizar providers: reemplazar `MessageService` por `SnackBarServicio` mock |

Archivos destino:
- `frontend/src/app/features/inventario/marcas-lista/marcas-lista.component.spec.ts`
- `frontend/src/app/features/inventario/etiquetas-lista/etiquetas-lista.component.spec.ts`
- `frontend/src/app/features/inventario/productos-lista/productos-lista.component.spec.ts`
- `frontend/src/app/features/inventario/productos-lista/catalogo-producto-dialog/catalogo-producto-dialog.component.spec.ts`

**Nota**: `sedes-lista`, `movimientos`, `sede-stock`, `producto-distribucion` no tienen specs existentes — no generan tests de actualización.

**Total frontend acumulado estimado: ~216 tests** (pending — no ejecutados aún)

---

## Sprint 2026-06-03 — Tests Fase 6 migración Angular Material (mi-pagina catálogo)

Tests derivados de la migración de `mi-pagina.component` — el componente de catálogo público con tabs, imágenes hero y preview vivo.

### Frontend unit (+5 tests pendientes)

| Archivo | Tests | Qué cubre |
|---|---|---|
| `mi-pagina.component.spec.ts` | 1 | `eliminarImagenHero()` abre `ConfirmDialogComponent` vía `MatDialog.open()` con datos `{ titulo, mensaje, textoConfirmar }` |
| `mi-pagina.component.spec.ts` | 1 | `afterClosed()` emite `true` → llama `servicio.eliminarImagenHero(id)` y actualiza signal `imagenesHero` |
| `mi-pagina.component.spec.ts` | 1 | `afterClosed()` emite `false` o `undefined` → NO llama `servicio.eliminarImagenHero()` |
| `mi-pagina.component.spec.ts` | 1 | `guardar()` exitoso → llama `snackBar.exito('Configuración guardada correctamente')` |
| `mi-pagina.component.spec.ts` | 1 | Error en carga inicial (`toObservable + switchMap`) → llama `snackBar.error('No se pudo cargar la configuración de la página')` |

Archivo destino: `frontend/src/app/features/catalogo/mi-pagina/mi-pagina.component.spec.ts`

#### Patrón mock para este componente

```typescript
const snackBarMock = { exito: jest.fn(), error: jest.fn() };
const dialogMock = { open: jest.fn().mockReturnValue({ afterClosed: () => of(true) }) };
const servicioMock = {
  obtener: jest.fn().mockReturnValue(of({ imagenesHero: [], maxImagenesHero: 3 })),
  actualizar: jest.fn().mockReturnValue(of({})),
  eliminarImagenHero: jest.fn().mockReturnValue(of({})),
};

TestBed.configureTestingModule({
  imports: [MiPaginaComponent],
  providers: [
    { provide: SnackBarServicio, useValue: snackBarMock },
    { provide: MatDialog, useValue: dialogMock },
    { provide: MiPaginaCatalogoServicio, useValue: servicioMock },
    { provide: NegocioContextoServicio, useValue: { negocioActivo: signal({ id: 1, nombre: 'Test' }) } },
    // ...otros servicios
  ],
});
```

**Nota**: `DragDropModule` de CDK ya estaba importado antes de la migración — no genera cambios en mocks.

**Total frontend acumulado estimado: ~221 tests** (pending — no ejecutados aún)

---

## Sprint 2026-06-06 — Reemplazo productos-lista → productos-premium (sesión 38)

`productos-lista` eliminado del proyecto. El componente `productos-premium` lo reemplaza completamente. Tests de specs eliminados marcados como obsoletos.

### Tests obsoletos (archivos eliminados — descartar)

| Archivo | Estado |
|---|---|
| `frontend/src/app/features/inventario/productos-lista/productos-lista.component.spec.ts` | **OBSOLETO** — archivo eliminado |
| `frontend/src/app/features/inventario/productos-lista/catalogo-producto-dialog/catalogo-producto-dialog.component.spec.ts` | **OBSOLETO** — archivo eliminado |

> Estos specs fueron marcados como pendientes en sprints anteriores (Fase 5 y backlog 2026-05-14). Ya no aplican.

### E2E Playwright (pendientes — `e2e/tests/04-productos/`)

| Test | Flujo | Archivo |
|---|---|---|
| Verificar reemplazo visual | Navegar a `/dashboard/productos` → carga KPI bar + tabla premium (no la tabla vieja) | `e2e/tests/04-productos/productos-crud.spec.ts` |
| Crear desde modal premium | Botón "Nuevo producto" → modal con tabs → guardar → aparece en tabla | `e2e/tests/04-productos/productos-crud.spec.ts` |
| Distribución inline | Abrir modal de un producto → tab "Distribución" → registrar movimiento INGRESO → stock actualizado | `e2e/tests/04-productos/productos-crud.spec.ts` |

**Nota de página**: `productos.page.ts` probablemente requiere actualización de locators — el componente premium usa selectores distintos a `productos-lista` (sin `.p-dialog`, sin `inv-productos-lista`).

---

## Sprint 2026-06-07 — sedes-lista rediseño premium (sesión 45)

Componente `sedes-lista` reescrito con el mismo design system que `productos-premium`. Tests pendientes por el nuevo lógica de señales y computed.

### Unit tests — Frontend (Jest)

| Test | Archivo |
|------|---------|
| `cargar()` llama GET `api/inventario/sedes` y popula `sedes()` | `frontend/src/app/features/inventario/sedes-lista/sedes-lista.component.spec.ts` |
| `totalActivas` computed retorna solo sedes con `estado === true` | `frontend/src/app/features/inventario/sedes-lista/sedes-lista.component.spec.ts` |
| `totalConDireccion` computed cuenta sedes donde `direccion` no está vacía | `frontend/src/app/features/inventario/sedes-lista/sedes-lista.component.spec.ts` |
| `ordenar('nombre')` alterna `asc` → `desc` al llamar dos veces | `frontend/src/app/features/inventario/sedes-lista/sedes-lista.component.spec.ts` |
| `sedesFiltradas` filtra por `nombre` y por `dirección` (ambos campos) | `frontend/src/app/features/inventario/sedes-lista/sedes-lista.component.spec.ts` |
| `abrirEditar(sede)` precarga `formulario` con `nombre` y `direccion` de la sede | `frontend/src/app/features/inventario/sedes-lista/sedes-lista.component.spec.ts` |
| `confirmarEliminar()` abre dialog y al confirmar llama `servicio.eliminar()` | `frontend/src/app/features/inventario/sedes-lista/sedes-lista.component.spec.ts` |

### E2E Playwright (pendientes)

| Test | Flujo | Archivo |
|------|-------|---------|
| Sedes CRUD | Crear sede → ver en lista con KPI actualizado → editar nombre → eliminar | `e2e/tests/13-sedes/sedes-premium-crud.spec.ts` |
| Filtro dual | Escribir texto de dirección en filtro → solo muestra sedes con esa dirección | `e2e/tests/13-sedes/sedes-premium-crud.spec.ts` |
| Bottom sheet mobile | En viewport 375px → botón crear → modal aparece como bottom sheet (radius top) | `e2e/tests/13-sedes/sedes-premium-crud.spec.ts` |

---

## Sprint 2026-06-07 — sede-stock drawer + toggle custom + SALIDA (sesión 46)

Tests derivados de: conversión de `sede-stock` a drawer, redesign premium, toggle custom de tipo y select nativo, SALIDA como 3er tipo de movimiento.

### Unit tests — Frontend (Jest)

| Test | Archivo |
|------|---------|
| `abrirStock(sede)` actualiza signal `sedeStock()` con la sede seleccionada | `frontend/src/app/features/inventario/sedes-lista/sedes-lista.component.spec.ts` |
| `cerrarStock()` resetea `sedeStock()` a `null` | `frontend/src/app/features/inventario/sedes-lista/sedes-lista.component.spec.ts` |
| Drawer no se renderiza cuando `sedeStock()` es `null` | `frontend/src/app/features/inventario/sedes-lista/sedes-lista.component.spec.ts` |
| `SedeStockComponent` con `[enModal]="true"` oculta `.page-header` | `frontend/src/app/features/inventario/sede-stock/sede-stock.component.spec.ts` |
| `SedeStockComponent` con `[enModal]="false"` muestra `.page-header` | `frontend/src/app/features/inventario/sede-stock/sede-stock.component.spec.ts` |
| `alCambiarTipo('SALIDA')` actualiza `tipoMovimiento` y limpia `productoId` | `frontend/src/app/features/inventario/sede-stock/sede-stock.component.spec.ts` |
| `registrar()` con tipo SALIDA llama `stockServicio.registrarMovimiento({ tipo: 'SALIDA', ... })` | `frontend/src/app/features/inventario/sede-stock/sede-stock.component.spec.ts` |
| `stockMaximoSeleccionado` retorna cantidad disponible en la sede para SALIDA y TRANSFERENCIA, null para INGRESO | `frontend/src/app/features/inventario/sede-stock/sede-stock.component.spec.ts` |
| `stockBajo` computed retorna count de items con `cantidad <= 5` | `frontend/src/app/features/inventario/sede-stock/sede-stock.component.spec.ts` |

### E2E Playwright (pendientes — nuevo spec)

| Test | Flujo | Archivo |
|------|-------|---------|
| Drawer abre sin navegar | Click "Ver stock" en sedes-lista → drawer visible, URL sin cambio | `e2e/tests/10-sedes/sedes-stock-drawer.spec.ts` |
| Backdrop cierra drawer | Click en `.stock-backdrop` → drawer desaparece | `e2e/tests/10-sedes/sedes-stock-drawer.spec.ts` |
| Botón X cierra drawer | Click en `.stock-drawer-close` → drawer desaparece | `e2e/tests/10-sedes/sedes-stock-drawer.spec.ts` |
| Toggle tipo funciona | Dentro del drawer → click "Salida" → tipo activo cambia a rojo, select muestra solo productos con stock | `e2e/tests/10-sedes/sedes-stock-drawer.spec.ts` |
| Mobile 100vw | Viewport 375px → drawer ocupa 100% del ancho | `e2e/tests/10-sedes/sedes-stock-drawer.spec.ts` |

---

## ADR relacionado

`ADR-estrategia-pruebas.md` en la raíz del proyecto documenta la decisión de herramientas y el glosario de términos (qué es E2E, unitaria, integración).

---

## Qué NO se prueba en unit tests

| Qué | Por qué | Dónde se prueba |
|---|---|---|
| Controladores | Solo delegan al servicio, sin lógica propia | E2E (Playwright) |
| Entidades/Modelos | Solo declaran columnas, sin lógica | No requiere prueba |
| Guards complejos | Opcional, solo si tienen lógica condicional | Jest si aplica |
| Queries raw SQL | Usan `sequelize.query()` con SQL puro | Integración (Fase 2+) |

---
titulo: Venta e Inventario — Módulo de Inventario
tipo: proyecto
tags: [inventario, stock, productos, sedes, movimientos, nestjs]
fecha_creacion: 2026-05-07
fecha_actualizacion: 2026-05-14
estado: activo
---

# Venta e Inventario — Módulo de Inventario

Sub-página de [[venta-inventario]].

## Estructura de módulos

```
InventarioModulo
  ├── ProductosModulo
  ├── MarcasModulo
  ├── EtiquetasModulo
  ├── SedesModulo
  └── StockModulo  ← lógica central
```

## Tablas

### `producto`

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| negocio_id | BIGINT FK | tenant isolation |
| nombre | VARCHAR(200) | required |
| sku | VARCHAR(100) | UNIQUE por negocio |
| descripcion | TEXT | nullable |
| stock_minimo | INTEGER | nullable → hereda `configuracion_sistema` |
| + auditoría estándar | | |

**Índices**: `uq_producto_sku_negocio (negocio_id, sku) WHERE eliminado=false`, `idx_producto_nombre`

### `sede`

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| negocio_id | BIGINT FK | |
| nombre | VARCHAR(150) | |
| direccion | VARCHAR(300) | |
| + auditoría estándar | | |

### `stock_sede`

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| negocio_id | BIGINT FK | |
| producto_id | BIGINT FK | |
| sede_id | BIGINT FK | |
| cantidad | INTEGER | CHECK >= 0, default 0 |
| + auditoría estándar | | |

**Constraint**: UNIQUE(producto_id, sede_id) → un registro por combinación producto-sede.

### `movimiento_inventario`

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| negocio_id | BIGINT FK | |
| producto_id | BIGINT FK | |
| sede_id | BIGINT FK | |
| usuario_id | BIGINT FK | quién registró |
| tipo | ENUM | 'INGRESO' \| 'SALIDA' |
| cantidad | INTEGER | CHECK > 0 |
| observacion | TEXT | |
| fecha_movimiento | DATE | default NOW() |
| + auditoría estándar | | |

> Tabla de historial **inmutable** — soft delete pero nunca se purga.

### `configuracion_sistema` (Singleton)

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| stock_minimo_global | INTEGER | default 5 |

### `marca`

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| negocio_id | BIGINT FK | tenant isolation |
| nombre | VARCHAR(150) | UNIQUE por negocio |
| + auditoría estándar | | |

### `etiqueta`

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| negocio_id | BIGINT FK | tenant isolation |
| nombre | VARCHAR(100) | UNIQUE por negocio |
| color | VARCHAR(7) | hex (#RRGGBB) |
| + auditoría estándar | | |

### `producto_etiqueta` (pivote N:M)

| Campo | Tipo | Notas |
|-------|------|-------|
| id | BIGINT PK AI | |
| producto_id | BIGINT FK | |
| etiqueta_id | BIGINT FK | |
| + auditoría estándar | | |

`Producto` tiene `@BelongsToMany(() => Etiqueta, () => ProductoEtiqueta)` — necesario para eager loading vía `include`.

## Endpoints

### Marcas (`/inventario/marcas`)

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/inventario/marcas` | Crear marca |
| GET | `/inventario/marcas` | Listar por negocio |
| PATCH | `/inventario/marcas/:id` | Actualizar |
| DELETE | `/inventario/marcas/:id` | Soft delete |

### Etiquetas (`/inventario/etiquetas`)

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/inventario/etiquetas` | Crear etiqueta |
| GET | `/inventario/etiquetas` | Listar por negocio |
| PATCH | `/inventario/etiquetas/:id` | Actualizar |
| DELETE | `/inventario/etiquetas/:id` | Soft delete |

### Productos (`/inventario/productos`)

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/inventario/productos` | Crear producto |
| GET | `/inventario/productos` | Listar (filtros: nombre, sku) — incluye etiquetas y marcaNombre |
| GET | `/inventario/productos/:id` | Buscar por ID |
| PATCH | `/inventario/productos/:id` | Actualizar |
| DELETE | `/inventario/productos/:id` | Soft delete |

### Stock (`/inventario/stock`)

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/inventario/stock/carga-inicial` | Carga inicial en múltiples sedes |
| GET | `/inventario/stock/producto/:id` | Stock por producto (desglose por sede) |
| GET | `/inventario/stock/sede/:id` | Stock por sede (todos los productos) |
| POST | `/inventario/stock/transferencia` | Transferir entre sedes (transacción ACID) |
| POST | `/inventario/stock/movimiento` | Registrar INGRESO o SALIDA |
| GET | `/inventario/stock/estadisticas-dashboard` | Top 5 bajo stock + sede con menor stock |
| GET | `/inventario/stock/movimientos` | Historial (filtros: productoId, sedeId, tipo) |

## Flujos de negocio

### Registrar movimiento

```
POST /inventario/stock/movimiento
├─ INGRESO: StockSede.cantidad += cantidad (crea registro si no existe)
└─ SALIDA:
   ├─ Valida: cantidad_disponible >= cantidad solicitada
   ├─ Si insuficiente → BadRequestException
   └─ Si ok: StockSede.cantidad -= cantidad
→ Siempre crea registro en movimiento_inventario
```

### Transferencia entre sedes

```
POST /inventario/stock/transferencia [TRANSACCIÓN ACID]
├─ Valida stock en sede origen
├─ Resta de origen
├─ Suma a destino (o crea si no existe)
├─ Crea movimiento SALIDA en origen
└─ Crea movimiento INGRESO en destino
```

### Carga inicial

```
POST /inventario/stock/carga-inicial
├─ Para cada sede en el request:
│  ├─ Si existe y cantidad > 0: omite (retorna razón)
│  ├─ Si existe y cantidad = 0: actualiza
│  └─ Si no existe: crea
└─ Crea movimiento INGRESO con observación "Carga inicial"
→ Retorna: { cargados: N, omitidos: [...] }
```

### Dashboard estadísticas

- Top 5 productos con menor stock total (JOIN stock_sede, suma, orden ASC)
- Para cada uno: sede con menor stock (LATERAL)
- Sede con menor cantidad total de unidades

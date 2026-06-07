---
titulo: Frontend — Migración PrimeNG → Angular Material
tipo: proyecto
tags: [inventario, angular, angular-material, primeng, migracion]
fecha_creacion: 2026-06-07
fecha_actualizacion: 2026-06-07
estado: activo
---

# Migración PrimeNG → Angular Material

Sub-página de [[venta-inventario-frontend]].

**Decisión**: reemplazar PrimeNG 17 completamente por Angular Material 18.x.
**Versión compatible**: Angular 18.2.0 → `@angular/material@18`.
**Estado**: completado 2026-06-03 (Fases 0–6).

## Dimensión del trabajo

32 archivos `.html` · 23 archivos `.ts` · 19 módulos PrimeNG · 15 componentes únicos
2 servicios reemplazados: `MessageService` → `SnackBarServicio`, `ConfirmationService` → `MatDialog`

## Equivalencias PrimeNG → Angular Material

| PrimeNG | Angular Material | Módulo |
|---------|-----------------|--------|
| `p-button` / `pButton` | `mat-button` / `mat-raised-button` | `MatButtonModule` |
| `p-inputText` | `<input matInput>` en `mat-form-field` | `MatInputModule` |
| `p-inputTextarea` | `<textarea matInput>` | `MatInputModule` |
| `p-inputNumber` | `<input matInput type="number">` | `MatInputModule` |
| `p-dropdown` | `<mat-select>` en `mat-form-field` | `MatSelectModule` |
| `p-multiSelect` | `<mat-select multiple>` | `MatSelectModule` |
| `p-checkbox` | `<mat-checkbox>` | `MatCheckboxModule` |
| `p-inputSwitch` | `<mat-slide-toggle>` | `MatSlideToggleModule` |
| `p-selectButton` | `<mat-button-toggle-group>` | `MatButtonToggleModule` |
| `p-dialog` | `MatDialog` + componente standalone | `MatDialogModule` |
| `p-confirmDialog` + `ConfirmationService` | `ConfirmDialogComponent` custom vía `MatDialog` | `MatDialogModule` |
| `p-toast` + `MessageService` | `SnackBarServicio` wrapper de `MatSnackBar` | `MatSnackBarModule` |
| `p-tabView` / `p-tabPanel` | `mat-tab-group` / `mat-tab` | `MatTabsModule` |
| `p-skeleton` | Tailwind `animate-pulse` custom | — |
| `p-progressSpinner` | `<mat-spinner>` | `MatProgressSpinnerModule` |
| `p-avatar` | `<img class="rounded-full w-8 h-8">` | — |
| `p-tag` | `<mat-chip>` | `MatChipsModule` |
| `pRipple` | `matRipple` | `MatRippleModule` |
| `pTooltip` | `matTooltip` | `MatTooltipModule` |
| PrimeIcons `pi pi-*` | `<mat-icon>nombre</mat-icon>` | `MatIconModule` |

## Fases

| Sesión | Fase | Qué se entrega | Estado |
|--------|------|---------------|--------|
| 1 | Fase 0 — Setup | Desinstalar PrimeNG, instalar Material 18, crear `MaterialModule`, `ConfirmDialogComponent`, `SnackBarServicio` | ✅ 2026-06-02 |
| 2 | Fase 1 | 6 componentes autenticación | ✅ 2026-06-02 |
| 3 | Fase 2 | `barra-lateral`, `boton`, `entrada` | ✅ 2026-06-02 |
| 4 | Fase 3 | Panel y dashboard | ✅ 2026-06-02 |
| 5 | Fase 4 | 10 componentes CRUD administración | ✅ 2026-06-02 |
| 6 | Fase 5 | 7 componentes inventario | ✅ 2026-06-03 |
| 7 | Fase 6 | `mi-pagina` (catálogo) | ✅ 2026-06-03 |

## Estado Fase 0 (detalle)

- `primeng`, `primeflex`, `primeicons` — desinstalados
- `@angular/material@18.2.14` — instalado (`@angular/cdk` ya existía)
- `angular.json` — tema PrimeNG reemplazado por `azure-blue.css`
- `styles.css` — variables PrimeNG eliminadas, `--spa-*` conservadas, clases `snack-exito/error/aviso/info`
- `index.html` — font `Material Icons` (ver gotcha en [[venta-inventario-frontend-gotchas]])

**Archivos nuevos creados en Fase 0:**
- `shared/material.module.ts` — exporta todos los módulos Material
- `shared/components/confirm-dialog/confirm-dialog.component.ts` — recibe `{ titulo, mensaje, textoConfirmar }` por `MAT_DIALOG_DATA`
- `core/services/snack-bar.servicio.ts` — API `exito()`, `error()`, `advertencia()`, `info()`

## Notas críticas

- `PrimeFlex` eliminado — sus clases `p-col-*` reemplazadas con Tailwind
- `mat-form-field` obligatorio para cada `matInput` / `mat-select`
- `MatDialog` exige componente formulario separado (standalone); `MatDialogRef` maneja el cierre
- Componente legacy `features/secciones/secciones-permiso-lista/` — aún tiene PrimeNG; no está en ninguna ruta activa; candidato a borrar
- Plan HTML completo: `docs/planes/plan-migracion-primeng-material.html`

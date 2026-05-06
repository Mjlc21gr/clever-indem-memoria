---
inclusion: always
---

# Clever Indem Frontend — Visión General

## ¿Qué es este proyecto?

**Clever Indem** es un microfrontend Angular que gestiona el flujo de siniestros de seguros de vida para **Seguros Bolívar**. Permite radicar casos, analizarlos con IA (OpenL), derivarlos a proveedores (UIFA, Médico, Investigador, Técnico), aprobar pagos/objeciones y consultar el historial completo.

## Stack tecnológico

| Tecnología | Versión | Uso |
|---|---|---|
| Angular | 20.3.17 | Framework principal (standalone components) |
| TypeScript | ~5.9.2 | Lenguaje |
| Tailwind CSS | ^3.4.17 | Utilidades CSS (complementa sb-ui) |
| PrimeNG | 20.4.0 | Solo `UIChart` para gráficas en el dashboard |
| Chart.js | 4.4.8 | Motor de gráficas (usado vía PrimeNG) |
| @angular-architects/module-federation | 20.0.0 | Microfrontend (Module Federation) |
| ngx-build-plus | 20.0.0 | Builder con soporte webpack custom |
| @seguros-bolivar/ui-bundle | ^1.0.7 | **Design System oficial de Seguros Bolívar (sb-ui)** |
| Jest | 30.2.0 | Testing unitario |
| RxJS | ~7.8.0 | Programación reactiva |

## Design System: sb-ui

**REGLA CRÍTICA**: Todo el UI usa clases del Design System `@seguros-bolivar/ui-bundle` (prefijo `sb-ui-`). **NO se usa PrimeNG** para componentes visuales (excepto `UIChart` en el dashboard).

### Clases sb-ui más usadas

```css
/* Botones */
.sb-ui-button                     /* base */
.sb-ui-button--primary            /* verde primario */
.sb-ui-button--secondary          /* secundario */
.sb-ui-button--error              /* rojo/danger */
.sb-ui-button--tertiary           /* amarillo/warn */
.sb-ui-button--fill               /* relleno sólido */
.sb-ui-button--icon-left          /* icono a la izquierda */
.sb-ui-button--icon-only          /* solo icono */

/* Inputs */
.sb-ui-input-container            /* wrapper del campo */
.sb-ui-input-label                /* etiqueta */
.sb-ui-input-inner                /* contenedor del input */
.sb-ui-input                      /* el input nativo */
.sb-ui-textarea                   /* textarea */

/* Select */
.sb-ui-select-container
.sb-ui-select-label
.sb-ui-select

/* Checkbox */
.sb-ui-checkbox
.sb-ui-checkbox-label

/* Tabla */
.sb-ui-table                      /* tabla */
.sb-ui-table--striped

/* Tags / Chips */
.sb-ui-chip
.sb-ui-chip--primary
.sb-ui-chip--soft
.sb-ui-chip--error

/* Badges */
.sb-ui-badge--success
.sb-ui-badge--info
.sb-ui-badge--warning
.sb-ui-badge--error

/* Íconos: Font Awesome 6 (fa-solid, fa-regular) */
```

## Arquitectura del Microfrontend

- **Nombre MFE**: `indem-clever`
- **Remote Entry**: `remoteEntry.js`
- **Puerto dev**: `4202`
- **Expone**: `./Routes` → `exposed.routes.ts`
- **Shell host**: Se conecta desde otro proyecto (shell Angular)

## Variables CSS globales (app)

```css
--app-topbar-height: 3.5rem
--app-sidebar-width: 16rem
--app-sidebar-collapsed-width: 4rem
--app-radius: 8px
--app-green: var(--sb-ui-color-primary-base, #009056)
--app-green-dark: var(--sb-ui-color-primary-D400, #0b613e)
--app-gold: var(--sb-ui-color-secondary-base, #ffe16f)
--app-border: var(--sb-ui-color-grayscale-L200, #e0e0e0)
--app-bg: var(--sb-ui-color-grayscale-L400, #fafafa)
--app-surface: var(--sb-ui-color-grayscale-white, #ffffff)
```

## Scripts disponibles

```bash
npm start                 # ng serve (puerto 4202, dev)
npm run build             # build producción
npm run build:dev         # build con environment dev
npm run build:stage       # build con environment stage
npm run build:prod        # build con environment prod
npm test                  # jest
npm run test:coverage     # jest con coverage
```

## Path Alias

```typescript
@shared  →  projects/indem-clever/src/app/shared/public-api
@shared/* →  projects/indem-clever/src/app/shared/*
```

## Environment

```typescript
// src/environments/environment.ts
{
  production: false,
  apiUrl: '/siniestros/api/v1',   // proxy → backend siniestros
  appName: 'CleverFlow',
  tokenKey: 'cf_token',
}
```

El proxy (`proxy.conf.json`) redirige `/siniestros/api/v1` al backend.

## Estado del backend

**El backend AÚN NO ESTÁ CONECTADO en la mayoría de módulos.** Solo `listar-radicaciones` tiene llamada real al backend vía `RadicacionesService`. El resto usa mocks. Cada TODO en el código marca donde conectar.

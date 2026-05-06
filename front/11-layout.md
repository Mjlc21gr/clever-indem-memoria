---
inclusion: always
---

# Layout de la Aplicación — Clever Indem Frontend

## Estructura del Layout Principal

```
┌─────────────────────────────────────────────────────────┐
│  TOPBAR (3.5rem alto, fixed)                            │
│  [☰ hamburger] [logo] ........................ [avatar] │
└────────────┬────────────────────────────────────────────┘
             │
┌────────────┴──────────────────────────────────────────┐
│ SIDEBAR    │  ROUTER OUTLET (main content)             │
│ (16rem)    │  padding, scroll                          │
│            │                                           │
│  Recepción │  /inicio → Dashboard                      │
│  ├ Radic.  │  /listar-radicaciones → Tabla + Modales   │
│  └ Mesa    │  ...                                       │
│            │                                           │
│  Análisis  │                                           │
│  ├ Gral.   │                                           │
│  ├ Línea   │                                           │
│  └ Mis     │                                           │
│            │                                           │
│ ...        │                                           │
└────────────┴───────────────────────────────────────────┘
```

## Comportamiento del Sidebar

### Desktop (>768px)
- **Normal**: `16rem` de ancho, grupos con labels y chevron para colapsar
- **Collapsed**: `4rem` de ancho (solo iconos), lista plana sin grupos
- Toggle: clic en hamburguesa del topbar → `toggleSidebar()` en App → `sidebarCollapsed` signal
- Clase `.sidebar-collapsed` se agrega a `<html>` cuando colapsado (usada por modal maximizado)

### Mobile (≤768px)
- Sidebar oculto por defecto
- Toggle: clic en hamburguesa → overlay encima del contenido
- Cierre automático al seleccionar un item del menú

## Comunicación App → TopBar → Sidebar

```
App (app.ts)
  ├── userName = computed(() => auth.nombreUsuario())
  ├── sidebarCollapsed = signal(false)
  ├── sidebarMobileOpen = signal(false)
  ├── isMobile = signal(...)
  │
  ├── TopbarComponent
  │     ├── [userName]="userName()"
  │     └── (toggleSidebar) → App.toggleSidebar()
  │
  └── SidebarComponent
        ├── [collapsed]="sidebarCollapsed()"  (desktop)
        ├── o visibilidad según sidebarMobileOpen()  (mobile)
        └── (itemSelected) → App.closeMobileSidebar()
```

## Template del Layout (app.html)

```html
<app-topbar [userName]="userName()" (toggleSidebar)="toggleSidebar()" />

<div class="app-layout">
  <!-- Overlay mobile -->
  @if (isMobile() && sidebarMobileOpen()) {
    <div class="sidebar-overlay" (click)="closeMobileSidebar()"></div>
  }

  <!-- Sidebar -->
  <app-sidebar
    [collapsed]="sidebarCollapsed()"
    [class.sidebar-mobile-open]="sidebarMobileOpen()"
    (itemSelected)="closeMobileSidebar()" />

  <!-- Contenido principal -->
  <main class="app-main" [class.sidebar-collapsed]="sidebarCollapsed()">
    <router-outlet />
  </main>
</div>
```

## Modales Maximizados

Cuando un modal se maximiza:
- Se añade clase `.modal-container--maximized` al contenedor del modal
- El modal ocupa `calc(100vw - 16rem)` × `calc(100vh - 3.5rem)` (desde el topbar hasta abajo, desde el sidebar hasta la derecha)
- Si el sidebar está colapsado (`.sidebar-collapsed` en `<html>`), el modal usa `calc(100vw - 4rem)`
- En mobile (≤768px), el modal maximizado ocupa 100vw × 100vh

## Valores CSS para el Layout

```css
:root {
  --app-topbar-height: 3.5rem;
  --app-sidebar-width: 16rem;
  --app-sidebar-collapsed-width: 4rem;
}

/* Cuando el sidebar está colapsado: */
html.sidebar-collapsed {
  /* Las reglas de modal maximizado se activan automáticamente */
}
```

## Ajuste del Contenido con Sidebar

El contenido principal (`router-outlet`) tiene padding-left que varía:
- `16rem` cuando el sidebar está expandido
- `4rem` cuando está colapsado
- Transición suave de 0.3s

Los modales maximizados respetan este mismo ajuste automáticamente via las clases CSS.

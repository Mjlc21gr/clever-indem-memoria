---
inclusion: always
---

# Estilos y Design System — Clever Indem Frontend

## Design System: @seguros-bolivar/ui-bundle (sb-ui)

Es la librería de componentes oficial de Seguros Bolívar.  
Se carga como asset CSS+JS en `angular.json`:
```
assets/sb-ui/sb-ui-seguros-bolivar.min.css
assets/sb-ui/sb-ui-seguros-bolivar.min.js
```

**REGLA**: No usar PrimeNG para UI. Solo `UIChart` de PrimeNG para gráficas.

---

## Variables CSS de la Aplicación

Definidas en `src/styles.scss`:

```css
:root {
  /* Layout */
  --app-topbar-height: 3.5rem;
  --app-sidebar-width: 16rem;
  --app-sidebar-collapsed-width: 4rem;
  --app-radius: 8px;
  --app-shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.06);
  --app-shadow-md: 0 4px 12px rgba(0, 0, 0, 0.08);
  --app-transition: 0.25s ease;

  /* Colores (referencias a sb-ui) */
  --app-green: var(--sb-ui-color-primary-base, #009056);        /* Verde primario */
  --app-green-dark: var(--sb-ui-color-primary-D400, #0b613e);   /* Verde oscuro */
  --app-green-mid: var(--sb-ui-color-primary-D100, #038450);
  --app-green-light: var(--sb-ui-color-primary-L300, #e5f4ee);  /* Verde muy claro */
  --app-gold: var(--sb-ui-color-secondary-base, #ffe16f);       /* Dorado Seguros Bolívar */
  --app-gold-dark: var(--sb-ui-color-secondary-D400, #ffc918);
  --app-border: var(--sb-ui-color-grayscale-L200, #e0e0e0);     /* Borde estándar */
  --app-bg: var(--sb-ui-color-grayscale-L400, #fafafa);         /* Fondo app */
  --app-surface: var(--sb-ui-color-grayscale-white, #ffffff);   /* Fondo cards */
  --app-text: var(--sb-ui-color-grayscale-black, #1b1b1b);
  --app-text-muted: var(--sb-ui-color-grayscale-base, #9b9b9b);
  --app-error: var(--sb-ui-color-feedback-error-base, #dc3545);
}
```

---

## Clases CSS Globales (styles.scss)

### Formularios

```css
.form-grid            /* grilla 3 columnas, gap 1rem */
.form-grid--2col      /* grilla 2 columnas */
.form-grid--full      /* 1 columna (full width) */
.form-group           /* flex column gap 0.25rem */
.form-group__label    /* label 0.8125rem font-weight 500 */
.required             /* texto rojo (#dc3545) */
```

### Section Card (reemplaza fieldsets)

```css
.section-card                    /* contenedor principal */
.section-card__header            /* header clickeable (toggle) */
.section-card__title             /* título con borde verde izquierdo */
.section-card__toggle            /* chevron que rota al colapsar */
.section-card__toggle--collapsed /* rotado -90deg */
.section-card__body              /* contenido de la sección */
```

### Modal

```css
.modal-overlay                   /* fondo oscuro fixed */
.modal-overlay--maximized        /* overlay en modo max (sin fondo) */
.modal-container                 /* caja del modal */
.modal-container--maximized      /* ocupa toda el área (sin topbar+sidebar) */
.modal-header                    /* cabecera verde con gradiente + línea dorada */
.modal-header__title             /* título en blanco */
.modal-header__actions           /* botones (maximizar, cerrar) */
.modal-header__btn               /* botón icono transparente en header */
.modal-body                      /* cuerpo scrolleable */
.modal-footer                    /* pie con botones (flex-end) */
```

**Modo maximizado**: El modal se expande a `calc(100vw - sidebar-width)` × `calc(100vh - topbar-height)`. Cuando el sidebar está colapsado, usa `--app-sidebar-collapsed-width`. La clase `.sidebar-collapsed` en `<html>` activa los overrides automáticamente.

### Page Panel (paneles de páginas de listado)

```css
.page-panel                      /* contenedor con borde */
.page-panel__header              /* cabecera verde con gradiente + línea dorada */
.page-panel__body                /* contenido */
```

### Toolbar de Tabla

```css
.table-toolbar                   /* flex between, botones y búsqueda */
.table-toolbar__search           /* wrapper del buscador */
.table-toolbar__search-icon      /* ícono dentro del buscador */
.table-toolbar__search-input     /* input con padding para el ícono */
```

### Paginación

```css
.pagination                      /* flex between info + controles */
.pagination__info                /* "Mostrando X-Y de Z" */
.pagination__controls            /* botones de página */
.pagination__btn                 /* botón individual de página */
.pagination__btn--active         /* página actual (fondo verde) */
.pagination__select              /* select de filas por página */
```

### Empty State

```css
.empty-state                     /* centered, gris, ícono grande */
```

### Checklist Grid

```css
.checklist-grid                  /* grid 2 columnas */
.checklist-item                  /* fila de checkbox con hover verde claro */
```

### Utilidades

```css
.mt-2                            /* margin-top: 0.5rem */
.mt-3                            /* margin-top: 0.75rem */
.text-muted                      /* color gris, 0.8125rem, italic */
```

### Animaciones

```css
@keyframes fadeIn  { from { opacity: 0 } to { opacity: 1 } }
@keyframes slideUp { from { opacity: 0; transform: translateY(20px) } to { opacity: 1; transform: translateY(0) } }
```

---

## Clases sb-ui más usadas en el código

### Botones
```html
<!-- Botón primario (verde sólido) -->
<button class="sb-ui-button sb-ui-button--primary sb-ui-button--fill">
<!-- Con icono izquierda -->
<button class="sb-ui-button sb-ui-button--primary sb-ui-button--fill sb-ui-button--icon-left">
  <i class="fa-solid fa-save"></i> Guardar
</button>
<!-- Secundario -->
<button class="sb-ui-button sb-ui-button--secondary">Cancelar</button>
<!-- Error/Danger -->
<button class="sb-ui-button sb-ui-button--error sb-ui-button--fill">Eliminar</button>
<!-- Solo icono -->
<button class="sb-ui-button sb-ui-button--secondary sb-ui-button--icon-only">
  <i class="fa-solid fa-xmark"></i>
</button>
```

### Inputs
```html
<div class="sb-ui-input-container">
  <label class="sb-ui-input-label">Etiqueta</label>
  <div class="sb-ui-input-inner">
    <input class="sb-ui-input" [(ngModel)]="valor" placeholder="..." />
  </div>
</div>

<!-- Textarea -->
<textarea class="sb-ui-textarea" [(ngModel)]="valor" rows="4"></textarea>

<!-- Select -->
<div class="sb-ui-select-container">
  <label class="sb-ui-select-label">Etiqueta</label>
  <select class="sb-ui-select" [(ngModel)]="valor">
    <option value="">Seleccionar</option>
    <option value="x">Opción X</option>
  </select>
</div>

<!-- Checkbox -->
<input type="checkbox" class="sb-ui-checkbox" [(ngModel)]="valor" id="chk" />
<label class="sb-ui-checkbox-label" for="chk">Texto</label>
```

### Tabla
```html
<table class="sb-ui-table">
  <thead>
    <tr>
      <th>Columna</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Valor</td>
    </tr>
  </tbody>
</table>
```

### Chips/Tags
```html
<span class="sb-ui-chip sb-ui-chip--primary sb-ui-chip--soft">Activo</span>
<span class="sb-ui-chip sb-ui-chip--error sb-ui-chip--soft">Rechazado</span>
<span class="sb-ui-chip sb-ui-chip--soft">Pendiente</span>
```

---

## Íconos: Font Awesome 6

Se usa Font Awesome 6 (cargado vía sb-ui bundle):
```html
<i class="fa-solid fa-eye"></i>          <!-- ver -->
<i class="fa-solid fa-pen"></i>          <!-- editar -->
<i class="fa-solid fa-trash"></i>        <!-- eliminar -->
<i class="fa-solid fa-plus"></i>         <!-- agregar -->
<i class="fa-solid fa-save"></i>         <!-- guardar -->
<i class="fa-solid fa-xmark"></i>        <!-- cerrar -->
<i class="fa-solid fa-check"></i>        <!-- confirmar -->
<i class="fa-solid fa-rotate-left"></i>  <!-- devolver -->
<i class="fa-solid fa-circle-info"></i>  <!-- info -->
<i class="fa-solid fa-search"></i>       <!-- buscar -->
<i class="fa-solid fa-chevron-down"></i> <!-- expandir -->
<i class="fa-solid fa-arrows-maximize"></i> <!-- maximizar -->
```

---

## Responsive

```scss
@media (max-width: 992px) {
  .form-grid → 2 columnas
  .modal-container → 95vw
}

@media (max-width: 768px) {
  .modal-container → fullscreen (100vw, 100vh, sin border-radius)
  .modal-container--maximized → top:0, left:0 (ignora sidebar)
  .checklist-grid → 1 columna
}

@media (max-width: 576px) {
  .form-grid, .form-grid--2col → 1 columna
  .modal-footer → flex-direction: column
  .modal-footer .sb-ui-button → width: 100%
}
```

---

## Scrollbar personalizado

```css
::-webkit-scrollbar { width: 6px; height: 6px; }
::-webkit-scrollbar-thumb { background-color: #cbd5e1; border-radius: 3px; }
::-webkit-scrollbar-thumb:hover { background-color: #94a3b8; }
```

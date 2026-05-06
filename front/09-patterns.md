---
inclusion: always
---

# Patrones y Convenciones de Código — Clever Indem Frontend

## Reglas Absolutas

1. **NO PrimeNG para UI** — Solo `UIChart` de PrimeNG para gráficas. Todo lo demás usa sb-ui.
2. **Standalone Components** — Todos los componentes son standalone (no NgModule).
3. **Signals everywhere** — Estado reactivo con `signal()`, `computed()`, `model()`. No BehaviorSubject.
4. **ChangeDetectionStrategy.OnPush** — Todos los componentes usan OnPush.
5. **inject() function** — Dependencias con `inject()`, no constructor injection.
6. **Functional guards** — Guards como funciones (`CanActivateFn`), no clases.
7. **Functional interceptors** — `HttpInterceptorFn`, no clases.

---

## Patrón de Componente Feature (página de listado)

```typescript
@Component({
  selector: 'app-listar-xxx',
  imports: [TablaDinamicaComponent, ModalXxxComponent],
  templateUrl: './listar-xxx.component.html',
  styleUrl: './listar-xxx.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ListarxxxComponent {
  // 1. Estado
  loading = signal(false);
  data = signal<unknown[]>([]);         // o el tipo específico
  showModal = signal(false);

  // 2. Configuración de tabla
  readonly columnas: ColumnaTabla[] = [
    { field: 'id', header: 'ID', sortable: true },
    { field: 'estado', header: 'Estado', type: 'tag', tagMap: { ... } },
  ];

  readonly acciones = [
    { action: 'ver', icon: 'fa-solid fa-eye', tooltip: 'Ver', severity: 'info' },
  ];

  // 3. Handlers
  onAccion(event: AccionFila): void {
    if (event.action === 'ver') { this.showModal.set(true); }
  }

  closeModal(): void { this.showModal.set(false); }
}
```

---

## Patrón de Modal

```typescript
@Component({
  selector: 'app-modal-xxx',
  imports: [ /* secciones necesarias */ ],
  templateUrl: './modal-xxx.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ModalXxxComponent {
  visible = input(false);
  readonly onClose = output<void>();

  maximized = false;  // boolean simple (no signal) para toggle maximizado

  close(): void { this.maximized = false; this.onClose.emit(); }
  toggleMaximize(): void { this.maximized = !this.maximized; }
}
```

### Template de modal estándar

```html
@if (visible()) {
  <div class="modal-overlay" [class.modal-overlay--maximized]="maximized" (click)="!maximized && close()">
    <div class="modal-container" [class.modal-container--maximized]="maximized" (click)="$event.stopPropagation()">
      
      <!-- Header -->
      <div class="modal-header">
        <span class="modal-header__title">
          <i class="fa-solid fa-file-circle-plus"></i> Título del Modal
        </span>
        <div class="modal-header__actions">
          <button class="modal-header__btn" (click)="toggleMaximize()">
            <i [class]="maximized ? 'fa-solid fa-compress' : 'fa-solid fa-expand'"></i>
          </button>
          <button class="modal-header__btn" (click)="close()">
            <i class="fa-solid fa-xmark"></i>
          </button>
        </div>
      </div>

      <!-- Body -->
      <div class="modal-body">
        <!-- Secciones -->
        <app-seccion-formulario-dinamico [campos]="campos" [valores]="valores()" (valoresChange)="valores.set($event)" />
      </div>

      <!-- Footer -->
      <div class="modal-footer">
        <button class="sb-ui-button sb-ui-button--secondary" (click)="close()">Cancelar</button>
        <button class="sb-ui-button sb-ui-button--primary sb-ui-button--fill" (click)="guardar()">
          Guardar
        </button>
      </div>

    </div>
  </div>
}
```

---

## Patrón de Página de Listado (template)

```html
<div class="page-panel">
  <!-- Header con título y botones de acción -->
  <div class="page-panel__header">
    <span><i class="fa-solid fa-icono"></i> Título Módulo</span>
    <div>
      <button class="sb-ui-button sb-ui-button--secondary sb-ui-button--icon-left" (click)="openModal()">
        <i class="fa-solid fa-plus"></i> Nueva Acción
      </button>
    </div>
  </div>

  <!-- Tabla dinámica -->
  <div class="page-panel__body">
    <app-tabla-dinamica
      [columnas]="columnas"
      [datos]="data()"
      [loading]="loading()"
      [showActions]="true"
      [acciones]="acciones"
      (accionEjecutada)="onAccion($event)">
    </app-tabla-dinamica>
  </div>
</div>

<!-- Modal -->
<app-modal-xxx [visible]="showModal()" (onClose)="closeModal()" />
```

---

## Two-Way Binding con Signals

```typescript
// En el componente padre — signal mutable:
valores = signal<Record<string, string>>({});

// En el template:
<app-seccion-formulario-dinamico
  [valores]="valores()"
  (valoresChange)="valores.set($event)" />

// Para model() en componentes hijos:
// El hijo: valor = model('')
// El padre: <app-hijo [(valor)]="miSignal" /> (requiere Angular 17.2+)
// O descompuesto: [valor]="miSignal()" (valueChange)="miSignal.set($event)"
```

---

## Convenciones de Nombres

| Tipo | Convención | Ejemplo |
|---|---|---|
| Componente página | `Listar{Nombre}Component` | `ListarradicacionesComponent` |
| Componente modal | `Modal{Nombre}Component` | `ModalAgregarCasoComponent` |
| Componente ventana | `Ventana{Nombre}Component` | `VentanaDatosGeneralesComponent` |
| Componente sección | `Seccion{Nombre}Component` | `SeccionFormularioDinamicoComponent` |
| Servicio | `{Nombre}Service` | `RadicacionesService` |
| Guard | `{nombre}Guard` | `authGuard` |
| Interceptor | `{nombre}Interceptor` | `authInterceptor` |
| Mock data | `MOCK_{NOMBRE}` | `MOCK_RADICACIONES` |
| Selector | `app-{kebab-case}` | `app-tabla-dinamica` |

---

## Cómo consumir el backend en un componente

```typescript
// En ngOnInit o en un método de carga:
ngOnInit(): void {
  this.cargar();
}

cargar(): void {
  this.loading.set(true);
  this.miServicio.listar(params).subscribe({
    next: (paginado) => {
      this.data.set(paginado.content);
      this.loading.set(false);
    },
    error: () => {
      this.notificacionService.error('Mensaje de error');
      this.loading.set(false);
    },
  });
}
```

---

## Severity → Clases sb-ui (referencia rápida)

| severity | Chip class | Button class | Badge class |
|---|---|---|---|
| `success` | `sb-ui-chip--primary sb-ui-chip--soft` | `sb-ui-button--primary sb-ui-button--fill` | `sb-ui-badge--success` |
| `danger` / `error` | `sb-ui-chip--error sb-ui-chip--soft` | `sb-ui-button--error sb-ui-button--fill` | `sb-ui-badge--error` |
| `warn` / `warning` | `sb-ui-chip--soft` | `sb-ui-button--tertiary sb-ui-button--fill` | `sb-ui-badge--warning` |
| `info` | `sb-ui-chip--soft` | `sb-ui-button--secondary sb-ui-button--fill` | `sb-ui-badge--info` |
| `secondary` | `sb-ui-chip--soft` | `sb-ui-button--secondary` | `sb-ui-badge--default` |

---

## Cómo agregar una nueva ruta/módulo

1. Crear carpeta `features/indem-clever/components/listar-{nombre}/`
2. Crear archivos: `.component.ts`, `.component.html`, `.component.scss`
3. Agregar ruta en `exposed.routes.ts`:
   ```typescript
   {
     path: 'listar-{nombre}',
     canActivate: [authGuard],
     loadComponent: () => import('./features/.../listar-{nombre}.component').then(m => m.Listar{nombre}Component),
   }
   ```
4. Agregar al `SIDEBAR_MENU` en `shared/data/sidebar-menu.data.ts` si aplica

---

## Convenciones de TagMap

Para columnas tipo `tag` en `TablaDinamicaComponent`:
```typescript
tagMap: {
  'VALOR_BACKEND': { label: 'Texto Mostrado', severity: 'success' | 'danger' | 'warn' | 'info' | 'secondary' }
}
// Ejemplo estados típicos:
'Activo' / 'A': severity: 'success'
'Inactivo' / 'I': severity: 'danger'
'En proceso' / 'P': severity: 'info'
'Cerrado' / 'C': severity: 'secondary'
'Pendiente': severity: 'warn'
'Aprobado': severity: 'success'
'Rechazado': severity: 'danger'
```

---

## Módulo Federation: Exponer nuevas rutas

Si el shell necesita acceder a nuevas rutas, se agregan en `exposed.routes.ts` (ya exportado como `./Routes` en webpack.config.js). No es necesario tocar el webpack config para agregar rutas, solo el archivo de rutas.

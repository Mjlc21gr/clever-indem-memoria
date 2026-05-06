---
inclusion: always
---

# Componentes Compartidos (shared/) — Clever Indem Frontend

Todos se exportan desde `@shared` (alias de `src/app/shared/public-api.ts`).

---

## Layout

### TopbarComponent
**Selector**: `app-topbar`  
**Archivo**: `shared/components/topbar/topbar.component.ts`

```typescript
// Inputs
userName = input('Usuario')         // nombre del usuario en avatar

// Outputs
toggleSidebar = output<void>()      // clic en hamburguesa → toggle sidebar
```

### SidebarComponent
**Selector**: `app-sidebar`  
**Archivo**: `shared/components/sidebar/sidebar.component.ts`

```typescript
// Inputs
collapsed = input(false)            // colapsar a modo icon-only

// Outputs
itemSelected = output<void>()       // clic en item → cerrar sidebar mobile

// Comportamiento:
// - Modo normal: grupos colapsables con labels
// - Modo collapsed: lista plana de iconos con tooltips
// - Usa SIDEBAR_MENU de shared/data/sidebar-menu.data.ts
// - HostBinding: class.sidebar-collapsed cuando collapsed()
```

---

## TablaDinamicaComponent
**Selector**: `app-tabla-dinamica`  
**Archivo**: `shared/components/tabla-dinamica/tabla-dinamica.component.ts`  
**Exportado**: `TablaDinamicaComponent`, tipos `ColumnaTabla`, `FilaSeleccionada`, `AccionFila`

### Inputs
```typescript
columnas = input.required<ColumnaTabla[]>()           // definición de columnas (REQUERIDO)
datos = input<unknown[]>([])                          // array de datos
loading = input(false)                               // mostrar skeleton/loader
rowsPerPage = input(10)                              // filas por página
rowsPerPageOptions = input([10, 25, 50])             // opciones de paginación
showGlobalFilter = input(true)                       // mostrar buscador global
showPaginator = input(true)                          // mostrar paginación
selectable = input(false)                            // habilitar selección de filas
selectionMode = input<'single' | 'multiple'>('single')
dataKey = input('id')                                // campo clave para selección
emptyMessage = input('No se encontraron registros.') // mensaje si no hay datos
showActions = input(false)                           // mostrar columna de acciones
actionsHeader = input('Acciones')                    // header columna acciones
actionsWidth = input('100px')                        // ancho columna acciones
acciones = input<{ action: string; icon: string; tooltip: string; severity?: string }[]>([])
```

### Outputs
```typescript
filaSeleccionada = output<FilaSeleccionada>()         // clic en fila
accionEjecutada = output<AccionFila>()                // clic en botón de acción
```

### Características
- Filtrado global en todos los campos (debounce por input del usuario)
- Ordenamiento por columna (click en header, toggle asc/desc)
- Paginación client-side con control de página
- Selección múltiple con checkbox (cuando `selectable=true` y `selectionMode='multiple'`)
- Tipos de celda: `text`, `date` (formato locale), `currency` (CurrencyPipe), `tag` (sb-ui-chip), `boolean`
- `tagMap`: mapea valor del campo a `{label, severity}` para badges de color
- Severity→clase sb-ui: `success`→`sb-ui-chip--primary`, `danger`→`sb-ui-chip--error`
- Soporta dot-notation en `field` (ej: `"usuario.nombre"`)

### Uso típico
```typescript
// En el componente padre:
columnas: ColumnaTabla[] = [
  { field: 'id', header: 'ID', sortable: true },
  { field: 'estado', header: 'Estado', type: 'tag', tagMap: {
    'Activo': { label: 'Activo', severity: 'success' },
    'Inactivo': { label: 'Inactivo', severity: 'danger' },
  }},
];
acciones = [{ action: 'ver', icon: 'fa-solid fa-eye', tooltip: 'Ver', severity: 'info' }];

onAccion(event: AccionFila): void {
  if (event.action === 'ver') { /* abrir modal */ }
}
```
```html
<app-tabla-dinamica
  [columnas]="columnas"
  [datos]="data()"
  [loading]="loading()"
  [showActions]="true"
  [acciones]="acciones"
  (accionEjecutada)="onAccion($event)">
</app-tabla-dinamica>
```

---

## BotonAccionComponent
**Selector**: `app-boton-accion`  
**Archivo**: `shared/components/boton-accion/boton-accion.component.ts`

```typescript
// Inputs
label = input('')
icon = input('')                     // clase FontAwesome (ej: 'fa-solid fa-save')
severity = input<'success' | 'secondary' | 'info' | 'warn' | 'danger' | 'contrast'>('secondary')
outlined = input(false)
disabled = input(false)

// Outputs
accion = output<void>()

// Mapeo severity → sb-ui clases:
// success → sb-ui-button--primary sb-ui-button--fill
// info    → sb-ui-button--secondary sb-ui-button--fill
// secondary → sb-ui-button--secondary
// warn    → sb-ui-button--tertiary sb-ui-button--fill
// danger  → sb-ui-button--error sb-ui-button--fill
// contrast → sb-ui-button--secondary
```

---

## DialogoConfirmacionComponent
**Selector**: `app-dialogo-confirmacion`  
**Archivo**: `shared/components/dialogo-confirmacion/dialogo-confirmacion.component.ts`

```typescript
// Inputs
visible = input(false)
titulo = input('Confirmar')
mensaje = input('¿Está seguro de realizar esta acción?')
labelConfirmar = input('Confirmar')
severityConfirmar = input<'success' | 'danger' | 'warn' | 'info'>('success')

// Outputs
confirmar = output<void>()
cancelar = output<void>()
```

---

## Secciones Reutilizables para Modales

### SeccionFormularioDinamicoComponent
**Selector**: `app-seccion-formulario-dinamico`

```typescript
// Inputs
legend = input('')                           // título de la sección
toggleable = input(true)                     // permite colapsar
campos = input<CampoFormulario[]>([])        // definición de campos
valores = input<Record<string, string>>({})  // valores actuales (two-way binding vía output)

// Outputs
valoresChange = output<Record<string, string>>()  // emite valores actualizados

// Behavior: campos sin fullWidth=true van en grilla de 3 columnas
//           campos con fullWidth=true van en fila completa debajo
```

### SeccionObservacionesGenericaComponent
**Selector**: `app-seccion-observaciones-generica`

```typescript
legend = input('Observaciones')
placeholder = input('Escriba sus observaciones aquí...')
rows = input(4)
valor = model('')                            // two-way binding NgModel
```

### SeccionPanelGenericoComponent
**Selector**: `app-seccion-panel-generico`

```typescript
header = input('')
emptyMessage = input('Sin datos registrados.')
toggleable = input(true)
collapsed = input(true)                     // estado inicial (colapso)
```

### SeccionArchivosComponent
**Selector**: `app-seccion-archivos`

```typescript
// Outputs
filesSelected = output<{ files: File[] }>()  // cuando el usuario selecciona archivos
// Acepta: PDF, imágenes. Multiple archivos.
```

### SeccionConsultaComponent
**Selector**: `app-seccion-consulta`

```typescript
consultaTipoDoc = model('')          // tipo de documento (CC, NIT, etc.)
consultaNumDoc = model('')           // número de documento
consultar = output<void>()           // clic en botón "Consultar"
```

### SeccionDecisionComponent
**Selector**: `app-seccion-decision`

```typescript
// Propiedades internas (no inputs)
nuevoEstado = ''          // select con opciones de decisión
causalAbordada = ''       // select de causal
observacionAnalista = signal('')

// Opciones de decisión:
// 'DEFINIDO PAGO' | 'DEFINIDO OBJECION' | 'MOVILIZADO' | 'DEVOLVER RADICACION' | 'ANULAR CASO'
```

### SeccionDerivacionComponent
**Selector**: `app-seccion-derivacion`

```typescript
// Todos son model() (two-way binding)
enviarProveedores = model('no')          // 'si' | 'no'
enviarLineaNegocio = model(false)
proveedorInvestigador = model(false)
proveedorUIFA = model(false)
proveedorTecnico = model(false)
agenteInvestigadorRadi = model('')
motivoMovilizacionInvestigador = model('')
motivoMovilizacionUIFA = model('')
agenteTecnicoRadi = model('')
motivoMovilizacionTecnico = model('')
agentesInvestigador = model<{ label: string; value: string }[]>([])
agentesTecnico = model<{ label: string; value: string }[]>([])

// Tipos de investigación: 'Verificación' | 'Informe Completo' | 'Fraude'
```

### SeccionChecklistComponent
**Selector**: `app-seccion-checklist`

```typescript
// Todos son model(false) — documentos del checklist:
regCivilDefuncion = model(false)
docIdentidadAsegurado = model(false)
epicrisis = model(false)
certFiscalia = model(false)
sentenciaFecha = model(false)
croquis = model(false)
dictamenCapacidad = model(false)
regCivilBeneficiarios = model(false)
docBeneficiarios = model(false)
formB114 = model(false)
formB121 = model(false)
certHospitalizacion = model(false)
certIncapacidad = model(false)
examenesDiagnostico = model(false)
formB337 = model(false)
pruebaAlcoholemia = model(false)
reciboFactura = model(false)
certDeuda = model(false)
```

### SeccionAnalisisIaComponent
**Selector**: `app-seccion-analisis-ia`

```typescript
// No tiene inputs/outputs externos — todo interno
// Secciones de análisis IA:
// - Hospitalización General (camposHospGeneral + camposReglasHosp)
// - Hospitalización por Accidente (camposHospAccidente + camposReglasAccidente)
// - Cirugía (camposCirugia + camposReglasCirugia)
// - Incapacidad (camposIncapacidad + camposReglasIncapacidad)
// Cada sección usa SeccionFormularioDinamicoComponent
// + SeccionObservacionesGenericaComponent para observaciones IA
```

### SeccionLineaTiempoComponent
**Selector**: `app-seccion-linea-tiempo`

```typescript
// Inputs
legend = input('Historial del Caso')
toggleable = input(true)
etapas = input<EtapaLineaTiempo[]>([])

// Interfaz EtapaLineaTiempo:
// { label: string, completada?: boolean, activa?: boolean, gestiones?: GestionEtapa[] }

// Behavior:
// - Stepper horizontal con estado visual (completada/activa/pendiente)
// - Clic en etapa → acordeón que muestra sus gestiones
// - Solo un panel abierto a la vez (activePanel signal)
```

---

## Configuración del Menú Lateral

**Archivo**: `shared/data/sidebar-menu.data.ts`

```typescript
// Grupos del menú (en orden):
// 1. Recepción   → Radicaciones, Mesa de Transformación
// 2. Análisis    → Análisis General, Línea de Negocio, Mis Casos
// 3. Proveedores → UIFA, Médico, Investigador, Técnico
// 4. Decisiones  → Pagos y Objeciones
// 5. Seguimiento → Consultar Casos
// 6. Administrar → Total Casos, Usuarios
```

Interfaz `SidebarMenuGroup`:
```typescript
interface SidebarMenuGroup {
  label: string;
  icon: string;           // clase FontAwesome
  collapsedTooltip: string;
  items: SidebarMenuItem[];
}

interface SidebarMenuItem {
  label: string;
  icon: string;           // clase FontAwesome
  routerLink: string;     // ruta absoluta (ej: '/listar-radicaciones')
}
```

---
inclusion: always
---

# Componentes de Features — Clever Indem Frontend

## InicioComponent — Dashboard Gerencial
**Ruta**: `/inicio`  
**Archivo**: `features/indem-clever/components/inicio/inicio.component.ts`

Dashboard ejecutivo con:
- **6 KPIs** (Casos Activos, Pendientes Hoy, En Análisis, Pagos por Aprobar, Tiempo Promedio, Tasa Resolución)
- **4 gráficas** vía `UIChart` de PrimeNG + Chart.js:
  - Casos por módulo (barras verticales, `chartModulosData`)
  - Distribución por estado (dona/donut, `chartEstadoData`)
  - Tendencia mensual (línea, `chartTendenciaData`)
  - Rendimiento por analista (barras horizontales, `chartRendimientoData`)
- **Top 5 gestores** (tabla estática, `topUsuarios`)
- **6 accesos rápidos** (cards con ruta y contador, `accesos`)

**Nota**: Todos los datos son mock. El nombre del usuario viene de `AuthService.nombreUsuario()`.

---

## ListarradicacionesComponent — Radicaciones
**Ruta**: `/listar-radicaciones`  
**Guard**: `authGuard`  
**Archivo**: `features/indem-clever/components/listar-radicaciones/listar-radicaciones.component.ts`

**ÚNICO módulo que llama al backend** vía `RadicacionesService.listar({ estado: 'R', page: 0, size: 10 })`.

Columnas: ID Radicado, Número Póliza, Fecha Aviso, Decisión (tag), Cobertura, Tipo Póliza, Estado (tag).

**Acciones por fila**: `ver` → abre `ModalRadicacionComponent`

**Botones en header**: "Agregar Caso" → `ModalAgregarCasoComponent`, "Mesa de Perfeccionamiento" → `ModalMesaPerfeccionamientoComponent`

### Subcomponentes:

#### ModalAgregarCasoComponent
**Selector**: `app-modal-agregar-caso`  
**Archivo**: `listar-radicaciones/modal-agregar-caso/modal-agregar-caso.component.ts`

Modal de 3 pestañas (tabs manejados con `activeTab` signal):
- **Tab 0 — Datos Generales**: `VentanaDatosGeneralesComponent`
- **Tab 1 — Data Operativa**: `VentanaDataOperativaComponent`
- **Tab 2 — Derivación**: `SeccionDerivacionComponent`

Soporta modo maximizado (`maximized` boolean).  
TODO pendiente: `guardarRadicado()`, `anularCaso()`, `onFileUpload()` → conectar al backend.

```typescript
// Inputs
visible = input(false)
// Outputs
onClose = output<void>()
```

#### VentanaDatosGeneralesComponent
**Selector**: `app-ventana-datos-generales`  
**Archivo**: `modal-agregar-caso/ventana-datos-generales/ventana-datos-generales.component.ts`

Formularios de datos del asegurado, radicado, siniestro, información bancaria.

#### VentanaDataOperativaComponent
**Selector**: `app-ventana-data-operativa`  
**Archivo**: `modal-agregar-caso/ventana-data-operativa/ventana-data-operativa.component.ts`

Consulta de datos operativos del asegurado.

#### ModalMesaPerfeccionamientoComponent
**Selector**: `app-modal-mesa-perfeccionamiento`  
**Archivo**: `listar-radicaciones/modal-mesa-perfeccionamiento/modal-mesa-perfeccionamiento.component.ts`

#### ModalRadicacionComponent
**Selector**: `app-modal-radicacion`  
**Archivo**: `listar-radicaciones/modal-radicacion/modal-radicacion.component.ts`

---

## ListarmesaperfectComponent — Mesa de Transformación
**Ruta**: `/listar-mesa-perfect`  
**Guard**: `authGuard`

Usa `MOCK_RADICACIONES`. Columnas: ID, Póliza, Cobertura, F. Aviso, Decisión (tag), Estado (tag).  
Acciones: `ver`, `editar` (TODO: sin modal aún).

---

## ListaranalisisComponent — Análisis General
**Ruta**: `/listar-analisis`  
**Guard**: `authGuard`

Usa `MOCK_ANALISIS`. Columnas: ID, Asegurado, Póliza, F. Aviso, Cobertura, Decisión IA (tag), Estado (tag).  
Acción: `ver` → `ModalAnalisisComponent`

### ModalAnalisisComponent (compartido)
**Selector**: `app-modal-analisis`  
**Archivo**: `listar-analisis/modal-analisis/modal-analisis.component.ts`  
**Reutilizado por**: `listar-analisis`, `listar-analisis-linea`, `listar-mis-analisis`

Modal completo de análisis de caso:

**Secciones via `SeccionFormularioDinamicoComponent`**:
- `camposAsegurado` — Tipo Doc, Nro Doc, Nombre, Edad, Correo, Teléfono
- `camposRadicado` — ID, Nro Póliza, Cód Producto, Nro Riesgo, Fecha Aviso, Cobertura, Cod Cobertura, Portafolio Cod/Nombre, Valor Asegurado, Val Aseg Vida/ITP, Score, Aprobación Términos, NIT/Nombre Empresa
- `camposSiniestro` — Nro Siniestro, Fecha, Ciudad, Causa (textarea), CUC Concepto (textarea), Cobertura Principal, Consecuencia, Causa Codificada, Intención, Acción, Caso Padre, Observación (textarea)
- `camposBancaria` — Nro Cuenta, Entidad, Cód Banco, Tipo Cuenta, Aprobación Cta
- `camposOpenL` — Sugerencia Final (textarea), Más de un ingreso, Decisión IA, OpenL Cause

**Otras secciones**: `SeccionPanelGenericoComponent` (checklist docs), `SeccionArchivosComponent`, `SeccionAnalisisIaComponent`, `SeccionDecisionComponent`

Soporta modo maximizado.  
TODO pendiente: `guardarDocumentos()`, `confirmarDecision()` → conectar al backend.

```typescript
visible = input(false)
onClose = output<void>()
```

---

## ListaranalisislineaComponent — Línea de Negocio
**Ruta**: `/listar-analisis-linea`  
**Guard**: `authGuard`

Similar a listar-analisis. Reutiliza `ModalAnalisisComponent`.

---

## ListarmisanalisisComponent — Mis Casos
**Ruta**: `/listar-mis-analisis`  
**Guard**: `authGuard`

Usa `MOCK_ANALISIS`. Mismas columnas que listar-analisis. Reutiliza `ModalAnalisisComponent` de `../listar-analisis/modal-analisis/`.

---

## Componentes de Proveedores

Todos siguen el mismo patrón: tabla con `MOCK_*` + modal propio.

### ListaruifaComponent — UIFA
**Ruta**: `/listar-uifa` | **Guard**: `authGuard`  
Usa `MOCK_UIFA`. Modal: `ModalUifaComponent`.

### ListarmedicoComponent — Médico
**Ruta**: `/listar-medico` | **Guard**: `authGuard`  
Usa `MOCK_MEDICO`. Modal: `ModalMedicoComponent`.

### ListarinvestigadorComponent — Investigador
**Ruta**: `/listar-investigador` | **Guard**: `authGuard`  
Usa `MOCK_INVESTIGADOR`. Modal: `ModalInvestigadorComponent`.

### ListartecnicoComponent — Técnico
**Ruta**: `/listar-tecnico` | **Guard**: `authGuard`  
Usa `MOCK_TECNICO`. Modal: `ModalTecnicoComponent`.

---

## ListarpagosComponent — Pagos y Objeciones
**Ruta**: `/listar-pagos`  
**Guards**: `authGuard`, `roleGuard` (solo `admin`)  
**Archivo**: `features/indem-clever/components/listar-pagos/listar-pagos.component.ts`

Vista unificada de Órdenes de Pago + Objeciones. Usa `MOCK_DECISIONES`.

Columnas: ID, Tipo (tag: Pago/Objeción), Fecha, Cobertura, Póliza, Total (currency).

Acciones por fila:
- `info` → `ModalPagoComponent`
- `aprobar` → `DialogoConfirmacionComponent` (severity: success)
- `devolver` → `DialogoConfirmacionComponent` (severity: warn)

Acciones masivas: `confirmarMasivo('aprobar')`, `confirmarMasivo('devolver')`.

TODO pendiente: `onConfirmar()` → conectar al backend.

---

## ListarcasosComponent — Consultar Casos
**Ruta**: `/listar-casos`  
**Guard**: `authGuard`

Datos inline (mock hardcodeado). Columnas: ID, Documento, F. Aviso, Estado (tag).  
Acción: `ver` → `ModalCasoComponent`.

### ModalCasoComponent
**Selector**: `app-modal-caso`  
**Archivo**: `listar-casos/modal-caso/modal-caso.component.ts`

Modal de seguimiento de caso (solo lectura). Contiene:
- Panel de datos estructurado (`bloquesInfo`: Datos Asegurado, Info Radicado, Info Siniestro)
- `SeccionLineaTiempoComponent` con etapas del historial
- Resumen de tiempos por etapa y por responsable
- Sección de documentos adjuntos

TODO pendiente: `generarReporte()` → PDF.

```typescript
visible = input(false)
onClose = output<void>()
// bloquesInfo: BloqueInfo[] — datos del caso (mock)
// etapas: signal<EtapaLineaTiempo[]> — historial
// tiemposPorEtapa / tiemposPorResponsable: signals
```

---

## ListarusuariosComponent — Usuarios
**Ruta**: `/listar-usuarios`  
**Guards**: `authGuard`, `roleGuard` (solo `admin`)

Placeholder vacío. `data = signal<unknown[]>([])`. Sin modal aún.

---

## Páginas de Error

### AccesoDenegadoComponent (403)
Template inline. Muestra ícono lock + mensaje + botón "Volver al inicio".

### NoEncontradoComponent (404)
Template inline. Muestra ícono + mensaje + botón "Volver al inicio".

---
inclusion: always
---

# Modelos de Dominio — Clever Indem Frontend

Todos los modelos están en `src/app/core/models/` y se exportan desde `index.ts`.

## Usuario y Autenticación

```typescript
// usuario.model.ts
type RolUsuario = 'admin' | 'gestor';

interface Usuario {
  id: string;
  nombre: string;
  correo: string;
  rol: RolUsuario;
  avatar?: string;
}
// admin: líder, aprueba pagos, gestiona usuarios
// gestor: radicador, analista, proveedor
```

## Radicación

```typescript
// radicacion.model.ts

// Para la tabla de listado (GET /radicaciones — datos resumidos)
interface RadicacionResumen {
  idRadicado: string;
  numeroPoliza: string;
  fechaAviso: string;
  decision: string;
  cobertura: string;
  tipoPoliza: string;
  estado: string;
}

// Para el detalle completo de un caso
interface Radicacion {
  idRadicado: string;
  tipoDocumento: string;
  numeroDocumento: string;
  nombreAsegurado: string;
  edad: number;
  correo: string;
  telefono: string;
  numeroPoliza: string;
  codigoProducto: string;
  numeroRiesgo: string;
  fechaAviso: string;
  cobertura: string;
  codCobertura: string;
  portafolioCodigo: string;
  portafolioNombre: string;
  valorAsegurado: number;
  scoreCliente: string;
  estado: string;
  fechaCreacion: string;
  // Siniestro
  numeroSiniestro: string;
  fechaSiniestro: string;
  ciudadOcurrencia: string;
  causa: string;
  consecuencia: string;
  // Bancaria
  numeroCuenta: string;
  entidadBancaria: string;
  tipoCuenta: string;
}
```

## Análisis

```typescript
// analisis.model.ts
// Usado por: listar-analisis, listar-analisis-linea, listar-mis-analisis
interface Analisis {
  idRadicado: string;
  documento: string;
  nombreAsegurado: string;
  fechaAviso: string;
  estado: string;
  analista: string;
  prioridad: string;
  lineaNegocio: string;
  decision?: string;
  causal?: string;
  observacionAnalista?: string;
}
```

## Proveedor

```typescript
// proveedor.model.ts
// Usado por: listar-uifa, listar-medico, listar-investigador, listar-tecnico
type TipoProveedor = 'uifa' | 'medico' | 'investigador' | 'tecnico';

interface CasoProveedor {
  idRadicado: string;
  documento: string;
  nombreAsegurado: string;
  fechaAviso: string;
  estado: string;
  tipoProveedor: TipoProveedor;
  agenteAsignado: string;
  motivoMovilizacion: string;
  fechaAsignacion: string;
  observacionProveedor?: string;
}
```

## Orden de Pago / Objeción

```typescript
// orden-pago.model.ts
// Usado por: listar-pagos
interface OrdenPago {
  idRadicado: string;
  documento: string;
  nombreAsegurado: string;
  valorPago: number;
  estado: string;
  fechaCreacion: string;
  aprobadoPor?: string;
  fechaAprobacion?: string;
}

interface Objecion {
  idRadicado: string;
  documento: string;
  nombreAsegurado: string;
  valorObjecion: number;
  estado: string;
  fechaCreacion: string;
  motivoObjecion?: string;
}
```

## Caso Consulta / Historial

```typescript
// caso-consulta.model.ts
// Usado por: listar-casos, modal-caso
interface CasoConsulta {
  idRadicado: string;
  documento: string;
  nombreAsegurado: string;
  fechaAviso: string;
  estado: string;
  numeroPoliza: string;
  cobertura: string;
  fechaSiniestro: string;
}

// Gestión dentro de una etapa de historial
interface GestionHistorial {
  titulo: string;
  responsable: string;
  fecha: string;
  observacion: string;
}

// Etapa del historial de un caso
interface EtapaHistorial {
  label: string;
  completada: boolean;
  activa: boolean;
  gestiones: GestionHistorial[];
}

// Tiempo por etapa o responsable
interface TiempoResumen {
  etapa?: string;
  usuario?: string;
  tiempo: string;
}
```

## Respuestas del Backend

```typescript
// api-response.model.ts
interface ApiResponse<T> {
  success: boolean;
  message: string;
  data: T;
}

interface PaginatedData<T> {
  content: T[];
  page: number;
  size: number;
  totalElements: number;
  totalPages: number;
}

// Ejemplo de uso:
// GET /radicaciones → ApiResponse<PaginatedData<RadicacionResumen>>
// RadicacionesService extrae → PaginatedData<RadicacionResumen>
```

## Interfaces de Componentes Compartidos

```typescript
// tabla-dinamica.component.ts
interface ColumnaTabla {
  field: string;         // nombre del campo en el objeto de datos
  header: string;        // título de la columna
  type?: 'text' | 'date' | 'currency' | 'tag' | 'boolean';
  sortable?: boolean;
  filterable?: boolean;
  width?: string;
  tagMap?: Record<string, { label: string; severity: string }>;
  // severity values: 'success' | 'danger' | 'warn' | 'info' | 'secondary'
}

interface FilaSeleccionada<T = unknown> {
  data: T;
  index: number;
}

interface AccionFila<T = unknown> {
  action: string;   // identificador de la acción (ej: 'ver', 'aprobar', 'devolver')
  data: T;
  index: number;
}

// seccion-formulario-dinamico.component.ts
interface CampoFormulario {
  key: string;
  label: string;
  type?: 'text' | 'textarea' | 'email' | 'tel' | 'date' | 'number';
  readonly?: boolean;
  rows?: number;
  fullWidth?: boolean;  // ocupa todo el ancho en lugar de la grilla de 3 columnas
}

// seccion-linea-tiempo.component.ts
interface GestionEtapa {
  titulo: string;
  responsable: string;
  fecha: string;
  observacion: string;
}

interface EtapaLineaTiempo {
  label: string;
  completada?: boolean;
  activa?: boolean;
  gestiones?: GestionEtapa[];
}
```

## Notificaciones

```typescript
// notificacion.service.ts
type TipoNotificacion = 'success' | 'error' | 'warning' | 'info';

interface Notificacion {
  id: number;
  tipo: TipoNotificacion;
  titulo: string;
  detalle: string;
}
```

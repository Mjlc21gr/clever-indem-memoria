---
inclusion: always
---

# Modelo de dominio completo

## Entidad: Caso (`caso/domain/Caso.java`)
POJO con Lombok `@Data @Builder @NoArgsConstructor @AllArgsConstructor`. Sin anotaciones de Spring.

### Campos completos
```java
String id                           // @NotBlank en Request
String threadId                     // @NotBlank en Request (snake: thread_id)
String casoPadre                    // (snake: caso_padre)
String correoElectronico            // (snake: correo_electronico)
String nombres                      // @NotBlank
String apellidos                    // @NotBlank
String tipoDocumento                // @NotBlank (snake: tipo_documento) — obligatorio en validate()
String numeroDocumento              // @NotBlank (snake: numero_documento) — obligatorio en validate()
String telefono
Boolean aprobacionTerminos          // @NotNull (snake: aprobacion_terminos)
String causa
String consecuencia
LocalDate fechaEvento               // viene como String en Request, parseado por mapper (snake: fecha_evento)
String numPoliza                    // (snake: num_poliza)
String codigoProducto               // (snake: codigo_producto)
String cucConcepto                  // (snake: cuc_concepto)
String cobertura
String estadoFormulario             // (snake: estado_formulario)
String numeroCuentaBancaria         // (snake: numero_cuenta_bancaria)
String entidadBancaria              // (snake: entidad_bancaria)
String codBanco                     // (snake: cod_banco)
String tipoCuenta                   // (snake: tipo_cuenta)
Boolean aprobacionCuenta            // (snake: aprobacion_cuenta)
LocalDate fechaAviso                // viene como String en Request (snake: fecha_aviso)
String versionSiniestro             // (snake: version_siniestro) — usado como "relato" en IAPayload
String respuestaAnalisis            // (snake: respuesta_analisis)
String ciudadOcurrencia             // (snake: ciudad_ocurrencia)
String nitEmpresa                   // (snake: nit_empresa)
String nombreEmpresa                // (snake: nombre_empresa)
String numeroRiesgo                 // (snake: numero_riesgo)
String radicadoBizagi               // (snake: radicado_bizagi)
String superoPeso                   // (snake: supero_peso)
String causaSiniestro               // (snake: causa_siniestro)
String observacionCausa             // (snake: observacion_causa)
String consecuenciaCoberturaPrincipal   // (snake: consecuencia_cobertura_principal)
String consecuenciaCoberturaIncapacidad // (snake: consecuencia_cobertura_incapacidad)
String observacion
String scoreCliente                 // (snake: score_cliente)
String valorAsegurado               // (snake: valor_asegurado)
String codigoCobertura              // (snake: codigo_cobertura)
String nivelRiesgoCliente           // (snake: nivel_riesgo_cliente)
String portafolioCodigoProducto     // (snake: portafolio_codigo_producto)
String portafolioNombreProducto     // (snake: portafolio_nombre_producto)
String valorAseguradoVida           // (snake: valor_asegurado_vida)
String valorAseguradoItp            // (snake: valor_asegurado_itp)
String accion
String coberturaPrincipal           // (snake: cobertura_principal)
String masDeUnIngreso               // (snake: mas_de_un_ingreso)
List<DocumentoAdjunto> documentosAdjuntos  // (snake: documentos_adjuntos)
String sugerencia
EstadoProcesamientoIA estadoProcesamientoIA  // (snake: estado_procesamiento_ia) — en Response como .name()
Instant fechaUltimoIntentoIA
int intentosFallidosIA              // (snake: intentos_fallidos_ia)
```

### Métodos de negocio
```java
// Valida tipoDocumento, numeroDocumento, nombres, apellidos. Lanza InvalidCasoException si vacíos.
public void validate()

// True si documentosAdjuntos != null && !isEmpty()
public boolean tieneDocumentacionCompleta()

// Normaliza "N/A" (case-insensitive, trim) a null; otros valores → trim. Static.
public static String normalizeNA(String value)
```

### Campos obligatorios para validate()
`tipoDocumento`, `numeroDocumento`, `nombres`, `apellidos`

### Campos obligatorios para IAPayload
`threadId`, `tipoDocumento`, `numeroDocumento` — validados en `IAPayloadMapper.validateMandatoryFields()`

---

## Entidad: DocumentoAdjunto (`caso/domain/DocumentoAdjunto.java`)
```java
@Data @Builder @NoArgsConstructor @AllArgsConstructor
String nombreDocumento   // nombre del documento
String observacion       // ATENCIÓN: este campo es la URL del documento en el contexto del IAPayload
```
> En `IAPayloadMapper`: `listaUrlsDocumentos` se extrae de `observacion` y `listaNombresDocumentos` de `nombreDocumento`.

---

## Entidad: RadicacionModel (`radicacion/domain/RadicacionModel.java`)
```java
@Data @Builder @NoArgsConstructor @AllArgsConstructor
String id
String idRadicado            // identificador único de negocio (ej: "RAD-001"), @Indexed unique en Document
String numeroPoliza          // (ej: "3001-4521"), @Indexed en Document
LocalDate fechaAviso
DecisionRadicacion decision  // enum
Cobertura cobertura          // enum
TipoPoliza tipoPoliza        // enum
EstadoRadicacion estado      // enum
```

---

## Enums (`shared/domain/enums/`)

### EstadoRadicacion
Etapa del proceso de siniestro. El frontend filtra por módulo.
```java
R("Radicación")
A("Análisis")
I("Investigación")
P("Pago")
C("Cerrado")
```
Cada valor tiene `getDescripcion()`.

### DecisionRadicacion
```java
APROBADO, PENDIENTE, RECHAZADO, EN_ANALISIS
```

### EstadoProcesamientoIA
Ciclo de vida del procesamiento IA en `Caso`:
```java
PENDIENTE   // asignado antes de llamar al servicio IA
COMPLETADO  // respuesta exitosa recibida
FALLIDO     // excepción en la llamada al servicio IA
```

### Cobertura
```java
VIDA_GRUPO, ITP, EXEQUIAS, VIDA_INDIVIDUAL, ACCIDENTES_PERSONALES
```

### TipoPoliza
```java
INDIVIDUAL, COLECTIVA
```

---

## Excepciones

### Jerarquía
```
RuntimeException
├── DomainException (shared/domain/)               → HTTP 400
│   └── InvalidCasoException (caso/domain/)        → HTTP 400
├── BusinessException (exception/)                 → HTTP 422
│   └── DuplicateCasoException (exception/)        → HTTP 409
├── ResourceNotFoundException (exception/)         → HTTP 404
├── ExternalApiException (exception/)
└── IAServiceException (exception/)                → HTTP 502
    ├── IAServiceException(message, statusCode)    // errores HTTP del servicio IA
    └── IAServiceException(message, cause)         // errores de conexión → statusCode=503
```

### Mensajes de error estándar
- `DuplicateCasoException`: `"El caso con ID {id} ya se encuentra radicado."`
- `ResourceNotFoundException`: `"Radicación no encontrada: {idRadicado}"` o `"Caso no encontrado: {casoId}"`
- `InvalidCasoException`: `"Campos obligatorios vacíos: tipoDocumento, ..."` o `"Campos obligatorios para procesamiento IA faltantes: ..."`
- `BusinessException`: `"El caso ya fue procesado exitosamente por IA"`

---

## Shared — clases reutilizables

### ApiResponse\<T\> (`shared/adapter/in/web/`)
```java
@Data @Builder @NoArgsConstructor @AllArgsConstructor
boolean success
String message
T data
```

### PaginatedResponse\<T\> (`shared/adapter/in/web/`)
```java
List<T> content
int page           // base 0
int size
long totalElements
int totalPages
```

### PaginatedResult\<T\> (`shared/domain/`)
```java
@Getter @AllArgsConstructor  // inmutable, para no exponer Spring Data Page al dominio
List<T> content
int page
int size
long totalElements
int totalPages
```

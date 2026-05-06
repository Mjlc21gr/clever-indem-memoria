---
inclusion: always
---

# Contratos API REST

## Patrón de rutas (CRÍTICO)
```java
@RequestMapping("${api.paths.prefix}/api/${api.paths.version}/casos")
```
Con los valores de `application.yaml` (`prefix=siniestros`, `version=v1`) el resultado es:
```
/siniestros/api/v1/casos
/siniestros/api/v1/radicaciones
```
> Nota: hay un segmento `/api/` fijo entre el prefix y la version. Nunca omitirlo.

`ApiPathProperties` valida que `version` cumpla el regex `v\d+` (ej: `v1`, `v2`).

---

## Endpoints disponibles

### GET /siniestros/api/v1/casos
Retorna todos los casos. Sin paginación.
```
Response: ApiResponse<List<CasoResponse>>
HTTP 200
```

### POST /siniestros/api/v1/casos
Radica un nuevo caso. Dispara procesamiento IA síncrono.
```
Body: CasoRequest (application/json)
Response: ApiResponse<CasoResponse>
HTTP 201 (CREATED)
```

### POST /siniestros/api/v1/casos/{casoId}/reintentar-ia
Reintenta el procesamiento IA de un caso con estado FALLIDO.
```
PathVariable: casoId (String)
Response: ApiResponse<CasoResponse>
HTTP 200
Falla con 422 si el caso ya está COMPLETADO
Falla con 404 si el caso no existe
```

### GET /siniestros/api/v1/radicaciones
Lista radicaciones con filtros opcionales y paginación.
```
QueryParams:
  estado    (EstadoRadicacion, opcional): R | A | I | P | C
  decision  (DecisionRadicacion, opcional): APROBADO | PENDIENTE | RECHAZADO | EN_ANALISIS
  search    (String, opcional): busca en idRadicado y numeroPoliza (regex case-insensitive)
  page      (int, default=0, min=0)
  size      (int, default=10, min=1, max=100)
Response: ApiResponse<PaginatedResponse<RadicacionResponse>>
HTTP 200
```

### GET /siniestros/api/v1/radicaciones/{idRadicado}
Obtiene una radicación por su ID de negocio (ej: "RAD-001").
```
PathVariable: idRadicado (String)
Response: ApiResponse<RadicacionResponse>
HTTP 200
Falla con 404 si no existe
```

---

## Convención de naming en DTOs

### Clases *Request — campos en snake_case
Los campos de entrada usan `snake_case` porque el JSON viene de sistemas externos:
```java
private String tipo_documento;    // correcto
private String tipoDocumento;     // INCORRECTO
```

### Clases *Response — campos camelCase con @JsonProperty
Los campos en la clase Java son camelCase (para Lombok/MapStruct), pero el JSON de salida
es snake_case mediante `@JsonProperty`:
```java
@JsonProperty("tipo_documento")
private String tipoDocumento;
```
> Excepciones: campos cortos de una sola palabra (ej: `nombres`, `apellidos`, `causa`, `observacion`, `sugerencia`, `accion`) no necesitan `@JsonProperty`.

### Enums en Response
Los enums en Response se devuelven como `String` usando `.name()` (solo para `EstadoProcesamientoIA`).
Los otros enums (`DecisionRadicacion`, `EstadoRadicacion`, `Cobertura`, `TipoPoliza`) se devuelven
directamente como objeto enum (Jackson los serializa como su nombre).

---

## CasoRequest — campos obligatorios (@NotBlank / @NotNull)
```
id                  @NotBlank
thread_id           @NotBlank
nombres             @NotBlank
apellidos           @NotBlank
tipo_documento      @NotBlank
numero_documento    @NotBlank
aprobacion_terminos @NotNull (Boolean)
documentos_adjuntos.nombre_documento  @NotBlank (cada elemento de la lista)
```

---

## CasoResponse — todos los campos
Mismos campos que `Caso` del dominio, todos con `@JsonProperty` en snake_case.
Campos adicionales que NO están en CasoRequest:
```
estado_procesamiento_ia  (String, .name() del enum EstadoProcesamientoIA)
intentos_fallidos_ia     (int)
```

---

## RadicacionResponse — campos
```java
String idRadicado
String numeroPoliza
LocalDate fechaAviso
DecisionRadicacion decision
Cobertura cobertura
TipoPoliza tipoPoliza
EstadoRadicacion estado
```

---

## Formato de respuesta estándar
### Éxito
```json
{ "success": true, "message": "Mensaje descriptivo", "data": { ... } }
```

### Error
```json
{ "success": false, "message": "Descripción del error", "data": null }
```

### Paginado
```json
{
  "success": true,
  "message": "...",
  "data": {
    "content": [...],
    "page": 0,
    "size": 10,
    "totalElements": 25,
    "totalPages": 3
  }
}
```

---

## Tabla de códigos HTTP por excepción
| Excepción | HTTP | Cuándo se lanza |
|-----------|------|-----------------|
| `InvalidCasoException` | 400 | `caso.validate()` falla o faltan campos para IA |
| `DomainException` | 400 | Excepción base de dominio |
| `MethodArgumentNotValidException` | 400 | Bean Validation falla en Request; message = "campo: mensaje; ..." |
| `ResourceNotFoundException` | 404 | Caso o radicación no encontrado por ID |
| `DuplicateCasoException` | 409 | ID de caso ya existe en MongoDB |
| `DuplicateKeyException` (MongoDB) | 409 | Violación de índice único |
| `OptimisticLockingFailureException` | 409 | Conflicto de concurrencia (`@Version` en Document) |
| `BusinessException` | 422 | Regla de negocio violada (ej: reintentar caso COMPLETADO) |
| `IAServiceException` | 502 | Fallo HTTP en llamada al servicio IA |
| `Exception` (genérico) | 500 | Error no controlado; log.error con stacktrace |

> Los errores de IAService solo loguean internamente. El cliente recibe `"Error en el servicio de procesamiento IA"` (mensaje genérico, no el detalle del error externo).

---

## Anotaciones Swagger obligatorias en controllers
```java
@Tag(name = "NombreGrupo", description = "Descripción del grupo")  // en la clase
@Operation(summary = "Descripción corta del endpoint")             // en cada método
// Para endpoints complejos, usar @ApiResponses con @ApiResponse por código HTTP
```

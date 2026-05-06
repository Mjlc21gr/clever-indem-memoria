---
inclusion: always
---

# Integración con servicio de IA externo

## Propósito
Envío del caso al microservicio externo de IA de indemnizaciones para análisis automático del siniestro.
El procesamiento es **síncrono** pero no propaga excepciones al cliente — el caso se persiste con estado FALLIDO.

## Configuración
```yaml
ia.service.url: ${IA_SERVICE_URL:https://ia-indemnizaciones-stg.bolnet.com.co/procesar-siniestro}
```
Inyectado con `@Value("${ia.service.url}")` en `IAProcessingRestAdapter`.

---

## Flujo completo del procesamiento IA

### Al radicar un caso (CasoService.radicarCaso)
```
1. caso.validate()                          → valida campos obligatorios
2. existsById()                             → verifica que no sea duplicado
3. caso.setEstadoProcesamientoIA(PENDIENTE) → estado previo a la llamada
4. caso.setFechaUltimoIntentoIA(now())
5. casoRepository.save(caso)               → primer save
6. try { iaProcessingPort.procesarCaso(saved) }
   → éxito: setRespuestaAnalisis(respuesta), setEstadoProcesamientoIA(COMPLETADO)
   → fallo:  setEstadoProcesamientoIA(FALLIDO), setIntentosFallidosIA(1), log.error
7. saved.setFechaUltimoIntentoIA(now())
8. casoRepository.save(saved)              → segundo save con estado final
```

### Al reintentar (CasoService.reintentarProcesamientoIA)
```
1. findById(casoId)                         → 404 si no existe
2. Si estado == COMPLETADO → BusinessException (422)
3. setEstadoProcesamientoIA(PENDIENTE), setFechaUltimoIntentoIA(now())
4. try { iaProcessingPort.procesarCaso(caso) }
   → éxito: setRespuestaAnalisis, setEstadoProcesamientoIA(COMPLETADO)
   → fallo:  setEstadoProcesamientoIA(FALLIDO), setIntentosFallidosIA(intentos + 1), log.error
5. casoRepository.save(caso)
```

---

## Puerto de dominio: IAProcessingPort
```java
// caso/domain/IAProcessingPort.java
public interface IAProcessingPort {
    String procesarCaso(Caso caso);  // retorna la respuesta cruda del servicio IA (String JSON)
}
```

---

## Adaptador: IAProcessingRestAdapter
```java
// caso/adapter/out/ia/IAProcessingRestAdapter.java
@Component @RequiredArgsConstructor
public class IAProcessingRestAdapter implements IAProcessingPort {
    private final RestTemplate restTemplate;
    private final IAPayloadMapper payloadMapper;

    @Value("${ia.service.url}")
    private String iaServiceUrl;

    @Override
    public String procesarCaso(Caso caso) {
        IAPayload payload = payloadMapper.toPayload(caso);
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<IAPayload> request = new HttpEntity<>(payload, headers);

        try {
            ResponseEntity<String> response = restTemplate.postForEntity(iaServiceUrl, request, String.class);
            return response.getBody();
        } catch (HttpClientErrorException | HttpServerErrorException ex) {
            throw new IAServiceException(ex.getResponseBodyAsString(), ex.getStatusCode().value());
        } catch (ResourceAccessException ex) {
            throw new IAServiceException("Servicio IA no disponible", ex);  // statusCode = 503
        }
    }
}
```

---

## IAPayload — contrato con el servicio externo
```java
// caso/adapter/out/ia/IAPayload.java  — campos snake_case con @JsonProperty
@Data @Builder
@JsonProperty("thread_id")              String threadId
@JsonProperty("tipo_documento")         String tipoDocumento
@JsonProperty("numero_documento")       String numeroDocumento
@JsonProperty("fecha_siniestro")        String fechaSiniestro   // LocalDate.toString() → "yyyy-MM-dd"
                                        String relato           // = caso.getVersionSiniestro()
@JsonProperty("lista_urls_documentos")  List<String> listaUrlsDocumentos   // = DocumentoAdjunto.observacion
@JsonProperty("lista_nombres_documentos") List<String> listaNombresDocumentos  // = DocumentoAdjunto.nombreDocumento
                                        String origen           // SIEMPRE = "Via Web"
```

---

## IAPayloadMapper — lógica de transformación
```java
// caso/adapter/out/ia/IAPayloadMapper.java — @Component (no es interfaz MapStruct, es clase manual)
public IAPayload toPayload(Caso caso):
    1. validateMandatoryFields(caso): verifica threadId, tipoDocumento, numeroDocumento
       → lanza InvalidCasoException si alguno está en blanco
    2. listaUrlsDocumentos    = documentosAdjuntos.stream().map(d -> d.getObservacion())
    3. listaNombresDocumentos = documentosAdjuntos.stream().map(d -> d.getNombreDocumento())
    4. relato = caso.getVersionSiniestro()
    5. fechaSiniestro = (caso.getFechaEvento() != null) ? caso.getFechaEvento().toString() : null
    6. origen = "Via Web"  (hardcoded)
```
> IMPORTANTE: `DocumentoAdjunto.observacion` actúa como URL del documento. Esto es una convención implícita del negocio.

---

## Estados del ciclo IA en Caso
| Estado | Cuándo | intentosFallidosIA |
|--------|--------|-------------------|
| `PENDIENTE` | Justo antes de llamar al servicio | 0 (primer intento) |
| `COMPLETADO` | Respuesta exitosa del servicio | 0 |
| `FALLIDO` | Excepción en la llamada | 1 (primer intento) / +1 (reintentos) |

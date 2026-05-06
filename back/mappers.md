---
inclusion: always
---

# Mappers: convenciones y patrones

## Tipos de mappers en el proyecto
Hay dos tipos de mappers y se usan en contextos diferentes:

| Tipo | Cuándo usarlo | Ejemplo |
|------|--------------|---------|
| Interfaz MapStruct (`@Mapper`) | Mapeos campo a campo con conversiones simples | `CasoRequestMapper`, `CasoPersistenceMapper` |
| Clase manual (`@Component`) | Lógica de mapeo compleja, validaciones, transformaciones | `CasoResponseMapper`, `IAPayloadMapper` |

---

## MapStruct — configuración obligatoria

### Annotation processors en build.gradle (ya configurados)
```groovy
implementation 'org.mapstruct:mapstruct:1.5.5.Final'
annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.5.Final'
compileOnly 'org.projectlombok:lombok'
annotationProcessor 'org.projectlombok:lombok'
annotationProcessor 'org.projectlombok:lombok-mapstruct-binding:0.2.0'  // OBLIGATORIO
```

### Declaración siempre con componentModel = "spring"
```java
@Mapper(componentModel = "spring")
public interface MiMapper { ... }
```

---

## CasoRequestMapper — snake_case → camelCase + transformaciones
Convierte `CasoRequest` (snake_case) → `Caso` (camelCase domain).

### Reglas de mapeo
1. Todos los campos String usan `qualifiedByName = "normalizeNA"` (convierte "N/A" a null)
2. Los campos `fecha_evento` y `fecha_aviso` usan `qualifiedByName = "parseDate"` (String → LocalDate)
3. Los campos Boolean (`aprobacion_terminos`, `aprobacion_cuenta`) se mapean directamente

### Métodos @Named
```java
@Named("normalizeNA")
default String normalizeNA(String value) {
    return Caso.normalizeNA(value);  // delega al método estático del dominio
}

@Named("parseDate")
default LocalDate parseDate(String date) {
    String normalized = Caso.normalizeNA(date);
    if (normalized == null) return null;
    try { return LocalDate.parse(normalized); }
    catch (DateTimeParseException e) { return null; }  // fechas inválidas → null, no excepción
}
```

### Regla de nombre de campo
El `source` en `@Mapping` usa el nombre del campo en `CasoRequest` (snake_case):
```java
@Mapping(target = "tipoDocumento", source = "tipo_documento", qualifiedByName = "normalizeNA")
```

---

## CasoPersistenceMapper — Caso ↔ CasoDocument
```java
@Mapper(componentModel = "spring")
public interface CasoPersistenceMapper {
    @Mapping(target = "version", ignore = true)
    @Mapping(target = "fechaCreacion", ignore = true)
    @Mapping(target = "fechaModificacion", ignore = true)
    CasoDocument toDocument(Caso domain);

    Caso toDomain(CasoDocument document);

    DocumentoAdjuntoDocument toDocument(DocumentoAdjunto domain);
    DocumentoAdjunto toDomain(DocumentoAdjuntoDocument document);
}
```
Siempre ignorar `version`, `fechaCreacion`, `fechaModificacion` en `toDocument()`.

---

## CasoResponseMapper — clase manual (@Component)
Construye `CasoResponse` desde `Caso` usando el Builder de Lombok.
```java
@Component
public class CasoResponseMapper {
    public CasoResponse toResponse(Caso caso) {
        return CasoResponse.builder()
            // ... todos los campos
            .estadoProcesamientoIA(caso.getEstadoProcesamientoIA() != null
                ? caso.getEstadoProcesamientoIA().name()   // enum → String
                : null)
            .intentosFallidosIA(caso.getIntentosFallidosIA())
            .documentosAdjuntos(mapDocumentos(caso.getDocumentosAdjuntos()))
            .build();
    }

    private List<DocumentoAdjuntoResponse> mapDocumentos(List<DocumentoAdjunto> documentos) {
        if (documentos == null) return Collections.emptyList();
        return documentos.stream()
            .map(d -> DocumentoAdjuntoResponse.builder()
                .nombreDocumento(d.getNombreDocumento())
                .observacion(d.getObservacion())
                .build())
            .collect(Collectors.toList());
    }
}
```

---

## IAPayloadMapper — clase manual (@Component)
Ver detalle completo en `ia-integration.md`. Puntos clave:
- Valida campos obligatorios ANTES de mapear (lanza `InvalidCasoException`)
- `observacion` de `DocumentoAdjunto` → `listaUrlsDocumentos`
- `nombreDocumento` de `DocumentoAdjunto` → `listaNombresDocumentos`
- `origen` siempre = `"Via Web"` (hardcoded)

---

## Patrón para mappers de Response en contexto radicacion/
En `RadicacionController` el mapeo `RadicacionModel → RadicacionResponse` se hace
con un método privado inline en el controller (no hay clase mapper separada):
```java
private RadicacionResponse toResponse(RadicacionModel model) {
    return RadicacionResponse.builder()
        .idRadicado(model.getIdRadicado())
        // ...
        .build();
}
```
Esto es válido para responses simples. Para responses complejas, crear clase `*ResponseMapper`.

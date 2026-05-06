---
inclusion: always
---

# Persistencia: MongoDB

## Configuración
```yaml
# application.yaml
spring.data.mongodb.uri: ${SPRING_DATA_MONGODB_URI:mongodb://localhost:27017/siniestros_db}
spring.data.mongodb.auto-index-creation: true

# application-local.yaml
spring.data.mongodb.database: siniestros_db
de.flapdoodle.mongodb.embedded.version: 4.0.2
```
- **Local/Test**: Flapdoodle embed automático (dependency `de.flapdoodle.embed.mongo.spring3x:4.12.3`)
- **Prod**: URI completa via `SPRING_DATA_MONGODB_URI`

---

## Colecciones y sus Documents

### Colección `casos` → `CasoDocument`
```java
@Document(collection = "casos")
// Campos con @Field("snake_case") para todos los campos multi-palabra
@Id                  String id            // ID de negocio (viene del request, no autogenerado)
@Version             Long version         // control de concurrencia optimista
@CreatedDate         Instant fechaCreacion       // auditoria automática
@LastModifiedDate    Instant fechaModificacion   // auditoría automática
// ... todos los campos de Caso con @Field("snake_case")
@Field("estado_procesamiento_ia") String estadoProcesamientoIA  // POJO String, no enum
@Field("fecha_ultimo_intento_ia") Instant fechaUltimoIntentoIA
@Field("intentos_fallidos_ia")    int intentosFallidosIA
@Field("documentos_adjuntos")     List<DocumentoAdjuntoDocument> documentosAdjuntos
```

### DocumentoAdjuntoDocument (embebido en CasoDocument)
```java
// Sin @Document, embebido
String nombreDocumento
String observacion    // contiene la URL del documento
```

### Colección `radicaciones` → `RadicacionDocument`
```java
@Document(collection = "radicaciones")
@CompoundIndex(name = "idx_estado_decision", def = "{'estado': 1, 'decision': 1}")
@Getter @Setter @NoArgsConstructor   // NO usa @Data ni @Builder — usa setters manuales al mapear
@Id                    String id
@Version               Long version
@Indexed(unique=true)  @Field("id_radicado")    String idRadicado
@Indexed               @Field("numero_poliza")  String numeroPoliza
                       @Field("fecha_aviso")    LocalDate fechaAviso
                       @Field("decision")       DecisionRadicacion decision  // enum directo
                       @Field("cobertura")      Cobertura cobertura          // enum directo
                       @Field("tipo_poliza")    TipoPoliza tipoPoliza        // enum directo
@Indexed               @Field("estado")         EstadoRadicacion estado      // enum directo
@CreatedDate           @Field("fecha_creacion") Instant fechaCreacion
@LastModifiedDate      @Field("fecha_modificacion") Instant fechaModificacion
```
> Diferencia con CasoDocument: RadicacionDocument almacena enums directamente; CasoDocument almacena `EstadoProcesamientoIA` como String.

---

## Patrón repositorio hexagonal (SEGUIR SIEMPRE ESTE PATRÓN)

### 1. Interfaz en domain/ (puerto de salida)
```java
// caso/domain/CasoRepository.java
public interface CasoRepository {
    Caso save(Caso caso);
    boolean existsById(String id);
    List<Caso> findAll();
    Optional<Caso> findById(String id);
}
```

### 2. Spring Data repository (interno al adapter, solo extiende MongoRepository)
```java
// caso/adapter/out/persistence/SpringDataCasoRepository.java
@Repository
public interface SpringDataCasoRepository extends MongoRepository<CasoDocument, String> {
    // Solo métodos derivados o @Query simples
}
```

### 3. MapStruct mapper de persistencia
```java
// caso/adapter/out/persistence/CasoPersistenceMapper.java
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
> `version`, `fechaCreacion`, `fechaModificacion` SIEMPRE se ignoran en `toDocument()` — los gestiona Spring Data.

### 4. Implementación del puerto (adapter)
```java
// caso/adapter/out/persistence/MongoCasoRepository.java
@Repository
@RequiredArgsConstructor
public class MongoCasoRepository implements CasoRepository {
    private final SpringDataCasoRepository springRepo;
    private final CasoPersistenceMapper mapper;

    @Override
    public Caso save(Caso caso) {
        return mapper.toDomain(springRepo.save(mapper.toDocument(caso)));
    }
    // ...
}
```

---

## MongoTemplate para consultas complejas
Cuando la consulta requiere filtros dinámicos o paginación custom, usar `MongoTemplate` en el adapter:
```java
// Patrón usado en MongoRadicacionRepository
Query query = new Query().with(pageable);
List<Criteria> criteriaList = new ArrayList<>();

if (StringUtils.hasText(estado)) criteriaList.add(Criteria.where("estado").is(estado));
if (StringUtils.hasText(search)) criteriaList.add(new Criteria().orOperator(
    Criteria.where("id_radicado").regex(search, "i"),
    Criteria.where("numero_poliza").regex(search, "i")));

if (!criteriaList.isEmpty()) {
    Criteria finalCriteria = new Criteria().andOperator(criteriaList.toArray(new Criteria[0]));
    query.addCriteria(finalCriteria);
    countQuery.addCriteria(finalCriteria);  // query separado para el count total
}

long total = mongoTemplate.count(countQuery, RadicacionDocument.class);
List<RadicacionDocument> docs = mongoTemplate.find(query, RadicacionDocument.class);
int totalPages = (size > 0) ? (int) Math.ceil((double) total / size) : 0;
return new PaginatedResult<>(content, page, size, total, totalPages);
```
> Los campos en `Criteria.where()` usan el nombre del campo MongoDB (snake_case), no el nombre Java.

---

## Mapeo manual vs MapStruct en persistence adapter
- `CasoDocument` → usa MapStruct (`CasoPersistenceMapper`) porque tiene muchos campos
- `RadicacionDocument` → usa mapeo manual en `MongoRadicacionRepository` (métodos privados `toDomain`/`toDocument`)
  porque el Document usa `@Getter @Setter` sin `@Builder`, y el mapper construye el objeto con setters directamente.

---

## Control de concurrencia
Todos los Documents tienen `@Version Long version`.
Spring Data incrementa automáticamente `version` en cada `save()`.
Si dos operaciones concurrentes modifican el mismo documento → `OptimisticLockingFailureException` → HTTP 409.

---

## Auditoría automática
`@CreatedDate` y `@LastModifiedDate` son gestionados automáticamente por Spring Data MongoDB.
Nunca se asignan manualmente ni se pasan en los mappers (siempre `ignore = true` en `toDocument()`).

---

## Repositorios legacy (NO usar como referencia)
- `repository/RadicacionRepository.java` + `RadicacionRepositoryCustom` + `RadicacionRepositoryCustomImpl`
  Pertenecen al paquete legacy. Todo código nuevo debe seguir el patrón hexagonal de `caso/`.

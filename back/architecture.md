---
inclusion: always
---

# Arquitectura: Hexagonal (Puertos y Adaptadores)

## Stack tecnológico
- Java 21, Spring Boot 3.3.12, Gradle
- MongoDB (Flapdoodle embed en local/test, URI externa en prod)
- MapStruct 1.5.5 + Lombok + `lombok-mapstruct-binding:0.2.0`
- Springdoc OpenAPI 2.6.0 (Swagger UI)
- JUnit 5, Mockito, jqwik, ArchUnit, JaCoCo

## Paquete base
`com.segurosbolivar.siniestros`

## Bounded contexts (código nuevo SIEMPRE aquí)
```
caso/
├── adapter/
│   ├── in/web/          → controllers, *Request, *Response, *Mapper (web)
│   └── out/
│       ├── persistence/ → *Document, *PersistenceMapper, Mongo*Repository, SpringData*Repository
│       └── ia/          → *Payload, *PayloadMapper, *RestAdapter
├── application/         → *Service (POJO puro, sin @Service)
└── domain/              → entidades, interfaces de puerto, excepciones de dominio

radicacion/              → misma estructura que caso/
```

## Paquetes legacy (NO agregar código nuevo)
`controller/`, `service/`, `repository/`, `domain/` — migración pendiente a bounded contexts.

## Regla #1 — Servicios de aplicación NO llevan @Service
Los servicios en `application/` son POJOs puros. Se registran como `@Bean` en `config/BeanConfiguration`:
```java
@Bean
public CasoService casoService(CasoRepository casoRepository, IAProcessingPort iaProcessingPort) {
    return new CasoService(casoRepository, iaProcessingPort);
}
```
Esto mantiene `application/` 100% libre de Spring.

## Regla #2 — domain/ es puro Java, sin Spring
Las entidades de dominio usan SOLO Lombok (`@Data`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`).
Sin `@Document`, sin `@Entity`, sin `@Component`, sin ninguna anotación de Spring o persistencia.

## Regla #3 — Dominio rico, no anémico
Las entidades de dominio contienen lógica de negocio:
- `Caso.validate()` → valida campos obligatorios, lanza `InvalidCasoException`
- `Caso.tieneDocumentacionCompleta()` → regla sobre documentos adjuntos
- `Caso.normalizeNA(String)` → normaliza "N/A" (case-insensitive, con espacios) a `null`

## Regla #4 — Inyección siempre por constructor
Usar `@RequiredArgsConstructor` de Lombok + campos `final`. Nunca `@Autowired`.

## Regla #5 — Controllers nunca exponen entidades de dominio
El flujo es siempre:
```
Request (DTO) → Mapper → Dominio → Service → Dominio → Mapper → Response (DTO)
```

## Regla #6 — Puertos (interfaces) en domain/, implementaciones en adapter/
```java
// En domain/: define el contrato
public interface CasoRepository { ... }
public interface IAProcessingPort { ... }

// En adapter/out/: implementa el contrato
@Repository
public class MongoCasoRepository implements CasoRepository { ... }

@Component
public class IAProcessingRestAdapter implements IAProcessingPort { ... }
```

## Shared kernel
Código compartido entre bounded contexts en `shared/`:
- `shared/domain/` → POJOs y enums de dominio (`DomainException`, `PaginatedResult`, enums)
- `shared/adapter/in/web/` → `ApiResponse<T>`, `PaginatedResponse<T>`

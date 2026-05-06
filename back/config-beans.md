---
inclusion: always
---

# Configuración y beans de Spring

## BeanConfiguration — el hub de inyección de dependencias
```java
// config/BeanConfiguration.java
@Configuration
public class BeanConfiguration {

    @Bean
    public CasoService casoService(CasoRepository casoRepository, IAProcessingPort iaProcessingPort) {
        return new CasoService(casoRepository, iaProcessingPort);
    }

    @Bean
    public RadicacionService radicacionService(RadicacionRepository radicacionRepository) {
        return new RadicacionService(radicacionRepository);
    }
}
```
**Regla:** Todo servicio de `application/` nuevo DEBE registrarse aquí como `@Bean`.
Los adapters (`@Repository`, `@Component`) sí usan anotaciones Spring directamente.

---

## ApiPathProperties — rutas centralizadas
```java
@Configuration
@ConfigurationProperties(prefix = "api.paths")
@Validated
public class ApiPathProperties {
    @NotBlank
    private String prefix = "siniestros";   // default

    @NotBlank
    @Pattern(regexp = "v\\d+")              // valida formato: v1, v2, etc.
    private String version = "v1";          // default
}
```
Usado en `@RequestMapping` de todos los controllers:
```java
@RequestMapping("${api.paths.prefix}/api/${api.paths.version}/casos")
```

---

## RestTemplateConfig
```java
@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();  // sin timeouts custom actualmente
    }
}
```

---

## CorsConfig
`config/CorsConfig.java` — configura políticas CORS para los controllers REST.

---

## MongoConfig
`config/MongoConfig.java` — customización del cliente MongoDB (conversores de tipos, etc.).

---

## SwaggerConfig
`config/SwaggerConfig.java` — configuración de OpenAPI 3 (info del servicio, servidores por entorno).
Swagger UI disponible en `/swagger-ui.html` cuando la app está corriendo.

---

## TransactionConfig
`config/TransactionConfig.java` — expone `TransactionTemplate` como `@Bean` para transacciones programáticas.
Usar cuando la operación requiere lógica transaccional más compleja que un simple `@Transactional`.

---

## DataInitializer — datos de prueba en local
```java
@Configuration
@Profile("local")   // solo activo con perfil "local"
public class DataInitializer {
    @Bean
    public CommandLineRunner initData(RadicacionRepository repository) {
        return args -> {
            if (repository.count() == 0) {  // idempotente: solo inserta si está vacío
                repository.saveAll(List.of(
                    // 5 radicaciones de prueba con distintos estados/decisions/coberturas
                ));
            }
        };
    }
}
```
Solo se ejecuta con `SPRING_PROFILES_ACTIVE=local` (default en desarrollo).

---

## Perfiles de Spring
| Perfil | Activación | Características |
|--------|-----------|-----------------|
| `local` | Por defecto (`SPRING_PROFILES_ACTIVE=local`) | MongoDB embebido Flapdoodle, DataInitializer activo, logging DEBUG |
| `(prod/stg)` | `SPRING_PROFILES_ACTIVE=<env>` | `SPRING_DATA_MONGODB_URI` requerido, `IA_SERVICE_URL` requerido |

---

## Variables de entorno
| Variable | Descripción | Default (local) |
|----------|-------------|-----------------|
| `SPRING_PROFILES_ACTIVE` | Perfil activo | `local` |
| `SPRING_DATA_MONGODB_URI` | URI completa de MongoDB | `mongodb://localhost:27017/siniestros_db` |
| `IA_SERVICE_URL` | URL del servicio externo de IA | `https://ia-indemnizaciones-stg.bolnet.com.co/procesar-siniestro` |

---

## Actuator (monitoreo)
```yaml
management.endpoints.web.exposure.include: health, info, metrics
management.endpoint.health.show-details: always
```
Puerto: 8081 (mismo que la app). Endpoints: `/actuator/health`, `/actuator/info`, `/actuator/metrics`.

---

## Logging en local
```yaml
# application-local.yaml
logging.level.com.segurosbolivar.siniestros: DEBUG
logging.level.org.springframework.data.mongodb.core.MongoTemplate: DEBUG
```

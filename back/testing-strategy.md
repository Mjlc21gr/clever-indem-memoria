---
inclusion: always
---

# Estrategia de pruebas

## Frameworks
| Framework | Versión | Uso |
|-----------|---------|-----|
| JUnit 5 + Spring Boot Test | 3.3.12 | Base para todos los tests |
| Mockito | (incluido en Spring Boot Test) | Mocks en unit tests |
| AssertJ | (incluido en Spring Boot Test) | Assertions fluidas |
| jqwik | 1.9.2 | Property-based testing |
| ArchUnit | 1.3.0 | Validación de reglas de arquitectura en CI |
| JaCoCo | 0.8.13 | Cobertura de código |
| Flapdoodle embed MongoDB | 4.12.3 | MongoDB en memoria para todos los tests |

---

## Patrón por capa

### Tests de Application/Service — unit tests puros
```java
@ExtendWith(MockitoExtension.class)
class CasoServiceTest {

    @Mock
    private CasoRepository casoRepository;

    @Mock
    private IAProcessingPort iaProcessingPort;

    @InjectMocks          // funciona porque CasoService tiene constructor con esos parámetros
    private CasoService casoService;

    @BeforeEach
    void setUp() {
        validCaso = Caso.builder()
            .id("CASO-001")
            .tipoDocumento("CC")
            .numeroDocumento("123456789")
            .nombres("Juan")
            .apellidos("Pérez")
            .threadId("thread-1")
            .build();
    }
}
```
Usa `when(...).thenReturn(...)`, `verify(...)`, `assertThatThrownBy(...)`.
AssertJ: `assertThat(result).isNotNull()`, `.isInstanceOf(X.class)`, `.hasMessageContaining("...")`.

### Tests de Controller — @WebMvcTest
```java
@WebMvcTest(CasoController.class)
class CasoControllerTest {
    @Autowired MockMvc mockMvc;
    @MockBean CasoService casoService;
    @MockBean CasoRequestMapper casoRequestMapper;
    @MockBean CasoResponseMapper casoResponseMapper;
}
```
Valida: código HTTP, cuerpo JSON, headers. No levanta Spring completo.

### Tests de Persistence — con Flapdoodle embed
```java
// Usa el contexto de test completo de Spring Boot con MongoDB embebido
// No mockear la base de datos
class MongoCasoRepositoryTest { ... }
```

### Tests de Arquitectura — ArchUnit
```java
class ApiPathArchitectureTest {
    // Verifica que todos los @RequestMapping usen el patrón correcto de rutas
}
```

### Property-based tests — jqwik
```java
class IAPayloadMapperPropertyTest {
    @Property
    void propiedadQueDebeMantenerse(@ForAll("casoValido") Caso caso) {
        // verifica invariantes con datos generados automáticamente
    }
}
```

---

## Convención de nombres de archivos
| Sufijo | Tipo |
|--------|------|
| `*Test.java` | Unit test con Mockito |
| `*Tests.java` | Test de integración Spring Boot |
| `*PropertyTest.java` | Property-based con jqwik |
| `*IntegrationTest.java` | Integración con contexto completo |
| `*ArchitectureTest.java` | Validación ArchUnit |

---

## Estructura de un test unitario de servicio (referencia: CasoServiceTest)
```java
@Test
@DisplayName("nombreMetodo - escenario: resultado esperado")
void nombreMetodo_escenario() {
    // Arrange
    when(casoRepository.existsById("CASO-001")).thenReturn(false);
    when(casoRepository.save(any(Caso.class))).thenReturn(validCaso);

    // Act
    Caso result = casoService.radicarCaso(validCaso);

    // Assert
    assertThat(result).isNotNull();
    assertThat(result.getId()).isEqualTo("CASO-001");
    verify(casoRepository).existsById("CASO-001");
    verify(casoRepository, times(2)).save(validCaso);  // save se llama dos veces en radicarCaso
}
```

### Patrón para verificar excepciones
```java
assertThatThrownBy(() -> casoService.radicarCaso(invalidCaso))
    .isInstanceOf(InvalidCasoException.class)
    .hasMessageContaining("tipoDocumento")
    .hasMessageContaining("numeroDocumento");

verify(casoRepository, never()).existsById(any());
verify(casoRepository, never()).save(any());
```

---

## Cobertura JaCoCo
- **Mide SOLO** el paquete `**/application/**` (configurado en `build.gradle`)
- Reportes: `build/reports/jacoco/test/html/index.html`
- XML para Sonar: `build/reports/jacoco/test/jacocoTestReport.xml`
- Ejecutar: `./gradlew test jacocoTestReport`

---

## Configuración de test
`src/test/resources/application.yaml` sobreescribe la config principal:
- MongoDB embebido activo automáticamente (Flapdoodle)
- No requiere config adicional para tests con Spring Boot

---

## Datos de test recomendados (basados en DataInitializer)
Para tests con contexto Spring que necesiten datos iniciales:
```java
// IDs de radicaciones disponibles en perfil local
"RAD-001" → APROBADO, VIDA_GRUPO, COLECTIVA, estado R
"RAD-002" → PENDIENTE, ITP, INDIVIDUAL, estado A
"RAD-003" → RECHAZADO, EXEQUIAS, COLECTIVA, estado C
"RAD-004" → EN_ANALISIS, VIDA_INDIVIDUAL, INDIVIDUAL, estado A
"RAD-005" → APROBADO, ACCIDENTES_PERSONALES, COLECTIVA, estado R
```

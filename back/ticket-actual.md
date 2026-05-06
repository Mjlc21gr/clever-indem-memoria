# 🔧 TICKET BACK — Fix schema validation de radicaciones (enum estado incorrecto)

**Problema:**
El `MongoMigration.java` aplica el enum `["RR", "RS", "A", "AU", "AN", "P", "C"]` al campo `estado` de la colección `radicaciones`. Pero esa colección usa el enum `EstadoRadicacion` de Java que tiene valores `R, A, AU, AN, I, P, C`. El DataInitializer falla al insertar con estado `"R"` porque no está en el enum del schema.

**Archivo a modificar:**
- `config/MongoMigration.java` → método `radicacionesSchema()`

**Cambio exacto:**

```java
// Cambiar esta línea:
.append("estado", new Document("enum", List.of("RR", "RS", "A", "AU", "AN", "P", "C")))

// Por:
.append("estado", new Document("enum", List.of("R", "A", "AU", "AN", "I", "P", "C")))
```

**Contexto:**
- `radicaciones.estado` → usa `EstadoRadicacion` enum: `R, A, AU, AN, I, P, C`
- `datos_casos.estado` → usa strings libres: `RR, RS, A, AU, AN, P, C` (este sí está correcto)

**Criterio de aceptación:**
- [ ] `./gradlew bootRun` arranca sin error
- [ ] Los 120 casos se insertan correctamente
- [ ] Build exitoso

**Depende de:** ninguna

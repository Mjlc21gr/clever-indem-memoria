# 🔧 TICKET BACK — Insertar 120 registros adicionales a MongoDB

**Qué hacer:**
Modificar el `DataInitializer` para que inserte **240 casos en total** (los 120 actuales + 120 nuevos) en las 6 colecciones. Los nuevos deben tener IDs `RAD-121` a `RAD-240` y `CLI-121` a `CLI-240`.

**Archivo a modificar:**
- `config/DataInitializer.java`

**Cambio:**
Cambiar el loop de `120` a `240`:
```java
// Cambiar:
for (int i = 0; i < 120; i++) { ... }

// Por:
for (int i = 0; i < 240; i++) { ... }
```

**Distribución de estados (240 casos total):**

| Estado | Cantidad | IDs |
|---|---|---|
| RR | 60 | RAD-001 a RAD-060 |
| RS | 30 | RAD-061 a RAD-090 |
| A | 50 | RAD-091 a RAD-140 |
| AU | 30 | RAD-141 a RAD-170 |
| AN | 30 | RAD-171 a RAD-200 |
| P | 20 | RAD-201 a RAD-220 |
| C | 20 | RAD-221 a RAD-240 |

**Reglas que se mantienen:**
- Casos RR/RS → `numSiniestro = null`
- Casos A, AU, AN, P, C → tienen `numSiniestro`
- Datos IA solo para estados A, AU, P
- 2 documentos por caso en `datos_docs`
- `deleteAll + saveAll` en cada arranque

**Criterio de aceptación:**
- [ ] `./gradlew bootRun` arranca sin error
- [ ] `mongosh siniestros_db --eval "db.radicaciones.countDocuments()"` → 240
- [ ] `mongosh siniestros_db --eval "db.datos_casos.countDocuments()"` → 240
- [ ] `mongosh siniestros_db --eval "db.datos_cliente.countDocuments()"` → 240
- [ ] Build exitoso

**Depende de:** ninguna

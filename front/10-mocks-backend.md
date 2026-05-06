---
inclusion: always
---

# Mock Data y Estado del Backend — Clever Indem Frontend

## Estado actual de conexión al backend

| Módulo | Estado | Servicio |
|---|---|---|
| `listar-radicaciones` | ✅ Conectado | `RadicacionesService.listar()` |
| `listar-mesa-perfect` | 🔴 Mock | `MOCK_RADICACIONES` |
| `listar-analisis` | 🔴 Mock | `MOCK_ANALISIS` |
| `listar-analisis-linea` | 🔴 Mock | `MOCK_ANALISIS` |
| `listar-mis-analisis` | 🔴 Mock | `MOCK_ANALISIS` |
| `listar-uifa` | 🔴 Mock | `MOCK_UIFA` |
| `listar-medico` | 🔴 Mock | `MOCK_MEDICO` |
| `listar-investigador` | 🔴 Mock | `MOCK_INVESTIGADOR` |
| `listar-tecnico` | 🔴 Mock | `MOCK_TECNICO` |
| `listar-pagos` | 🔴 Mock | `MOCK_DECISIONES` |
| `listar-casos` | 🔴 Inline | datos hardcodeados en el componente |
| `listar-usuarios` | 🔴 Sin datos | `signal<unknown[]>([])` |
| Dashboard (inicio) | 🔴 Estático | todo hardcodeado en el componente |
| AuthService | 🔴 Stub | usuario hardcodeado |

---

## Mocks disponibles en `@shared`

### MOCK_RADICACIONES
```typescript
// Archivo: shared/mocks/mock-radicaciones.data.ts
// 12 registros de tipo: { idRadicado, numeroPoliza, fechaAviso, decision, cobertura, tipoPoliza, estado }
// Usado por: listar-mesa-perfect, listar-radicaciones (como fallback)
// Coberturas: 'Vida Grupo', 'ITP', 'Exequias', 'Vida Individual', 'Accidentes Personales'
// Estados: 'Activo', 'En proceso', 'Cerrado', 'Inactivo'
// Decisiones: 'Aprobado', 'Pendiente', 'Rechazado', 'En análisis'
```

### MOCK_ANALISIS
```typescript
// Archivo: shared/mocks/mock-analisis.data.ts
// 10 registros de tipo: { id, nombre, numeroPoliza, fechaAviso, fechaSiniestro, cobertura, decisionIA, tipoPoliza, estado }
// Usado por: listar-analisis, listar-analisis-linea, listar-mis-analisis
```

### MOCK_UIFA, MOCK_MEDICO, MOCK_INVESTIGADOR, MOCK_TECNICO
```typescript
// Archivo: shared/mocks/mock-proveedores.data.ts
// Cada uno: array de CasoProveedor con tipoProveedor correspondiente
```

### MOCK_DECISIONES
```typescript
// Archivo: shared/mocks/mock-pagos.data.ts
// 14 registros: { id, tipo, fecha, numeroSiniestro, cobertura, poliza, total }
// tipo: 'Orden de Pago' | 'Objeción'
// Usado por: listar-pagos

// Aliases deprecados (señalados con @deprecated):
MOCK_ORDENES_PAGO = MOCK_DECISIONES.filter(d => d.tipo === 'Orden de Pago')
MOCK_OBJECIONES   = MOCK_DECISIONES.filter(d => d.tipo === 'Objeción')
// → Usar MOCK_DECISIONES directamente
```

---

## Cómo reemplazar un mock por una llamada real

### Patrón de migración:

**Antes (mock):**
```typescript
export class ListarXxxComponent {
  data = signal<unknown[]>(MOCK_XXX);
}
```

**Después (backend):**
```typescript
@Injectable({ providedIn: 'root' })
export class XxxService {
  private http = inject(HttpClient);
  private readonly baseUrl = `${environment.apiUrl}/xxx`;

  listar(params: XxxParams): Observable<PaginatedData<XxxModel>> {
    return this.http
      .get<ApiResponse<PaginatedData<XxxModel>>>(this.baseUrl, { params: ... })
      .pipe(map(r => r.data));
  }
}

// En el componente:
export class ListarXxxComponent implements OnInit {
  private xxxService = inject(XxxService);
  private notificacion = inject(NotificacionService);

  loading = signal(false);
  data = signal<XxxModel[]>([]);

  ngOnInit(): void { this.cargar(); }

  cargar(): void {
    this.loading.set(true);
    this.xxxService.listar(params).subscribe({
      next: (paginado) => {
        this.data.set(paginado.content);
        this.loading.set(false);
      },
      error: () => {
        this.notificacion.error('No se pudieron cargar los datos');
        this.loading.set(false);
      },
    });
  }
}
```

---

## URL del backend

```
Proxy: /siniestros/api/v1  →  backend real
Endpoint radicaciones: GET /siniestros/api/v1/radicaciones?estado=R&page=0&size=10
```

Respuesta esperada:
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "content": [ ... ],
    "page": 0,
    "size": 10,
    "totalElements": 100,
    "totalPages": 10
  }
}
```

---

## TODOs pendientes de conexión al backend

Marcados con `// TODO:` en el código:

1. **AuthService** (`auth.service.ts`) — `login()`: llamar POST /auth/login con httpOnly cookie
2. **ModalAgregarCasoComponent** — `guardarRadicado()`, `anularCaso()`, `onFileUpload()`
3. **ModalAnalisisComponent** — `guardarDocumentos()`, `confirmarDecision()`
4. **ListarpagosComponent** — `onConfirmar()` (aprobar/devolver)
5. **ModalCasoComponent** — `generarReporte()` (PDF)
6. **ListarusuariosComponent** — todo (sin implementar)
7. **listar-mesa-perfect** — sin modal funcional
8. **Dashboard (inicio)** — conectar KPIs, gráficas y top usuarios al backend
9. **authGuard** — cuando exista página de login, redirigir ahí en vez de `/inicio`

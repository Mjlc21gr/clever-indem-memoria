---
inclusion: always
---

# Servicios, Guards e Interceptores — Clever Indem Frontend

## AuthService
**Archivo**: `core/services/auth.service.ts`  
**Scope**: `providedIn: 'root'`  
**Estado**: STUB — simula usuario autenticado, pendiente conectar al backend real

```typescript
// Señales (signals) públicas
readonly usuario: Signal<Usuario | null>          // usuario actual
readonly isAuthenticated: Signal<boolean>         // si hay usuario
readonly rol: Signal<RolUsuario | null>           // 'admin' | 'gestor' | null
readonly nombreUsuario: Signal<string>            // nombre para topbar/dashboard

// Métodos
hasRole(...roles: RolUsuario[]): boolean          // verifica si el usuario tiene algún rol
login(_correo: string, _password: string): void  // TODO: conectar POST /auth/login
logout(): void                                    // limpia estado

// Usuario stub actual (desarrollo):
// { id: '1', nombre: 'Jorge Morales', correo: 'dev@segurosbolivar.com', rol: 'admin' }
```

**Plan de conexión al backend**:
1. `login()` llama `POST /auth/login` → el backend setea httpOnly cookie con JWT
2. Decodificar JWT para obtener datos del usuario
3. `logout()` llama `POST /auth/logout` → el backend invalida la cookie

---

## NotificacionService
**Archivo**: `core/services/notificacion.service.ts`  
**Scope**: `providedIn: 'root'`

Sistema de toasts (notificaciones transitorias).

```typescript
readonly notificaciones: Signal<Notificacion[]>   // lista actual de notificaciones

// Métodos para mostrar toasts:
exito(detalle: string, titulo?: string): void     // success — 4s
error(detalle: string, titulo?: string): void     // error — 6s
advertencia(detalle: string, titulo?: string): void // warning — 4s
info(detalle: string, titulo?: string): void      // info — 4s
dismiss(id: number): void                         // eliminar por ID
```

Los toasts se auto-eliminan: errores a los 6s, el resto a los 4s.  
Los componentes que muestran los toasts deben escuchar `notificaciones()` e iterar.

---

## RadicacionesService
**Archivo**: `core/services/radicaciones.service.ts`  
**Scope**: `providedIn: 'root'`  
**Estado**: CONECTADO al backend real

```typescript
// URL base: ${environment.apiUrl}/radicaciones = /siniestros/api/v1/radicaciones

interface RadicacionesQueryParams {
  estado: string;    // 'R' = Radicado, 'A' = Activo, etc.
  page: number;      // 0-based
  size: number;      // elementos por página
}

// Método
listar(params: RadicacionesQueryParams): Observable<PaginatedData<RadicacionResumen>>
// GET /radicaciones?estado=R&page=0&size=10
// Respuesta: ApiResponse<PaginatedData<RadicacionResumen>> → extrae .data
```

**Cómo agregar nuevos servicios**:
Seguir el patrón de `RadicacionesService`:
```typescript
@Injectable({ providedIn: 'root' })
export class NuevoService {
  private http = inject(HttpClient);
  private readonly baseUrl = `${environment.apiUrl}/endpoint`;

  listar(params: QueryParams): Observable<PaginatedData<Modelo>> {
    return this.http
      .get<ApiResponse<PaginatedData<Modelo>>>(this.baseUrl, { params: httpParams })
      .pipe(map(response => response.data));
  }
}
```

---

## authInterceptor (HTTP)
**Archivo**: `core/interceptors/auth.interceptor.ts`  
**Registro**: `app.config.ts` → `provideHttpClient(withInterceptors([authInterceptor]))`

Interceptor funcional que maneja errores HTTP globalmente:

| Status | Acción |
|---|---|
| 401 | Toast "Sesión expirada" + navega a `/inicio` |
| 403 | Toast "Sin permisos" + navega a `/acceso-denegado` |
| 0 | Toast "Sin conexión con servidor" |
| 5xx | Toast "Error interno" |
| Otros | Toast con `error.error.detail` o "Error inesperado" |

**Nota**: El JWT (cuando se conecte el backend) vendrá en httpOnly cookie, no en header manual.

---

## authGuard
**Archivo**: `core/guards/auth.guard.ts`  
**Tipo**: `CanActivateFn`

```typescript
// Si isAuthenticated() → permite acceso (return true)
// Si NO autenticado → redirige a /inicio
// TODO: cuando exista página de login, redirigir ahí
```

---

## roleGuard
**Archivo**: `core/guards/role.guard.ts`  
**Tipo**: `CanActivateFn`

```typescript
// Lee route.data.roles: RolUsuario[] de la configuración de ruta
// Si hasRole(...requiredRoles) → permite acceso
// Si NO tiene el rol → redirige a /acceso-denegado

// Configuración en ruta:
{
  path: 'listar-pagos',
  canActivate: [authGuard, roleGuard],
  data: { roles: ['admin'] },
  loadComponent: () => ...
}
```

---

## ApplicationConfig (app.config.ts)

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor])),
    provideAnimations(),
  ],
};
```

---

## Componente Raíz (App)

El `App` (app.ts) gestiona el layout principal:
- `sidebarCollapsed: signal(false)` — collapsed en desktop
- `isMobile: signal(window.innerWidth <= 768)` — breakpoint mobile
- `sidebarMobileOpen: signal(false)` — overlay en mobile
- `userName = computed(() => auth.nombreUsuario())` — nombre del usuario
- `@HostListener('window:resize')` — actualiza estado mobile
- `toggleSidebar()` — toggle colapso (desktop) u overlay (mobile)
- `closeMobileSidebar()` — cierra overlay mobile al seleccionar item

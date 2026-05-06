---
inclusion: always
---

# Routing — Clever Indem Frontend

## Rutas expuestas vía Module Federation

Archivo: `src/app/exposed.routes.ts`

| Path | Componente | Guards | Restricción |
|---|---|---|---|
| `/inicio` | `InicioComponent` | ninguno | pública |
| `/listar-radicaciones` | `ListarradicacionesComponent` | `authGuard` | autenticado |
| `/listar-mesa-perfect` | `ListarmesaperfectComponent` | `authGuard` | autenticado |
| `/listar-analisis` | `ListaranalisisComponent` | `authGuard` | autenticado |
| `/listar-analisis-linea` | `ListaranalisislineaComponent` | `authGuard` | autenticado |
| `/listar-mis-analisis` | `ListarmisanalisisComponent` | `authGuard` | autenticado |
| `/listar-uifa` | `ListaruifaComponent` | `authGuard` | autenticado |
| `/listar-medico` | `ListarmedicoComponent` | `authGuard` | autenticado |
| `/listar-investigador` | `ListarinvestigadorComponent` | `authGuard` | autenticado |
| `/listar-tecnico` | `ListartecnicoComponent` | `authGuard` | autenticado |
| `/listar-pagos` | `ListarpagosComponent` | `authGuard`, `roleGuard` | `admin` |
| `/listar-casos` | `ListarcasosComponent` | `authGuard` | autenticado |
| `/listar-usuarios` | `ListarusuariosComponent` | `authGuard`, `roleGuard` | `admin` |
| `/listar-otros` | redirect → `/listar-pagos` | — | — |
| `` (vacío) | redirect → `/inicio` | — | — |

## Rutas de error (app.routes.ts)

| Path | Componente |
|---|---|
| `/acceso-denegado` | `AccesoDenegadoComponent` |
| `**` (wildcard) | `NoEncontradoComponent` |

## Cómo funcionan los Guards

### authGuard (`core/guards/auth.guard.ts`)
```typescript
// Verifica AuthService.isAuthenticated()
// Si no autenticado → redirige a /inicio
// (TODO: cuando exista login, redirigir a /login)
```

### roleGuard (`core/guards/role.guard.ts`)
```typescript
// Lee route.data.roles: RolUsuario[]
// Verifica AuthService.hasRole(...roles)
// Si no tiene el rol → redirige a /acceso-denegado
// Ejemplo de uso en ruta:
// { path: 'listar-pagos', canActivate: [authGuard, roleGuard], data: { roles: ['admin'] } }
```

## Todos los componentes usan lazy loading

```typescript
loadComponent: () => import('./...component').then(m => m.NombreComponent)
```

## Navegación en SidebarComponent

El sidebar usa `Router.navigate([routerLink])` directamente (no `[routerLink]` en el template) para poder emitir el evento `itemSelected` al mismo tiempo y cerrar el sidebar en mobile.

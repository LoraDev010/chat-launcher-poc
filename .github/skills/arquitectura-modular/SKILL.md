---
name: arquitectura-modular
description: Diseña e implementa arquitectura modular orientada a features para cualquier proyecto. Úsala cuando necesites estructurar un proyecto nuevo, refactorizar un proyecto existente hacia una arquitectura limpia, o decidir dónde colocar nuevos módulos/features. Detecta automáticamente el lenguaje y stack tecnológico y aplica los patrones arquitectónicos más adecuados (Feature-Sliced Design, Clean Architecture, Vertical Slice, etc.). Genera estructura de carpetas, convenciones de nombrado, separación de responsabilidades y guías de extensión.
---

Este skill guía la definición e implementación de arquitectura modular por features. Su objetivo es producir una estructura de proyecto mantenible, escalable y fácil de navegar, independientemente del lenguaje o framework.

El usuario provee: un proyecto existente (para analizar y proponer mejoras) o una descripción de un sistema nuevo (para diseñar desde cero). Puede incluir restricciones de stack, tamaño del equipo o complejidad esperada.

---

## PASO 1 — Detectar Stack y Contexto

Antes de proponer cualquier estructura, explorar el proyecto:

1. **Leer archivos raíz**: `package.json`, `pom.xml`, `build.gradle`, `go.mod`, `Cargo.toml`, `requirements.txt`, `pyproject.toml`, `*.csproj`, `composer.json`
2. **Identificar framework principal**: React, Vue, Angular, Next.js, NestJS, Express, Spring Boot, Django, FastAPI, Laravel, Go stdlib, etc.
3. **Detectar lenguaje**: TypeScript, JavaScript, Python, Java, Go, C#, PHP, Rust, etc.
4. **Evaluar tamaño del proyecto**:
   - **Pequeño** (< 10 features): módulos simples, mínima burocracia
   - **Mediano** (10-30 features): Feature-Sliced o Vertical Slice
   - **Grande** (30+ features / microservicios): Clean Architecture + bounded contexts

> Si no puedes inferir el stack, pregunta al usuario antes de continuar.

---

## PASO 2 — Principios Universales

Aplicar siempre, sin importar el stack:

### Organización por Feature, no por Tipo Técnico

```
# MAL — organización por capa técnica
src/
  controllers/
  services/
  models/
  utils/

# BIEN — organización por feature
src/
  features/
    auth/
    orders/
    products/
    notifications/
  shared/
```

### Reglas de Dependencia
- Los features **no se importan entre sí** directamente
- La comunicación entre features pasa por `shared/` o eventos/mensajes
- Las capas internas no conocen las externas (inversión de dependencias)
- `shared/` solo contiene código genuinamente reutilizable entre features

### Cohesión y Acoplamiento
- Todo lo relacionado con un feature vive **junto** (UI + lógica + tipos + tests)
- Un feature puede eliminarse borrando su carpeta sin romper otros
- Los nombres de archivos reflejan su propósito, no su tipo (`useOrderForm.ts`, no `hook1.ts`)

---

## PASO 3 — Estructuras por Stack

### React / Next.js (TypeScript)

```
src/
  features/
    auth/
      components/         # Componentes exclusivos de este feature
        LoginForm.tsx
        AuthGuard.tsx
      hooks/              # Hooks del feature
        useAuth.ts
        useSession.ts
      services/           # Llamadas a API / lógica de negocio
        authService.ts
      store/              # Estado local del feature (Zustand slice, Redux slice, etc.)
        authStore.ts
      types/              # Tipos e interfaces del feature
        auth.types.ts
      index.ts            # Barrel: exporta solo la API pública del feature
    orders/
      components/
      hooks/
      services/
      types/
      index.ts
  shared/
    components/           # Componentes UI genéricos (Button, Modal, Table)
    hooks/                # Hooks genéricos (useDebounce, usePagination)
    services/             # Cliente HTTP, config de API
    types/                # Tipos globales compartidos
    utils/                # Funciones utilitarias puras
    constants/
  app/                    # Enrutamiento, providers, configuración global
    router.tsx
    App.tsx
    providers.tsx
  pages/                  # (Next.js) Solo orquestación, sin lógica
```

**Convenciones React:**
- Componentes: `PascalCase.tsx`
- Hooks: `camelCase.ts` con prefijo `use`
- Servicios: `camelCase.ts` con sufijo `Service`
- Stores: `camelCase.ts` con sufijo `Store` o `Slice`
- Cada feature exporta solo lo necesario desde su `index.ts`

---

### Node.js / Express / NestJS (TypeScript)

```
src/
  modules/
    auth/
      auth.controller.ts
      auth.service.ts
      auth.repository.ts
      auth.module.ts        # (NestJS)
      dto/
        login.dto.ts
        register.dto.ts
      entities/
        user.entity.ts
      guards/
        jwt.guard.ts
    orders/
      orders.controller.ts
      orders.service.ts
      orders.repository.ts
      dto/
      entities/
  shared/
    database/             # Conexión, migraciones, ORM config
    middleware/
    pipes/
    interceptors/
    exceptions/
    utils/
  config/                 # Variables de entorno, validación de config
  main.ts
```

**Convenciones NestJS:**
- Cada módulo es autónomo y declara sus propias dependencias
- Los repositorios abstraen el acceso a datos (no usar el ORM directamente en servicios)
- DTOs validan entrada; entidades modelan persistencia; no mezclar

---

### Python / FastAPI / Django

```
# FastAPI
app/
  features/
    auth/
      router.py           # Endpoints del feature
      service.py          # Lógica de negocio
      repository.py       # Acceso a datos
      schemas.py          # Pydantic models (request/response)
      models.py           # SQLAlchemy models
    products/
      router.py
      service.py
      repository.py
      schemas.py
      models.py
  shared/
    database.py           # Sesión DB, base declarativa
    dependencies.py       # Dependencias FastAPI reutilizables
    exceptions.py
    utils/
  main.py                 # App factory, registro de routers

# Django — usar apps por feature
myproject/
  apps/
    auth/                 # Django app por feature
      models.py
      views.py
      urls.py
      serializers.py      # (DRF)
      services.py         # Lógica fuera de views
      admin.py
    orders/
  shared/
    mixins/
    permissions/
    utils/
  config/
    settings/
      base.py
      development.py
      production.py
```

---

### Java / Spring Boot

```
src/main/java/com/empresa/app/
  features/
    auth/
      AuthController.java
      AuthService.java
      AuthRepository.java
      dto/
        LoginRequest.java
        LoginResponse.java
      entity/
        User.java
      exception/
        AuthException.java
    orders/
      OrderController.java
      OrderService.java
      OrderRepository.java
      dto/
      entity/
  shared/
    config/               # SecurityConfig, SwaggerConfig, etc.
    exception/            # GlobalExceptionHandler
    utils/
  Application.java
```

---

### Go

```
internal/
  features/
    auth/
      handler.go          # HTTP handlers
      service.go          # Lógica de negocio
      repository.go       # Acceso a datos
      model.go            # Structs del dominio
    orders/
      handler.go
      service.go
      repository.go
      model.go
  shared/
    middleware/
    database/
    utils/
cmd/
  api/
    main.go               # Entry point
pkg/                      # Paquetes exportables (si es una librería)
```

---

## PASO 4 — Decisiones Arquitectónicas Clave

### ¿Cuándo crear un nuevo feature?
- Cuando el código resuelve un problema de negocio cohesivo e identificable
- Cuando un desarrollador puede trabajar en él de forma independiente
- Cuando tiene su propio ciclo de vida (puede activarse/desactivarse)

### ¿Qué va en `shared/`?
Solo código que:
- Es usado por **3+ features** (regla de tres)
- No pertenece a ningún dominio específico
- Es genuinamente genérico (Button, useDebounce, formatDate)

### Barrel files (`index.ts`)
- Cada feature expone solo su API pública
- Oculta implementación interna
- Permite refactorizar internamente sin romper consumidores

```ts
// features/auth/index.ts — SOLO lo que otros necesitan
export { AuthGuard } from './components/AuthGuard'
export { useAuth } from './hooks/useAuth'
export type { User, AuthState } from './types/auth.types'
// NO exportar: LoginForm (UI interna), authService (implementación)
```

### Tests
- Los tests viven junto al código que prueban (`auth.service.test.ts` junto a `auth.service.ts`)
- Tests de integración en carpeta `__tests__/` o `tests/` en la raíz del feature
- No crear carpeta `tests/` global separada del código

---

## PASO 5 — Presentación de la Propuesta

Al proponer una arquitectura, presentar siempre:

1. **Árbol de carpetas** con comentarios explicativos
2. **Justificación de decisiones no obvias** (máx. 3 bullets)
3. **Convenciones de nombrado** específicas al stack
4. **Reglas de dependencia** entre módulos
5. **Ejemplo de barrel file** (si aplica)
6. **Plan de migración** si el proyecto ya existe (no proponer reescritura total)

### Formato de árbol

```
src/
  features/
    auth/           # Autenticación y autorización
      components/   # UI exclusiva de auth
      hooks/        # Lógica de estado de auth
      services/     # API calls de auth
      types/        # Tipos del dominio auth
      index.ts      # API pública del feature
    [feature-n]/
  shared/           # Código transversal (regla de 3)
    components/
    utils/
  app/              # Bootstrap, routing, providers
```

---

## PASO 6 — Migración de Proyectos Existentes

Si el proyecto ya tiene código, **NO proponer reescritura total**. En cambio:

1. **Identificar los features** implícitos en el código actual
2. **Crear la nueva estructura** de carpetas sin mover código aún
3. **Migrar feature por feature**, empezando por el más aislado
4. **Mantener compatibilidad** durante la transición con re-exports temporales
5. **Documentar** en un `ARCHITECTURE.md` las convenciones adoptadas

```
# Plan de migración gradual
Semana 1: Crear estructura de carpetas + mover feature Auth
Semana 2: Mover feature Orders + limpiar shared/
Semana 3: Mover feature Products + eliminar re-exports temporales
```

---

## Anti-patrones a Evitar

| Anti-patrón | Por qué es malo | Solución |
|---|---|---|
| Carpetas por tipo técnico (`/controllers`, `/services`) | Un feature queda fragmentado en 5 lugares | Agrupar por feature |
| Un `utils.ts` con 500 funciones | No tiene cohesión, crece sin control | Dividir por dominio o mover al feature |
| Features que se importan entre sí | Acoplamiento cruzado, difícil de cambiar | Usar `shared/` o eventos |
| Lógica de negocio en controllers/views | Imposible testear sin levantar el servidor | Extraer a services |
| Tipos `any` / objetos sin tipar | Pierde la seguridad del tipado | Definir tipos en `types/` del feature |
| `index.ts` que exporta todo | Expone implementación interna | Exportar solo la API pública |

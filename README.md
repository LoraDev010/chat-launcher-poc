# Chat Application - Launcher

Sistema de chat en tiempo real con arquitectura modular basada en Socket.io, React, Node.js y SQLite. El proyecto está dividido en frontend y backend como submódulos Git independientes.

## Quick Start

### Requisitos Previos

- Docker y Docker Compose instalados ([Instalar Docker](https://docs.docker.com/get-docker/))
  - Windows: Docker Desktop
  - macOS: Docker Desktop o Colima  
  - Linux: Docker Engine + Docker Compose plugin

### Con Docker Compose (Recomendado)

```bash
# 1. Clonar con submódulos
git clone --recurse-submodules https://github.com/LoraDev010/chat-launcher-poc.git
cd chat-launcher-poc

# 2. Levantar todo el stack (primera vez puede tardar 2-3 min)
docker compose up -d

# 3. Ver logs (opcional)
docker compose logs -f
```

**URLs:**
- Frontend: http://localhost:5173
- Backend API: http://localhost:3000

**Para detener:**
```bash
docker compose down
```

### Setup Manual (Desarrollo sin Docker)

#### Requisitos
- Node.js 20+ ([Descargar](https://nodejs.org/))
- npm o pnpm

```bash
# 1. Clonar con submódulos
git clone --recurse-submodules https://github.com/LoraDev010/chat-launcher-poc.git
cd chat-launcher-poc

# Si ya clonaste sin --recurse-submodules:
git submodule update --init --recursive

# 2. Setup backend
cd chat/server
npm install
npm run dev   # Corre en http://localhost:3000

# 3. Setup frontend (en otra terminal)
cd ../../chat/client
npm install
npm run dev   # Corre en http://localhost:5173
```

## Estructura del Proyecto

```
chat-launcher-poc/
├── README.md                    # Este archivo
├── docker-compose.yml           # Orquestación Docker
├── .gitmodules                  # Configuración de submódulos
├── .github/                     # CI/CD, agents, instructions
│   ├── instructions/           # Guías de arquitectura para AI
│   │   ├── nodejs-architecture.instructions.md
│   │   └── react-architecture.instructions.md
├── chat/
│   ├── server/                 # Backend (submódulo: chat-backend-poc)
│   │   ├── src/
│   │   │   ├── config/
│   │   │   │   └── database/  # Drizzle ORM + better-sqlite3
│   │   │   ├── modules/
│   │   │   │   ├── chat/      # Lógica de mensajes, rate limiting
│   │   │   │   ├── rooms/     # Gestión de salas + repository pattern
│   │   │   │   └── users/     # Join/leave/kick, bans temporales
│   │   │   ├── shared/        # Tipos, schemas Zod
│   │   │   ├── app.ts         # Express app factory
│   │   │   └── server.ts      # Entry point + Socket.io
│   │   ├── test/              # Tests con Jest (93 tests, 92% coverage)
│   │   ├── Dockerfile
│   │   └── package.json
│   └── client/                 # Frontend (submódulo: chat-frontend-poc)
│       ├── src/
│       │   ├── app/           # Componente raíz
│       │   ├── features/
│       │   │   ├── auth/      # Pantalla de alias
│       │   │   ├── chat/      # ChatRoom, mensajes, typing
│       │   │   └── rooms/     # Lobby, crear/unirse a salas
│       │   └── shared/        # Hooks, tipos, constantes
│       ├── Dockerfile
│       └── package.json
└── MIGRATION_GUIDE.md          # Historial de refactors
```

## Stack Tecnológico

### Backend
- Node.js 22 + TypeScript 5 (ES Modules, strict mode)
- Express para REST API
- Socket.io 4 para comunicación en tiempo real
- Drizzle ORM + better-sqlite3 para persistencia
- Zod para validación de schemas
- Jest 30 para testing (93 tests, 92% coverage)

### Frontend
- React 18 + TypeScript 5
- Vite 8 para bundling y dev server
- Socket.io Client para comunicación en tiempo real
- Arquitectura por features con hooks personalizados

### DevOps
- Docker + Docker Compose para orquestación
- Multi-platform builds (linux/amd64, linux/arm64)
- Colima compatible (macOS ARM/x86)
- Volumes nombrados para persistencia de DB
- Hot-reload habilitado en dev

## Comandos Útiles

### Docker

```bash
# Ver estado de containers
docker compose ps

# Ver logs en tiempo real
docker compose logs -f

# Ver logs de un servicio específico
docker compose logs -f backend
docker compose logs -f frontend

# Rebuild si cambias dependencias
docker compose build
docker compose up -d

# Rebuild forzando BuildKit (recomendado para multi-platform)
DOCKER_BUILDKIT=1 docker compose build --no-cache
DOCKER_BUILDKIT=1 docker compose up -d
docker compose up -d

# Reiniciar un servicio
docker compose restart backend

# Detener y limpiar todo (incluyendo volumes)
docker compose down -v
```

### Backend (desarrollo manual)

```bash
cd chat/server

npm run dev           # Modo watch con tsx
npm run build         # Compilar TypeScript a dist/
npm start             # Ejecutar desde dist/
npm test              # Correr todos los tests
npm run test:coverage # Coverage report

# Drizzle ORM
npm run db:generate   # Generar migraciones desde schema
npm run db:migrate    # Aplicar migraciones
npm run db:studio     # Abrir Drizzle Studio (GUI para DB)
```

### Frontend (desarrollo manual)

```bash
cd chat/client

npm run dev           # Dev server en http://localhost:5173
npm run build         # Build de producción a dist/
npm run preview       # Preview del build
```

## Troubleshooting

### "Error loading shared library ... better_sqlite3.node: Exec format error"

**Causa**: Binario nativo de better-sqlite3 compilado para otra arquitectura.

**Solución**:
```bash
# Opción 1: Rebuild la imagen (recomendado)
docker compose down
docker compose build --no-cache backend
docker compose up -d

# Opción 2: Rebuild manual dentro del container
docker exec chat-backend npm rebuild better-sqlite3
docker restart chat-backend
```

### Frontend no se conecta al backend

1. Verifica que el backend esté corriendo:
   ```bash
   docker compose logs backend
   # Debería mostrar: "Server listening 3000"
   ```

2. Verifica la variable de entorno en `docker-compose.yml`:
   ```yaml
   frontend:
     environment:
       - VITE_SERVER_URL=http://localhost:3000  # ← debe ser localhost
   ```

3. Si usas setup manual, el frontend busca `VITE_SERVER_URL` o usa `http://localhost:3000` por defecto.

### Submódulos vacíos después de clonar

```bash
# Inicializar submódulos
git submodule update --init --recursive

# Si siguen vacíos, forzar actualización
git submodule foreach --recursive git fetch
git submodule update --remote
```

### Puerto 3000 o 5173 ya en uso

```bash
# Opción 1: Detener el proceso que usa el puerto
# En macOS/Linux:
lsof -ti:3000 | xargs kill -9
lsof -ti:5173 | xargs kill -9

# En Windows (PowerShell):
Get-Process -Id (Get-NetTCPConnection -LocalPort 3000).OwningProcess | Stop-Process
Get-Process -Id (Get-NetTCPConnection -LocalPort 5173).OwningProcess | Stop-Process

# Opción 2: Cambiar puertos en docker-compose.yml
services:
  backend:
    ports:
      - "3001:3000"  # ← Host:Container
  frontend:
    ports:
      - "5174:5173"
```

### Docker no compila en Apple Silicon (M1/M2/M3)

**Problema**: Colima puede estar forzando emulación x86.

**Verificar arquitectura**:
```bash
docker run --rm alpine uname -m
# Debería mostrar: aarch64 (ARM) o x86_64
```

**Forzar arquitectura nativa**:
```bash
# Si usas Colima
colima stop
colima start --arch aarch64 --cpu 2 --memory 4

# Rebuild imágenes
docker compose build --no-cache
docker compose up -d
```

### Tests fallan con "Database not initialized"

```bash
# Los tests usan SQLite en memoria, asegúrate de llamar initDb
cd chat/server
npm test

# Si fallan, verifica que initDb(':memory:') esté en beforeAll de cada test
```

## Testing

### Backend Tests

```bash
cd chat/server
npm test              # 93 tests en 8 suites
npm run test:coverage # Coverage: 92% statements, 75% branches
```

**Cobertura actual**:
- `app.ts`: 100%
- `config/`: 100%
- `database/`: 89% (líneas de fallback no testeadas)
- `modules/chat/`: 96%
- `modules/rooms/`: 92%
- `modules/users/`: 100%

## Arquitectura

### Patrón de Capas (Backend)

```
Gateway (Socket.io handlers)
    ↓ valida input con Zod
Service (Lógica de negocio)
    ↓ llama repositorio
Repository (Drizzle ORM)
    ↓ ejecuta queries
SQLite (Persistencia)
```

### Flujo de Datos (Frontend)

```
App.tsx (State global)
    ↓ props
Feature Components
    ↓ custom hooks
useSocket / useRooms / useChatMessages
    ↓ eventos
Socket.io Client
```

## Seguridad

- Rate limiting en mensajes (2s) y typing (1s)
- Validación estricta con Zod en todos los eventos
- Bans temporales (10 min) al kickear usuarios
- Alias únicos por sala
- Códigos de sala de 6 caracteres alfanuméricos únicos
- Input sanitization (max 1000 caracteres por mensaje)
- CORS abierto (*) - configurar en producción

## Licencia

MIT

## Contribuir

1. Fork el proyecto
2. Crea una rama: `git checkout -b feature/nueva-funcionalidad`
3. Commit: `git commit -m 'Add: nueva funcionalidad'`
4. Push: `git push origin feature/nueva-funcionalidad`
5. Abre un Pull Request

## Soporte


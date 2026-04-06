# Chat Application - Launcher

Sistema de chat en tiempo real con arquitectura modular, dividido en frontend y backend como submódulos independientes.

## 🚀 Quick Start

### Con Docker Compose (Recomendado)

```bash
# Clonar con submódulos
git clone --recurse-submodules <repo-url>
cd agentes

# Levantar todo el stack
docker-compose up
```

- Frontend: http://localhost:5173
- Backend: http://localhost:3000

### Setup Manual

```bash
# Clonar con submódulos
git clone --recurse-submodules <repo-url>
cd agentes

# Si ya clonaste sin --recurse-submodules
git submodule update --init --recursive

# Setup backend
cd chat/server
npm install
npm run dev

# Setup frontend (en otra terminal)
cd chat/client
npm install
npm run dev
```

## 📁 Estructura del Proyecto

```
agentes/
├── README.md              # Este archivo
├── docker-compose.yml     # Orquestación de servicios
├── .gitmodules           # Configuración de submódulos
├── .github/              # CI/CD, agents, instructions, skills
│   ├── agents/          # Agentes personalizados de Copilot
│   ├── instructions/    # Instrucciones de arquitectura
│   └── skills/          # Skills de dominio
├── .vscode/              # Configuración de VS Code
└── chat/
    ├── client/  →  submódulo → chat-frontend
    └── server/  →  submódulo → chat-backend
```

## 🏗️ Arquitectura

### Frontend (React + Vite + Socket.io)
- Arquitectura modular basada en features
- TypeScript strict mode
- Comunicación real-time con Socket.io
- Sin autenticación (solo alias)

### Backend (Node.js + Express + Socket.io)
- Arquitectura modular por features
- Validación con Zod
- Testing con Jest (>80% coverage)
- Sin persistencia (memoria)

## 📦 Submódulos

Este proyecto usa Git submodules para mantener frontend y backend en repositorios separados:

- **chat/client** → `chat-frontend` (repo independiente)
- **chat/server** → `chat-backend` (repo independiente)

### Comandos útiles de submódulos

```bash
# Actualizar submódulos a último commit
git submodule update --remote

# Pull de cambios en submódulos
git submodule foreach git pull origin main

# Clonar con submódulos
git clone --recurse-submodules <repo-url>

# Inicializar submódulos después de clonar
git submodule update --init --recursive
```

## 🛠️ Features

- ✅ Salas de chat públicas
- ✅ Mensajes en tiempo real
- ✅ Indicador de "escribiendo..."
- ✅ Lista de usuarios en sala
- ✅ Emojis
- ✅ Sin registro (solo alias)

## 🔧 Configuración

### Variables de Entorno

Cada submódulo puede tener su propio `.env`:

**Backend (chat/server/.env):**
```env
PORT=3000
NODE_ENV=development
```

**Frontend (chat/client/.env):**
```env
# Si necesitas configurar la URL del backend
VITE_API_URL=http://localhost:3000
```

## 📝 Development Workflow

### Trabajar en un submódulo

```bash
# Navegar al submódulo
cd chat/client

# Hacer cambios normalmente
git checkout -b feature/nueva-funcionalidad
# ... hacer cambios ...
git add .
git commit -m "feat: nueva funcionalidad"
git push origin feature/nueva-funcionalidad
```

### Actualizar launcher después de cambios en submódulos

```bash
# Desde la raíz del launcher
git add chat/client  # actualiza referencia del submódulo
git commit -m "chore: actualizar frontend a versión X"
git push
```

## 🧪 Testing

```bash
# Backend tests
cd chat/server
npm test
npm run test:coverage

# Frontend tests (si existen)
cd chat/client
npm test
```

## 🐳 Docker

El `docker-compose.yml` orquesta:
- Servicio backend (Node.js)
- Servicio frontend (Vite dev server)
- Network compartida
- Volúmenes para node_modules

## 📚 Documentación

- [Frontend README](chat/client/README.md)
- [Backend README](chat/server/README.md)
- [Instrucciones de Arquitectura](.github/instructions/)
- [Agentes Personalizados](.github/agents/)

## 🤝 Contribuir

1. Fork de los repos necesarios (launcher, frontend, backend)
2. Crear feature branch
3. Hacer cambios en el submódulo correspondiente
4. Push a tu fork
5. Crear Pull Request al repo correspondiente
6. Actualizar launcher si es necesario

## 📄 Licencia

MIT (o la que prefieras)

## 🆘 Troubleshooting

### Submódulos vacíos después de clonar
```bash
git submodule update --init --recursive
```

### Conflictos de puerto
Cambiar puertos en `docker-compose.yml` o en los `.env` de cada servicio

### node_modules no se encuentran
```bash
# Reinstalar en cada submódulo
cd chat/server && npm install
cd ../client && npm install
```

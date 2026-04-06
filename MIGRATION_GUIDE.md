# 🚀 Guía de Migración a 3 Repositorios

Esta guía detalla los pasos para migrar el proyecto actual a 3 repositorios independientes con submódulos.

## 📋 Archivos Preparados

✅ Ya creados localmente:
- `.gitignore` en cada nivel (launcher, frontend, backend)
- `README.md` completo en cada repo
- `docker-compose.yml` para orquestación
- `Dockerfile` para frontend y backend

## 🎯 Pasos de Migración

### Paso 1: Crear Repositorios en GitHub

Crea 3 repositorios **vacíos** (sin README, sin .gitignore):

1. `chat-launcher` (o el nombre que prefieras para el principal)
2. `chat-frontend` 
3. `chat-backend`

Anota las URLs de cada uno.

### Paso 2: Inicializar Frontend

```bash
cd chat/client

# Inicializar git
git init
git add .
git commit -m "feat: initial commit - chat frontend"

# Conectar con repo remoto
git remote add origin <URL-chat-frontend>
git branch -M main
git push -u origin main
```

### Paso 3: Inicializar Backend

```bash
cd ../server

# Inicializar git
git init
git add .
git commit -m "feat: initial commit - chat backend"

# Conectar con repo remoto
git remote add origin <URL-chat-backend>
git branch -M main
git push -u origin main
```

### Paso 4: Preparar Launcher

```bash
# Volver a la raíz
cd ../..

# Eliminar las carpetas client y server (ahora serán submódulos)
# IMPORTANTE: Asegúrate de que ya hiciste push en los pasos 2 y 3
rm -rf chat/client
rm -rf chat/server

# Inicializar git del launcher
git init
git add .
git commit -m "feat: initial commit - chat launcher"

# Conectar con repo remoto
git remote add origin <URL-chat-launcher>
git branch -M main
git push -u origin main
```

### Paso 5: Agregar Submódulos al Launcher

```bash
# Agregar frontend como submódulo
git submodule add <URL-chat-frontend> chat/client

# Agregar backend como submódulo
git submodule add <URL-chat-backend> chat/server

# Commit de los submódulos
git add .gitmodules chat/client chat/server
git commit -m "chore: add frontend and backend as submodules"
git push
```

### Paso 6: Verificar Estructura

```bash
# Verificar submódulos
git submodule status

# Debe mostrar algo como:
# <commit-hash> chat/client (heads/main)
# <commit-hash> chat/server (heads/main)
```

## ✅ Verificación Final

Tu estructura debe verse así:

```
agentes/                    ← chat-launcher repo
├── README.md
├── docker-compose.yml
├── .gitmodules
├── .github/
└── chat/
    ├── client/  →  submódulo → chat-frontend
    └── server/  →  submódulo → chat-backend
```

## 🧪 Probar el Setup

### 1. Clonar desde cero (simular nuevo desarrollador)

```bash
# En otro directorio
git clone --recurse-submodules <URL-chat-launcher> test-clone
cd test-clone

# Verificar que los submódulos se clonaron
ls -la chat/client
ls -la chat/server
```

### 2. Probar Docker Compose

```bash
docker-compose up
```

Debe levantar frontend (localhost:5173) y backend (localhost:3000)

## 🔄 Workflow de Desarrollo

### Hacer cambios en Frontend

```bash
cd chat/client
git checkout -b feature/nueva-funcionalidad
# ... hacer cambios ...
git add .
git commit -m "feat: nueva funcionalidad"
git push origin feature/nueva-funcionalidad
# Crear PR en el repo chat-frontend
```

### Hacer cambios en Backend

```bash
cd chat/server
git checkout -b fix/bug-importante
# ... hacer cambios ...
git add .
git commit -m "fix: corregir bug importante"
git push origin fix/bug-importante
# Crear PR en el repo chat-backend
```

### Actualizar Launcher después de merge

```bash
# Desde la raíz del launcher
cd chat/client
git pull origin main

cd ../..
git add chat/client  # actualiza la referencia del submódulo
git commit -m "chore: update frontend to v1.2.0"
git push
```

## 📝 Comandos Útiles

### Actualizar todos los submódulos

```bash
git submodule update --remote --merge
```

### Ver estado de submódulos

```bash
git submodule status
git submodule foreach git status
```

### Pull en todos los submódulos

```bash
git submodule foreach git pull origin main
```

## ⚠️ Importante

1. **Antes de borrar chat/client y chat/server** asegúrate de que hiciste push exitoso
2. **Verifica los commits** en GitHub antes de proceder al Paso 4
3. **Haz backup** si tienes dudas (puedes copiar toda la carpeta agentes)

## 🆘 Troubleshooting

### "Submódulo aparece vacío"
```bash
git submodule update --init --recursive
```

### "Cambios no se reflejan en submódulo"
Asegúrate de estar dentro del submódulo y hacer push desde ahí, luego actualizar la referencia en el launcher.

### "Conflicto al agregar submódulo"
Asegúrate de que la carpeta destino NO exista antes de `git submodule add`

---

**¿Todo listo?** Empieza por el Paso 1: crear los repos en GitHub 🚀

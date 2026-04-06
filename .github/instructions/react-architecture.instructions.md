---
description: "Usar cuando se creen o modifiquen componentes React, hooks, servicios, o se estructure el frontend. Cubre: arquitectura por features, separación de responsabilidades, hooks personalizados, tipado con TypeScript, naming conventions."
applyTo: "chat/client/**"
---

# Arquitectura React — Buenas Prácticas

## Estructura por Features (no por tipo técnico)

Organizar el código por dominio funcional. Cada feature es una carpeta autónoma:

```
src/
  features/
    chat/
      components/        # UI exclusiva del feature
      hooks/             # Lógica de estado y side-effects
      services/          # Llamadas a API / lógica de negocio
      types/             # Interfaces y tipos del dominio
      index.ts           # Barrel: exporta solo la API pública
    auth/
      components/
      hooks/
      services/
      types/
      index.ts
  shared/
    components/          # Componentes UI genéricos (Button, Modal, Toast)
    hooks/               # Hooks genéricos (useDebounce, useClickOutside)
    services/            # Cliente HTTP, config de conexión
    types/               # Tipos globales compartidos
    utils/               # Funciones utilitarias puras
    constants/           # Configuración, magic numbers centralizados
  app/
    App.tsx              # Composición raíz, providers
    router.tsx           # Enrutamiento
```

## Reglas de Dependencia

- Los features **nunca se importan entre sí** directamente
- Comunicación entre features: a través de `shared/` o eventos
- `shared/` solo contiene código usado por **3+ features** (regla de tres)
- Cada feature exporta únicamente su API pública desde `index.ts`

## Componentes

- Un componente = un archivo. Nombre en `PascalCase.tsx`
- Extraer lógica de estado a hooks personalizados — el componente solo renderiza
- No mezclar lógica de negocio con presentación
- Favorecer composición sobre herencia y props drilling

```tsx
// MAL — lógica de negocio en el componente
function ChatRoom() {
  const [messages, setMessages] = useState([]);
  useEffect(() => { socket.on('message', m => setMessages(prev => [...prev, m])); }, []);
  // ... 200 líneas más de lógica
  return <div>...</div>;
}

// BIEN — lógica extraída a hook
function ChatRoom() {
  const { messages, sendMessage } = useChatMessages(socket);
  const { typingUsers } = useTypingIndicator(socket);
  return <div>...</div>;
}
```

## Hooks Personalizados

- Nombre: `camelCase` con prefijo `use` → `useChat.ts`, `useTypingIndicator.ts`
- Un hook = una responsabilidad. No crear hooks "god object"
- Mover la gestión de Socket.io a hooks dedicados
- Usar `useCallback` y `useMemo` con dependencias correctas
- No usar variables mutables a nivel de módulo — usar `useRef`

```tsx
// MAL — contadores globales mutables
let messageCounter = 0;

// BIEN — refs dentro del componente
const messageCounterRef = useRef(0);
```

## TypeScript

- Usar `.tsx` para componentes y `.ts` para lógica pura
- Definir interfaces para props, estado, y eventos de Socket.io
- No usar `any` — definir tipos en `types/` del feature
- Tipar eventos de Socket.io con interfaces explícitas

```ts
// types/chat.types.ts
interface ChatMessage {
  id: string;
  type: 'user' | 'system';
  text: string;
  alias: string;
  ts: number;
}

interface SocketEvents {
  message: (msg: ChatMessage) => void;
  typing: (data: { alias: string }) => void;
  user_joined: (data: { alias: string; users: string[] }) => void;
}
```

## Servicios

- Nombre: `camelCase` con sufijo `Service` → `chatService.ts`
- Encapsular lógica de conexión y comunicación (Socket.io, fetch)
- No llamar sockets o APIs directamente desde componentes

## Constantes — Eliminar Magic Numbers

```ts
// shared/constants/chat.ts
export const TYPING_TIMEOUT_MS = 2000;
export const TYPING_EMIT_INTERVAL_MS = 1000;
export const TOAST_DURATION_MS = 4000;
export const MAX_MESSAGE_LENGTH = 1000;
export const MAX_ALIAS_LENGTH = 32;
```

## Estado

- Estado local simple: `useState` + `useReducer`
- Estado complejo o compartido: Zustand o Context con `useReducer`
- No prop-drill más de 2 niveles — extraer a Context o store

## Convenciones de Nombrado

| Tipo | Patrón | Ejemplo |
|------|--------|---------|
| Componente | `PascalCase.tsx` | `ChatRoom.tsx` |
| Hook | `use*.ts` | `useChat.ts` |
| Servicio | `*Service.ts` | `chatService.ts` |
| Tipos | `*.types.ts` | `chat.types.ts` |
| Constantes | `SCREAMING_SNAKE` | `MAX_MESSAGE_LENGTH` |
| Barrel | `index.ts` | Solo API pública del feature |

## Manejo de Errores

- Usar Error Boundaries para errores de renderizado
- Los hooks que manejan async deben exponer estado de error
- Mostrar feedback al usuario (toasts, banners) — no fallar silenciosamente

## Testing

- Tests junto al código: `ChatRoom.test.tsx` al lado de `ChatRoom.tsx`
- Tests de integración por feature en `__tests__/`
- Hooks testeables con `@testing-library/react-hooks`

## Documentación

Generar documentación de componentes y hooks con JSDoc. Explicar props, eventos, y comportamiento esperado(SOLO DONDE SEA NECESARIO).

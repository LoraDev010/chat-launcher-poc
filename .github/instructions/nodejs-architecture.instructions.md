---
description: "Usar cuando se cree o modifique código del servidor Node.js/Express con Socket.io. Cubre: arquitectura modular por features, validación, seguridad, manejo de errores, rate limiting, TypeScript, convenciones de nombrado."
applyTo: "chat/server/**"
---

# Arquitectura Node.js/Express — Buenas Prácticas

## Estructura por Módulos

Organizar el servidor por dominio funcional, no por capa técnica:

```
src/
  modules/
    chat/
      chat.gateway.ts       # Handlers de Socket.io del feature
      chat.service.ts        # Lógica de negocio
      chat.types.ts          # Interfaces y tipos
      chat.constants.ts      # Configuración del módulo
    rooms/
      rooms.service.ts
      rooms.types.ts
    users/
      users.service.ts
      users.types.ts
  shared/
    middleware/              # CORS, logging, error handler
    validation/              # Schemas y funciones de validación
    exceptions/              # Clases de error tipadas
    utils/                   # Funciones utilitarias puras
    constants/               # Configuración global
  config/
    env.ts                   # Variables de entorno validadas
    socket.ts                # Configuración Socket.io
  app.ts                     # Express app factory
  server.ts                  # Entry point, arranque
```

## Separación de Responsabilidades

- **Gateway/Controller**: Recibe eventos, valida input, delega al servicio
- **Service**: Lógica de negocio pura, testeable sin framework
- **Types**: Interfaces de dominio, DTOs, eventos tipados

```ts
// MAL — toda la lógica en el handler de socket
socket.on('message', (data) => {
  if (!data.text || data.text.length > 1000) return;
  const now = Date.now();
  if (now - user.lastMessageAt < 2000) return;
  // ... 30 líneas más
});

// BIEN — handler delega al servicio
socket.on('message', (data, ack) => {
  const result = chatService.processMessage(socket.id, data);
  if (result.error) return ack({ error: result.error });
  io.to(roomId).emit('message', result.message);
  ack({ ok: true });
});
```

## TypeScript

- Usar `.ts` para todo el servidor
- Tipar todos los eventos de Socket.io con interfaces

```ts
// chat.types.ts
interface ServerToClientEvents {
  message: (msg: ChatMessage) => void;
  user_joined: (data: { alias: string; users: string[] }) => void;
  user_left: (data: { alias: string; users: string[] }) => void;
  typing: (data: { alias: string }) => void;
  user_kicked: (data: { alias: string; reason: string }) => void;
}

interface ClientToServerEvents {
  join_room: (data: JoinRoomDTO, ack: AckCallback) => void;
  message: (data: MessageDTO, ack: AckCallback) => void;
  typing: () => void;
  kick_user: (data: KickDTO) => void;
}

interface ChatMessage {
  id: string;
  type: 'user' | 'system';
  text: string;
  alias: string;
  ts: number;
}
```

## Validación de Input

- Validar **toda** entrada del cliente en la capa gateway/controller
- Usar schemas (Zod, joi) para validación declarativa
- No confiar en el cliente — sanitizar strings, validar tipos y rangos
- Validar en el boundary del sistema, no dentro de la lógica de negocio

```ts
import { z } from 'zod';

const messageSchema = z.object({
  text: z.string().trim().min(1).max(MAX_MESSAGE_LENGTH),
  roomId: z.string().regex(/^[a-zA-Z0-9-]+$/),
});
```

## Seguridad

- Sanitizar mensajes contra XSS antes de broadcast
- Implementar rate limiting por socket/IP, no solo por timestamp
- No exponer stack traces ni detalles internos en errores al cliente
- Agregar autenticación en la conexión Socket.io (middleware de handshake)
- Validar permisos antes de acciones destructivas (kick, ban)

## Manejo de Errores

- Crear clases de error tipadas para errores de dominio
- Capturar errores en handlers — no dejar que crashes tiren el servidor
- Logging estructurado (no `console.log` en producción)
- Implementar graceful shutdown (cerrar conexiones, limpiar estado)

```ts
// shared/exceptions/app-error.ts
class AppError extends Error {
  constructor(
    public readonly code: string,
    message: string,
    public readonly statusCode: number = 400
  ) {
    super(message);
  }
}

class RateLimitError extends AppError {
  constructor() {
    super('RATE_LIMITED', 'Too many requests', 429);
  }
}
```

## Constantes — Eliminar Magic Numbers

```ts
// shared/constants/server.ts
export const MESSAGE_RATE_LIMIT_MS = 2000;
export const TYPING_RATE_LIMIT_MS = 1000;
export const BAN_DURATION_MS = 10 * 60 * 1000; // 10 minutos
export const MAX_MESSAGE_LENGTH = 1000;
export const MAX_BUFFER_SIZE = 1e6; // 1MB
export const SERVER_PORT = parseInt(process.env.PORT || '3000', 10);
```

## Configuración de Entorno

- Leer variables de entorno desde un archivo centralizado `config/env.ts`
- Validar variables requeridas al arrancar — fallar rápido si faltan
- No hardcodear puertos, URLs, o credenciales

```ts
// config/env.ts
const env = z.object({
  PORT: z.coerce.number().default(3000),
  CORS_ORIGIN: z.string().default('http://localhost:5173'),
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
}).parse(process.env);

export default env;
```

## Convenciones de Nombrado

| Tipo | Patrón | Ejemplo |
|------|--------|---------|
| Gateway | `*.gateway.ts` | `chat.gateway.ts` |
| Servicio | `*.service.ts` | `chat.service.ts` |
| Tipos | `*.types.ts` | `chat.types.ts` |
| Constantes | `*.constants.ts` | `chat.constants.ts` |
| Middleware | `*.middleware.ts` | `auth.middleware.ts` |
| Config | descriptivo | `env.ts`, `socket.ts` |

## Testing

- Tests junto al código: `chat.service.test.ts` al lado de `chat.service.ts`
- Servicios testeables sin levantar el servidor (inyección de dependencias)
- Tests de integración con `supertest` + socket.io-client mock

## Anti-patrones a Evitar

| Anti-patrón | Solución |
|---|---|
| Toda la lógica en un solo `index.js` | Separar en módulos por feature |
| `console.log` como logging | Logger estructurado (pino, winston) |
| Sin validación de input | Zod/joi en el boundary |
| Magic numbers por todo el código | Constantes con nombre descriptivo |
| Estado global mutable sin encapsular | Encapsular en servicios con interfaz clara |
| Catch vacío o sin logging | Siempre log + respuesta de error |

## Documentación

Generar documentación de componentes y hooks con JSDoc. Explicar props, eventos, y comportamiento esperado(SOLO DONDE SEA NECESARIO).
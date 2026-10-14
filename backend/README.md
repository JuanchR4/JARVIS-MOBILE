# Puente Claude API para JARVIS-MOBILE

Este Worker mantiene la clave de Claude en el servidor. La página de GitHub Pages nunca recibe `ANTHROPIC_API_KEY`.

## Requisitos

- Cuenta de Cloudflare con Workers habilitado.
- Cuenta de Claude Console con acceso a API y facturación/límite de gasto configurados.
- Wrangler instalado para desplegar desde terminal, o crear el Worker desde el panel de Cloudflare.

## Secretos y configuración

Configura en Cloudflare Worker → Settings → Variables and Secrets:

- `ANTHROPIC_API_KEY` — API key creada en Claude Console (marcar como Secret).
- `JARVIS_ACCESS_CODE` — frase de acceso fuerte, de al menos 16 caracteres (Secret).
- `JARVIS_SESSION_SECRET` — secreto aleatorio largo, mínimo 32 caracteres (Secret).
- `CLAUDE_MODEL` — identificador actual de modelo publicado en la documentación de modelos de Claude. Es configuración, no una clave.

El código de acceso autentica Jarvis ante el puente. El Worker intercambia ese código por un token de sesión firmado, con vencimiento de 12 horas. No uses el mismo valor para la API key, el código de acceso y el secreto de sesión.

## Despliegue con Wrangler

Desde esta carpeta:

```sh
npx wrangler deploy
npx wrangler secret put ANTHROPIC_API_KEY
npx wrangler secret put JARVIS_ACCESS_CODE
npx wrangler secret put JARVIS_SESSION_SECRET
```

Define `CLAUDE_MODEL` en Variables and Secrets del Worker, luego vuelve a desplegar si el panel lo solicita. El despliegue entrega una URL `workers.dev`; conserva esa URL para configurar la consola Jarvis. No publiques secretos en GitHub ni los pegues en chats.

Antes de dar acceso a terceros, configura una regla de rate limiting en Cloudflare para `/auth/login` y `/api/chat`, y un límite de gasto en Claude Console. CORS limita el origen del navegador pero no sustituye autenticación ni rate limiting.

## Rutas

- `GET /health` — prueba de disponibilidad, sin revelar secretos.
- `POST /auth/login` con `{ "passcode": "..." }` — devuelve token firmado.
- `POST /api/chat` con `{ "question": "..." }` y `Authorization: Bearer <token>` — devuelve respuesta en español.

La primera versión solo conversa y genera texto/código. No tiene herramientas para controlar archivos, programas, impresora ni robot. Añadir esas herramientas requiere un agente local separado, lista permitida y confirmaciones.

## Claude Console

Crear la API key en Settings → API keys. La clave se muestra al generarla: guárdala directamente como Secret en Cloudflare. No la pongas en `index.html`, `knowledge.json`, `.env` versionado ni en el chat.

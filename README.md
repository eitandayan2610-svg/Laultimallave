# La Última Llave + Zapia (MCP para Netlify)

Este paquete agrega un endpoint MCP remoto y OAuth a un sitio Netlify existente. **No reemplaza el juego ni su página**: copiá/mezclá los archivos en la raíz del repositorio que publica `https://laultimallave.netlify.app` y desplegá ese sitio. Esta revisión corrige el formato de respuesta de las funciones de Netlify; si ya publicaste la primera versión, reemplazá los archivos de `netlify/functions/` con los de esta versión y volvé a desplegar.

## Qué incluye y qué no

- Endpoint MCP `https://laultimallave.netlify.app/mcp`.
- Descubrimiento OAuth, registro dinámico de clientes, autorización con consentimiento, PKCE S256 y tokens de una hora.
- Almacenamiento de clientes y códigos de autorización de un solo uso con Netlify Blobs.
- Dos herramientas MCP de solo lectura: `get_game_info` y `get_website_features`.
- No obtiene partidas guardadas ni datos de jugadores. La web actual guarda localmente en el navegador y no tiene sincronización de cuentas.

## Instalación en el proyecto Netlify

1. Hacé una copia del proyecto actual (o creá una rama de Git) antes de tocarlo.
2. Copiá `netlify/functions/` a la carpeta correspondiente de tu proyecto. Si ya existe `netlify.toml`, **no lo sobrescribas**: combina `[functions]` y las seis reglas `[[redirects]]` de este paquete con tu configuración actual. Si no existe, copiá el `netlify.toml` incluido.
3. Combina la dependencia de `package.json` con la de tu proyecto; no borres dependencias que ya usa el juego. Netlify debe instalar `@netlify/blobs` (incluida en el package.json de este paquete).
4. En Netlify, en la configuración del sitio, agregá la variable de entorno `LL_MCP_SECRET`. Generá un valor aleatorio de al menos 32 bytes, por ejemplo con `openssl rand -hex 32`. Guardalo como secreto; no lo publiques ni lo incluyas en el repositorio.
5. Desplegá el sitio. Netlify Blobs se usa para conservar los registros OAuth entre invocaciones.
6. Probá en el navegador:
   - `https://laultimallave.netlify.app/.well-known/oauth-protected-resource/mcp`
   - `https://laultimallave.netlify.app/.well-known/oauth-authorization-server`
   Deben responder con JSON (no la página 404 de Netlify). El MCP `/mcp` debe exigir autorización y devolver 401 si se abre sin token; ese 401 es esperado.
7. En Zapia, agregá como servidor MCP la URL exacta `https://laultimallave.netlify.app/mcp` y completá el consentimiento OAuth. Si Zapia tiene un panel de conectores MCP, usá ese mismo endpoint.

## Desarrollo local

Requiere Node.js 20+ y Netlify CLI. En el directorio del proyecto ejecutá `npm install`, establecé `LL_MCP_SECRET` en el entorno y levantá el sitio con `netlify dev`. Las rutas se sirven bajo el mismo dominio local de Netlify CLI.

## Seguridad y mantenimiento

- La conexión solo da permiso de lectura sobre información pública; la página de consentimiento lo explica.
- El consentimiento no requiere cuenta porque no hay datos privados expuestos. Si luego agregás perfiles, puntajes privados, progreso sincronizado o acciones de escritura, no amplíes estas herramientas sin implementar inicio de sesión de jugadores, autorización por usuario y controles adicionales.
- No incluyas `LL_MCP_SECRET` en archivos, commits ni en el código del navegador.
- Si el sitio ya tiene reglas de rutas, revisalas: las reglas de MCP tienen que permanecer activas sin interferir con las existentes.

## Diagnóstico rápido

- 404 en `/.well-known/...`: las reglas de `netlify.toml` no se combinaron o el deploy no se publicó.
- Error de función indicando `LL_MCP_SECRET`: falta configurar la variable en Netlify; guardala y volvé a desplegar.
- Error de Blobs: confirmá que las funciones corren en Netlify y que `@netlify/blobs` se instaló desde el `package.json`.
- Si la página funciona pero `/mcp` no, revisá los logs de Functions del último deploy.

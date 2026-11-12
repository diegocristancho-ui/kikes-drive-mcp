# Kikes MCP Server

Servidor MCP local (stdio) con cuatro tools que llaman al webhook de Kikes para gestionar mensajes y documentos de Telegram.

## Descarga

**[Descargar la última versión](../../releases/latest)**: en la página de esa release, baja el archivo que necesites:

- `kikes-mcp-server-<versión>.mcpb`: extensión para Claude Desktop (recomendada; se instala con doble clic).
- `kikes-mcp-server-<versión>.tgz`: paquete npm, para instalación manual.

Antes de usar el conector debes aceptar los [Términos de uso](#términos-de-uso-y-acceso-eula).

> El enlace es relativo: funciona al ver este README en GitHub y apunta siempre a la release más reciente del repositorio donde esté publicado. En otros visores (por ejemplo, en local) no funcionará.

## Términos de uso y acceso (EULA)

**Versión 1.0 · 2026-11-12**

Este conector permite enviar mensajes y documentos a Telegram, consultar los últimos mensajes y eliminar el más reciente a través del webhook de Kikes. **Solo funciona si la persona que lo usa acepta estos términos**, configurando la variable de entorno `KIKES_ACCEPT_EULA` en `true`. Si no está configurada, o tiene cualquier otro valor, el conector queda **bloqueado**: ninguna herramienta ejecuta acciones. Estos términos también se muestran en la descripción de cada tool y en las instrucciones que recibe el asistente.

### 1. Aceptación

- Al configurar `KIKES_ACCEPT_EULA=true`, el usuario declara que ha leído, entendido y aceptado estos términos.
- La aceptación la hace **personalmente el usuario**. Ningún asistente o agente de IA puede aceptarlos en su nombre ni modificar la configuración para hacerlo.
- Si no estás de acuerdo, no configures la variable y no uses el conector.

### 2. Clave de acceso (`KIKES_AUTH`)

- La clave es personal e intransferible y da acceso al webhook de Kikes.
- El usuario es el **único responsable de custodiarla** y de **no divulgarla ni difundirla**: no compartirla, no publicarla en repositorios, capturas de pantalla, chats, correos, tickets o documentos, y no dejarla donde otras personas puedan leerla (por ejemplo, el archivo de configuración de Claude en un equipo compartido).
- Todo lo que se haga con la clave se considera hecho por el usuario, lo haya hecho él o un tercero que accedió a ella.
- Si sospecha que la clave se filtró, debe avisar de inmediato a Kikes para que la rote.

### 3. Contenido enviado

- El usuario es el **único responsable de todo lo que se envíe con el conector**: mensajes y archivos, su contenido, su legalidad, la titularidad de los derechos, su confidencialidad y los datos personales que contengan.
- Está prohibido enviar **archivos maliciosos** (virus, malware, documentos con código o enlaces dañinos), contenido ilícito, información confidencial sin autorización o que infrinja derechos de terceros.
- Los controles del conector (lista de tipos de archivo, verificación del contenido y límite de tamaño) son limitados y **no garantizan** que un archivo esté libre de código malicioso ni de información sensible. No sustituyen la revisión del usuario.

### 4. Uso a través de asistentes de IA

- Lo que un asistente (por ejemplo, Claude) ejecute con este conector se considera ejecutado por el usuario.
- Los asistentes pueden equivocarse o ser manipulados (por ejemplo, por instrucciones ocultas en un documento). El usuario debe revisar lo que se va a enviar o eliminar antes de autorizarlo.

### 5. Usos prohibidos

Spam, phishing, suplantación de identidad, acoso, acceso o intentos de acceso no autorizados, eludir los controles del conector y cualquier uso contrario a la ley.

### 6. Sin garantías

El conector se ofrece "tal cual", sin garantía de disponibilidad, entrega ni ausencia de errores (por ejemplo, los túneles de ngrok pueden caerse).

### 7. Limitación de responsabilidad

En la medida en que lo permita la ley aplicable, Kikes no responde por los daños derivados del uso del conector, de la divulgación o el mal uso de la clave de acceso, ni del contenido que el usuario envíe.

### 8. Suspensión y cambios

Kikes puede rotar claves, suspender accesos y modificar estos términos. Los cambios se publicarán en este documento con una nueva versión.

### 9. Ley aplicable

[Pendiente de definir por Kikes: ley y jurisdicción aplicables.]

### Cómo aceptar los términos (configurar `KIKES_ACCEPT_EULA` en `true`)

Lo debe hacer el propio usuario, de una de estas formas, y después **reiniciar Claude Desktop**:

- **Extensión `.mcpb` (Claude Desktop):** en la configuración de la extensión, marcar la casilla *Acepto los Términos de uso (EULA)*.
- **Instalación manual:** en `claude_desktop_config.json`, dentro de `env` del conector, añadir `"KIKES_ACCEPT_EULA": "true"`.
- **Claude Code:** añadir `-e KIKES_ACCEPT_EULA=true` al comando `claude mcp add`.

Solo el valor `true` (sin distinguir mayúsculas) acepta los términos; `false`, `1`, `yes` o un valor vacío no.

**Comportamiento del asistente:** si el EULA no está aceptado, el asistente recibe esta información desde el inicio y cada tool devuelve un mensaje de bloqueo. Debe recordarte los términos y explicarte cómo configurar la variable en `true`; no puede aceptarlos por ti ni reintentar la acción.

## Tools

| Nombre | Parámetro | Descripción |
|---|---|---|
| `send_telegram_message` | `text_content` (string, 1–4096 caracteres, **en formato HTML de Telegram**) | Envía el mensaje. |
| `send_telegram_document` | `file_path` (ruta absoluta del archivo en el equipo); `caption` (opcional, texto plano, hasta 1024 caracteres) | Envía un documento (PDF, Office, Markdown y similares; ver tipos permitidos), de hasta 50 MB, con un pie opcional. |
| `delete_last_telegram_message` | ninguno | Elimina el mensaje más reciente. No se puede deshacer. |
| `get_last_telegram_messages` | ninguno | Consulta los últimos 10 mensajes (contenido, ID del mensaje, ID del chat, fecha de creación y de actualización). Solo lectura. |

`send_telegram_message`, `send_telegram_document` y `delete_last_telegram_message` devuelven el mensaje que responda el endpoint (campo `message`, `mensaje` o `text` si es JSON, o el texto plano). Si no hay mensaje, devuelven uno por defecto (`Mensaje enviado correctamente.` / `Documento enviado correctamente.` / `Mensaje eliminado correctamente.`).

`get_last_telegram_messages` devuelve el JSON del endpoint indentado, sin renombrar campos. Si la respuesta viene vacía (cuerpo vacío, `[]` o `{}`), devuelve `No hay mensajes enviados.`, que no es un error.

Los errores HTTP y de red se devuelven como error de la tool.

Peticiones que realizan:

```
POST <TELEGRAM_WEBHOOK_URL>
Content-Type: application/json

{"text_content": "..."}
```

```
POST <TELEGRAM_WEBHOOK_URL sin el "/send" final>/document/send
Content-Type: multipart/form-data

caption=<texto>        (solo si se indica)
archivo=<archivo binario>
```

```
DELETE <TELEGRAM_WEBHOOK_URL>/delete-last-message
(sin body)
```

```
GET <TELEGRAM_WEBHOOK_URL sin el "/send" final>/get/lastest
```

## Estructura del código

Cada módulo tiene una sola responsabilidad y depende de interfaces, no de los detalles de otro módulo. Todo se conecta en un único sitio, `index.ts`.

```
src/
  index.ts                 Arranque: carga la config, crea las piezas y conecta el transporte stdio
  eula.ts                  Aviso legal, instrucciones para el agente y bloqueo de todas las tools sin EULA aceptado
  config.ts                Lee y valida las variables de entorno (no escribe logs ni termina el proceso)
  icons.ts                 Carga el icono (assets/icon.png)
  files/
    attachment.ts          Lee un archivo local para enviarlo: valida ruta, tipo, tamaño y contenido
    document-types.ts      Lista de tipos permitidos (extensión, tipo MIME y comprobación de contenido)
  server.ts                Crea el servidor MCP: metadatos, instrucciones y registro de las tools
  webhook/
    client.ts              Cliente HTTP: fetch, header kikes_auth, reintento con el dominio de respaldo
    response.ts            Interpreta el cuerpo de las respuestas (mensaje del endpoint, lista vacía)
  tools/
    context.ts             Lo que reciben las tools: endpoints + execute()
    respond.ts             Convierte el resultado HTTP en la respuesta que ve el modelo
    index.ts               Registra todas las tools
    send-telegram-message.ts
    send-telegram-document.ts
    delete-last-telegram-message.ts
    get-last-telegram-messages.ts
```

Flujo de una llamada: la tool llama a `execute()` → el cliente hace la petición (con auth y reintento) y devuelve un resultado sin interpretar → `respond.ts` lo convierte en texto para el modelo.

**Para añadir una tool:** crea `src/tools/<nombre>.ts` con una función `register...(server, context)` que use `context.execute()`, y regístrala en `src/tools/index.ts`. El bloqueo por EULA y el aviso legal se aplican solos a toda tool registrada ahí. Si usa un endpoint nuevo, añádelo a `endpoints` en `config.ts`.

## Instalación

```bash
npm install
npm run build
```

## Empaquetado e instalación en otro equipo

Generar el paquete (compila automáticamente antes de empaquetar):

```bash
npm pack --pack-destination releases
```

Crea `releases/kikes-mcp-server-<versión>.tgz`. Para usarlo en otro equipo (requiere Node 18 o superior):

```bash
npm install -g kikes-mcp-server-<versión>.tgz
```

Esto instala el comando `kikes-mcp-server`. En la configuración de Claude Desktop se puede usar así:

```json
{
  "mcpServers": {
    "kikes": {
      "command": "kikes-mcp-server",
      "env": { "KIKES_ACCEPT_EULA": "true", "KIKES_AUTH": "<tu-clave>" }
    }
  }
}
```

Si el comando no se encuentra, usa la ruta completa a `node` y al `build/index.js` instalado (ver más abajo).

Para publicar una versión nueva, sube la versión en `package.json` (la del servidor se lee de ahí) y vuelve a empaquetar: `npm run pack:mcpb` genera el `.mcpb` y `npm pack --pack-destination releases` el `.tgz`. Después, sube ambos archivos a una nueva release de GitHub; el enlace de la sección [Descarga](#descarga) apuntará a ella automáticamente.

## Extensión de Claude Desktop (.mcpb)

Alternativa al paquete npm: un único archivo que se instala sin npm ni editar JSON.

```bash
npm run pack:mcpb
```

Genera `releases/kikes-mcp-server-<versión>.mcpb` (incluye las dependencias; unos 4 MB). Para instalarlo: doble clic en el archivo, arrastrarlo a la ventana de Claude Desktop, o *Settings > Extensions > Advanced settings > Install Extension…*. Al instalar, Claude Desktop muestra un formulario con dos campos obligatorios: la casilla *Acepto los Términos de uso (EULA)* (`accept_eula`, que se pasa al servidor como `KIKES_ACCEPT_EULA`) y la clave de autenticación (`kikes_auth`, campo sensible, que se pasa como `KIKES_AUTH`).

El manifest está en `mcpb/manifest.json` y el script de empaquetado en `scripts/pack-mcpb.mjs`. La versión del manifest se toma siempre de `package.json`.

## Icono

El icono es `assets/icon.png`. Se usa en dos sitios:

- **Servidor MCP:** se incrusta como data URI en la información del servidor (`serverInfo.icons`). No hay confirmación de que Claude Desktop lo muestre para servidores lanzados desde `claude_desktop_config.json`.
- **Extensión .mcpb:** se copia como `icon.png` en la raíz del bundle y se referencia desde el manifest. Es la vía documentada por Anthropic (recomendado 512×512 px, mínimo 256×256, PNG con transparencia).

Para cambiarlo, reemplaza `assets/icon.png` y vuelve a empaquetar. El actual es provisional (320×320 px, tomado de `ti_logo.png`). Si falta el archivo, el servidor arranca igual sin icono.

## Configuración

Variables de entorno (se definen en el bloque `env` del conector de Claude):

- `KIKES_ACCEPT_EULA`: **obligatoria para usar el conector.** Debe valer `true` para aceptar los [Términos de uso](#términos-de-uso-y-acceso-eula). Sin ella, el conector arranca pero todas las tools devuelven un mensaje de bloqueo y el servidor deja un aviso por stderr.
- `KIKES_AUTH`: clave de autenticación. Su valor se envía en el header **`kikes_auth`** en todas las peticiones, tanto al dominio principal como al de respaldo. Si no se define, las peticiones van sin ese header (y el servidor registra un aviso por stderr). Solo admite ASCII imprimible: con otros caracteres el servidor no arranca. El valor nunca aparece en los mensajes de error ni en los logs.
- `TELEGRAM_WEBHOOK_URL`: URL del endpoint de envío (`.../telegram/send`). La de borrado se deriva añadiendo `/delete-last-message` y la de consulta cambiando el `/send` final por `/get/lastest`. Si no se define, usa la URL de ngrok incluida en `src/index.ts`. Los dominios gratuitos de ngrok pueden cambiar; si cambia, define esta variable en lugar de recompilar.

- `TELEGRAM_FALLBACK_HOST`: dominio de respaldo (por defecto `darn-trapdoor-survivor.ngrok-free.dev`). Si vale `""` (vacío), el reintento se desactiva. Admite `host:puerto`.

### Reintento automático con el dominio de respaldo

Es interno: Claude solo ve el resultado final. Se repite la misma petición (mismo método y ruta) cambiando únicamente el dominio.

| Situación en el dominio principal | ¿Reintenta con el respaldo? |
|---|---|
| Fallo de conexión (DNS, conexión rechazada, etc.) | Sí |
| Túnel de ngrok caído (HTTP 404 con cabecera `Ngrok-Error-Code`, p. ej. `ERR_NGROK_3200`) | Sí |
| Timeout (10 s) en la consulta (`GET`) | Sí |
| Timeout (10 s) al enviar o eliminar (`POST`/`DELETE`) | **No**: la petición pudo procesarse y reintentar duplicaría el mensaje o borraría otro. La tool avisa de que se verifique antes de reintentar. |
| Error HTTP normal (400, 500, etc.) | No: el servidor sí recibió la petición |

Si ambos dominios fallan, el error indica el motivo de cada uno.

### Claude Desktop

Edita `%APPDATA%\Claude\claude_desktop_config.json` y reinicia Claude Desktop:

```json
{
  "mcpServers": {
    "kikes": {
      "command": "node",
      "args": ["C:\\Users\\juan.cristancho\\Documents\\Kikes MCP Server\\build\\index.js"],
      "env": { "KIKES_ACCEPT_EULA": "true", "KIKES_AUTH": "<tu-clave>" }
    }
  }
}
```

Para fijar otra URL añade `"TELEGRAM_WEBHOOK_URL": "https://..."` dentro de `env`.

### Claude Code

```bash
claude mcp add kikes -e KIKES_ACCEPT_EULA=true -e KIKES_AUTH=<tu-clave> -- node "C:\Users\juan.cristancho\Documents\Kikes MCP Server\build\index.js"
```

## Autenticación

Las peticiones llevan el header `kikes_auth: <valor de KIKES_AUTH>`. Quien reciba la petición (n8n) debe validarlo; este servidor solo lo envía.

- Si el endpoint responde 401 o 403, la tool lo indica y avisa si falta configurar `KIKES_AUTH` o si su valor es incorrecto.
- No se siguen redirecciones HTTP, para que la clave no pueda reenviarse a otro host.
- Algunos proxies inversos (nginx por defecto) descartan los headers con guion bajo. Si el endpoint nunca ve `kikes_auth`, revisa ese punto.
- La clave queda en texto plano en el archivo de configuración de Claude. El paquete no la incluye, pero sí las URLs de ngrok.

## Envío de documentos: tipos permitidos y seguridad

`send_telegram_document` lee un archivo del equipo y lo sube al webhook, así que es la tool con más riesgo de fuga de información. Por eso solo acepta **documentos**, según una lista de permitidos (todo lo demás se rechaza):

| Categoría | Extensiones |
|---|---|
| PDF | `.pdf` |
| Word y equivalentes | `.doc`, `.docx`, `.odt`, `.rtf` |
| Excel y equivalentes | `.xls`, `.xlsx`, `.ods`, `.csv`, `.tsv` |
| PowerPoint y equivalentes | `.ppt`, `.pptx`, `.odp` |
| Markdown y texto | `.md`, `.markdown`, `.txt` |
| Google Workspace (accesos directos que exporta Drive) | `.gdoc`, `.gsheet`, `.gslides` |
| Artifacts de Claude | `.html`, `.htm`, `.svg` |

Quedan fuera, entre otros: ejecutables y scripts (`.exe`, `.msi`, `.bat`, `.ps1`, `.sh`, `.command`, `.jar`…), archivos comprimidos (`.zip`, `.rar`, `.7z`, `.gz`…), archivos de Office con macros (`.docm`, `.xlsm`, `.pptm`), imágenes (`.png`, `.jpg`…), archivos sin extensión (`.env`, claves) y `.json`. Para permitir otro tipo, añádelo a `src/files/document-types.ts`.

- **El contenido debe corresponder a la extensión.** Se comprueban las firmas del archivo: un `.exe` renombrado a `.pdf` se rechaza, igual que un `.zip` renombrado a `.docx` o un script con extensión `.txt` o `.md`.
- La tool exige una ruta absoluta, que el archivo exista, no esté vacío y pese 50 MB o menos (el máximo de Telegram para bots). Los enlaces simbólicos se resuelven y se valida el archivo real.
- **Límite de esta protección:** un `.txt`, `.md`, `.csv` o `.pdf` puede contener información sensible (contraseñas, contratos). La lista de tipos no lo detecta. La descripción de la tool le indica al modelo que solo envíe lo que el usuario señale; es una instrucción, no una garantía.
- Subir un archivo tiene un timeout de 60 s (el resto de peticiones, 10 s). Como en los demás envíos, un timeout no se reintenta con el dominio de respaldo, porque el archivo pudo haberse enviado ya.

## Notas

- La protección real depende de que el endpoint valide `kikes_auth`: si no lo hace, quien conozca la URL puede enviar mensajes.
- Tiempo máximo de espera: 10 s.
- stdout está reservado para el protocolo MCP; los logs salen por stderr.

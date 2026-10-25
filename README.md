# Biznaga 26 — Web Agéntica

> **Construyendo la Web Agéntica: Introducción, Utilidades y Retos de Web-MCP**
> Charla de Raúl Tovar en [BiznagaFest 2026](https://www.biznagafest.com) — 7 de noviembre de 2026, 12:00–12:50, Auditorio Biznaguito.

## Recursos de la charla

¿Vienes del código QR? Aquí tienes todo lo que salió en la charla.

### Las slides

- **PDF de las slides**: [`webmcp-biznaga-2026.pdf`](webmcp-biznaga-2026.pdf)
- **Contenido completo con notas**: [`slides/slides.md`](slides/slides.md)

### Spec y documentación

- **WebMCP (spec)**: [webmachinelearning.github.io/webmcp](https://webmachinelearning.github.io/webmcp/)
- **Explainer e issues**: [github.com/webmachinelearning/webmcp](https://github.com/webmachinelearning/webmcp)
- **Chrome**: [developer.chrome.com/docs/ai/webmcp](https://developer.chrome.com/docs/ai/webmcp)
- **Chrome Status**: [chromestatus.com/feature/5117755740913664](https://chromestatus.com/feature/5117755740913664)
- **MCP (spec 2026-07-28)**: [modelcontextprotocol.io/specification/2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28)
- **Angular**: [angular.dev/ai/webmcp](https://angular.dev/ai/webmcp)
- **Cloudflare**: [blog.cloudflare.com/webmcp](https://blog.cloudflare.com/webmcp/)

### Herramientas

- **knot.kz** (el laboratorio de la demo): [www.knot.kz](https://www.knot.kz/)
- **chrome-debug** (el MCP de la demo 2): [github.com/raultov/chrome-debug-mcp](https://github.com/raultov/chrome-debug-mcp)
- **chrome-devtools-mcp**: [github.com/ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- **Playwright MCP**: [github.com/microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp)
- **Model Context Tool Inspector**: [github.com/beaufortfrancois/model-context-tool-inspector](https://github.com/beaufortfrancois/model-context-tool-inspector)
- **media-transcriber** (MCP para transcribir y capturar vídeo; con él se analizaron los ensayos de esta charla): [github.com/raultov/media-transcriber-mcp](https://github.com/raultov/media-transcriber-mcp)

### Librerías (slide «¿Y mi framework?»)

- **React**: [`use-webmcp-tool`](https://www.npmjs.com/package/use-webmcp-tool) (Google Chrome Labs, experimental)
- **Vue / Nuxt**: [`vue-webmcp`](https://www.npmjs.com/package/vue-webmcp), [`nuxt-webmcp`](https://www.npmjs.com/package/nuxt-webmcp)
- **Polyfill y tipos**: [`@mcp-b/global`](https://www.npmjs.com/package/@mcp-b/global), [`webmcp-types`](https://www.npmjs.com/package/webmcp-types)

### Otros conceptos citados

- **C2PA** (metadatos de procedencia de imágenes y vídeos): [c2pa.org](https://c2pa.org)
- **llms.txt**: [llmstxt.org](https://llmstxt.org)
- **schema.org**: [schema.org](https://schema.org)

---

## ¿Para qué sirve este repo?

Este repositorio reúne **todo el material de la charla** en un solo sitio:

1. **La presentación** ([Slidev](https://sli.dev), 30 diapositivas) lista para proyectar, ensayar o exportar a PDF/PPTX.
2. **La demo en vivo de WebMCP**: el prompt con el que se instruye a un agente para que descubra y ejecute las herramientas que un sitio web publica (`PROMPT.md`).
3. **Las notas de investigación y la bibliografía** de la charla (`notes.txt`).
4. **Las capturas del caso de seguridad C2PA** (el incidente del bridge de Cloudflare) usadas en la sección *Retos*.

En resumen: si quieres **dar o replicar la charla**, este repo tiene todo lo necesario; si solo te interesa **WebMCP**, empieza por `PROMPT.md` y la sección de demo de abajo.

## Contenido

| Ruta | Contenido |
| --- | --- |
| `slides/` | Presentación completa en Slidev (código fuente) |
| `slides/slides.md` | **Todo el contenido de las diapositivas** + notas de ponente |
| `slides/dist-export/` | Exportaciones `webmcp-biznaga-2026.pdf` / `.pptx` (generadas, no versionadas) |
| `webmcp-biznaga-2026.pdf` | PDF de las slides enlazado desde el QR (copia versionada de `slides/dist-export/`) |
| `PROMPT.md` | Prompt para que un agente LLM use las tools WebMCP de un sitio (demo con `chrome-debug` y [knot.kz](https://www.knot.kz)) |
| `notes.txt` | Bibliografía, guion de la demo y notas de argumentación de la charla |
| `images/security/` | Capturas del caso de seguridad C2PA que se analiza en *Retos* |

## Requisitos

- **Node.js >= 22.12** (probado con 24.x vía `nvm install 24`)
- **Google Chrome del sistema** (`google-chrome` en el `PATH`): se usa para exportar a PDF/PPTX; no se descarga el Chromium de Playwright
- Git (para clonar)

No hace falta conexión durante la charla: las fuentes (Press Start 2P, Rajdhani, Inter, JetBrains Mono) y las imágenes están empaquetadas en local.

## Instalación y guía de uso

```bash
git clone https://github.com/raultov/biznaga-26-web-agentica.git
cd biznaga-26-web-agentica/slides
npm install          # con red, una sola vez
npm run dev          # servidor de desarrollo → http://localhost:3030
```

### Comandos disponibles (ejecutar dentro de `slides/`)

| Comando | Qué hace |
| --- | --- |
| `npm run dev` | Levanta Slidev en <http://localhost:3030> · vista de ponente en `/presenter` |
| `npm run build` | Genera la SPA estática en `slides/dist/` (funciona sin red) |
| `npm run preview` | Sirve `slides/dist/` en local (puerto 3030) |
| `npm run export` | Exporta **PDF + PPTX** a `slides/dist-export/` |
| `npm run export:pdf` | Solo PDF → `slides/dist-export/webmcp-biznaga-2026.pdf` (cópialo a la raíz del repo para actualizar el del QR) |
| `npm run export:pptx` | Solo PPTX (con notas de ponente) → `slides/dist-export/webmcp-biznaga-2026.pptx` |
| `npm run export:png` | PNG por diapositiva → `slides/dist-export/png/` |

> **Consejo para el día de la charla:** ejecuta `npm run build` y `npm run preview` de antemano: así la presentación se proyecta desde un build estático y no dependes del servidor de desarrollo.

### Migrar a PowerPoint / Google Slides

Si te exigen el material en otro formato, consulta **[`slides/README.md`](slides/README.md)**: reglas de portabilidad (solo Markdown estándar en `slides.md`), correspondencia de layouts y pasos de migración. La vía rápida es `npm run export:pptx` (cada slide como imagen + notas de ponente; el texto no es editable).

## Demo WebMCP en vivo

La demo muestra cómo un agente descubre y ejecuta las herramientas que una página publica con [WebMCP](https://github.com/webmachinelearning/webmcp), usando el servidor MCP [`chrome-debug`](https://github.com/raultov/chrome-debug-mcp) (abre Chrome con la feature WebMCP habilitada y habla con la pestaña por CDP). El sitio de referencia es **[knot.kz](https://www.knot.kz)** — 6 tools imperativas en la portada y 7 en `/#/contact` (añade el formulario declarativo `contact-knot-team`).

Pasos resumidos (el detalle completo está en [`PROMPT.md`](PROMPT.md)):

1. Añadir `chrome-debug` como servidor MCP en tu cliente (p. ej. opencode).
2. `open_instance` con `label: "webmcp"`, `features: ["WEB_MCP"]`, `headless: false` — el humano tiene que ver la ventana para aprobar acciones.
3. `navigate` a `https://www.knot.kz`.
4. `webmcp_list_tools` — obligatorio antes de ejecutar nada; anota `frameId` y las `annotations` de cada tool.
5. `webmcp_invoke_tool` — solo con los nombres/argumentos devueltos por el listado; las tools con `readOnly: false` requieren aviso previo al usuario.
6. Las tools con consentimiento (portapapeles, formulario) quedan en `Pending`: el humano aprueba en la ventana y se consulta con `webmcp_get_invocation`. **Nunca** pulses tú el botón de enviar.
7. `close_instance` al terminar.

Demo complementaria (índice de código local):

```bash
opencode mcp add knot-local --url http://127.0.0.1:8788/mcp
opencode mcp list
# y en opencode: "¿qué lenguajes soporta el indexador de knot? usa knot-local"
```

## Estructura interna de `slides/`

| Ruta | Contenido |
| --- | --- |
| `slides.md` | Todo el contenido: Markdown plano + notas de ponente en `<!-- -->` |
| `styles/arcade.css` | Tokens del tema arcade BiznagaFest (colores, tipografías) |
| `layouts/` | `cover` (portada), `section` (ROUND N), `end` (CONTINUE?) |
| `global-bottom.vue` | Pie `BIZNAGAFEST 2026 · WEB-MCP — STAGE NN/TT` |
| `public/images/` | Logo y capturas usadas en las diapositivas |

## Estructura de la charla

1. **Introducción** — MCP en 60 segundos, Web-MCP (la página es el servidor), `document.modelContext`, comparativa MCP vs Web-MCP.
2. **Utilidades** — API declarativa (formularios), API imperativa (hooks), anotaciones (`readOnly`, `untrustedContent`) y herramientas.
3. **Demo** — knot.kz bajo demanda.
4. **Retos** — refactorizar para desacoplar, ciclo de vida de la pestaña, costes y abusos, el caso C2PA de Cloudflare, dos bridges en una pestaña y la frontera real entre MCP y Web-MCP.
5. **El futuro** — del buscador al agente, cierre.

## Licencia

Repositorio privado. Los materiales están pensados para la charla de BiznagaFest 2026; para cualquier otro uso, escríbeme primero.

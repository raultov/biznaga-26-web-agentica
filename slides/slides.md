---
theme: default
title: 'Construyendo la Web Agéntica: Introducción, Utilidades y Retos de Web-MCP'
info: BiznagaFest 2026 · Auditorio Biznaguito · 7 nov 2026 · 17:10–18:00
author: Raúl Tovar
colorSchema: dark
aspectRatio: 16/9
canvasWidth: 980
fonts:
  provider: none
highlighter: shiki
lineNumbers: false
transition: slide-left
drawings:
  persist: false
download: false
exportFilename: webmcp-biznaga-2026
layout: cover
eyebrow: BiznagaFest 2026 · Player 1
speaker: Raúl Tovar
role: Senior Software Engineer · The Workshop
when: 07.11.2026 · 17:10
---

# Construyendo la Web Agéntica

Introducción, Utilidades y Retos de Web-MCP

<!--
[0:00] Bienvenida. Último slot antes del coffee break: energía alta desde el principio.
-->

---

# Player 1

- Raúl Tovar, Senior Software Engineer en **The Workshop**
- Backend y frontend: me interesa todo lo que pasa entre la base de datos y el navegador
- Laboratorio de esta charla: **knot.kz**, un side-project donde pruebo WebMCP en producción

<!--
[0:30] 30 segundos máximo. Lo importante: hablo desde las trincheras, no desde el spec.
knot.kz no se vende: es el sitio donde experimento. Se menciona aquí porque todos los ejemplos
de código y la demo salen de él, y el primero aparece ya en la slide de document.modelContext.
TODO: foto, redes, QR a las slides.
-->

---
layout: two-cols
layoutClass: bz-wide-left
---

# La web está hecha para humanos

- Los agentes usan la interfaz pensada para ti: la leen a base de **capturas de pantalla** y DOM
- Frágil: un cambio de CSS rompe al agente
- Caro: miles de tokens para "leer" una página
- Ciego: no sabe qué hace un botón hasta que lo pulsa

> ¿Y si la página pudiera decirle al agente qué sabe hacer?

::right::

<SlidevVideo v-click autoplay autoreset="slide" loop muted playsinline>
  <source src="/videos/datepicker-hell.mp4" type="video/mp4" />
</SlidevVideo>

<!--
[1:00] Hook. Pregunta al público: ¿quién ha visto a un agente pelearse con un datepicker?
[CLICK] aparece y arranca el vídeo (4 s en bucle, sin audio); otro click pasa de slide. El datepicker "orbital", la Tierra gira alrededor del Sol para elegir la fecha.
Es UX pensada (mal) para humanos: ahora imagina a un agente resolviéndolo a base de capturas y clics.
Migración a Google Slides/PowerPoint: insertar public/videos/datepicker-hell.mp4 con reproducción automática y en bucle.
-->

---
layout: section
round: ROUND 1
---

# Introducción

De MCP a Web-MCP

<!--
[3:00]
-->

---

# MCP en 60 segundos

- **Model Context Protocol**: estándar abierto para conectar LLMs con herramientas
- Arquitectura *host → client → server*
- El servidor expone **tools**, *resources* y *prompts*
- Se comunica por stdio (proceso local) o HTTP (servicio remoto)
- Spec actual **2026-07-28**: **sin estado**, cada petición se describe a sí misma

<!--
[3:30] Solo lo justo. Mucha gente ya lo conoce: preguntar a mano alzada.

**Host** = tu CLI (opencode). **Client** = el conector que crea por cada servidor de su config. **Server** = knot-site (www.knot.kz/mcp).

La mayoría de servidores MCP que usa la gente **solo exponen tools** (WebMCP, directamente, solo tiene tools).

Si necesita **credenciales**, se las das tú al configurarlo (token, OAuth), nunca usa tu sesión del navegador.

2026-07-28: **Antes** el cliente abría una sesión con el servidor y la mantenía. **Ahora** no hay sesión,
cada petición llega sola y completa.

-->

---

# Web-MCP: la página es el servidor

- La web publica sus tools **desde el frontend**, con JavaScript o con HTML
- El navegador media cada llamada: la tool corre en la página, **con la sesión del usuario**
- El agente no puede hacer nada que el usuario no pueda
- *Draft* del *W3C Web Machine Learning Community Group* (Microsoft + Google)

> "Instead of your application being a guest within an agent, the agent is a guest on your platform."

<!--
[5:00] Idea clave: el agente puede estar dentro del navegador (Leo en Brave) o fuera (opencode vía chrome-debug),
pero la TOOL siempre se ejecuta DENTRO de la pestaña, con la sesión del usuario.

**Media**: el agente no habla con la página, se lo pide al navegador, que ejecuta la tool y le devuelve el resultado.
**Con la sesión del usuario**: la tool es JS de la web, usa sus cookies y su estado; el agente nunca ve los tokens.

Microsoft y Google empujan este proyecto. No está en el Standards Track del W3C.
Estado comprobado el 01-10-2026: Draft Community Group Report, última edición 30-09-2026.

El agente entra en tu web y juega con tus reglas.
-->

---

# ¿Dónde funciona hoy?

| Navegador / agente | Estado |
| --- | --- |
| Chrome | *Origin trial* desde **149** · flag `#enable-webmcp-testing` |
| Edge | *Origin trial* desde **150** |
| Brave | Experimental en Leo AI |
| ChatGPT Desktop | Soportado |
| Firefox · Safari | *Standards position* abierta |

<!--
[6:00] Fuente: github.com/webmachinelearning/webmcp/blob/main/implementation-status.md (27-09-2026).
NO está activo por defecto en Chrome estable: hace falta origin trial o flag.

**Origin trial**: la web registra su dominio, recibe un token y la API funciona para sus visitantes en Chrome normal.
knot.kz lo tiene desde el 01-10-2026 (caduca el 30-03-2027).
Edge tiene su propio origin trial: el token de Chrome no sirve allí.
-->

---

# MCP vs Web-MCP

| | MCP | Web-MCP |
| --- | --- | --- |
| Dónde vive | Backend | Frontend, en la pestaña |
| Ciclo de vida | Persistente | **Efímero: muere con la pestaña** |
| Sesión | Credenciales propias | La del usuario |
| Descubrimiento | Registro en cada agente | Al visitar la página |
| Interfaz | Headless | Integrado en el DOM |

> "WebMCP is not an extension or a replacement of MCP."

<!--
[6:45] Tabla oficial resumida de developer.chrome.com/docs/ai/webmcp/compare-mcp.
Analogía de Chrome: MCP es el call center, WebMCP el experto en la tienda.
"The most effective agentic applications use both MCP and WebMCP." → lo demostramos en Round 3 con knot.kz.
Web-MCP no tiene resources ni prompts: solo tools.
-->

---

# `document.modelContext`

```js
const controller = new AbortController()

await document.modelContext.registerTool({
  name: 'get-latest-releases',
  description: 'Returns the latest releases of Knot and Knot Server...',
  inputSchema: {
    type: 'object',
    properties: { limit: { type: 'number', minimum: 1, maximum: 4 } },
  },
  annotations: { readOnlyHint: true, untrustedContentHint: true },
  async execute({ limit = 3 }, { signal }) {
    return { content: [{ type: 'text', text: JSON.stringify(await releases(limit)) }] }
  },
}, { signal: controller.signal })   // controller.abort() → desregistra
```

<!--
[7:00] Tools estrictamente tipadas con JSON Schema.
Verificado en Chrome 153 (27-09-2026): solo existe document.modelContext
(navigator.modelContext ya no existe). Miembros: registerTool, getTools, executeTool, ontoolchange.
No hay unregisterTool: se desregistra abortando la señal. registerTool devuelve una Promise.
El abstract habla de navigator.modelContext: contarlo como prueba de lo rápido que cambia el spec.
Fuente: src/webmcp/useWebMcp.ts y src/toolcore/definitions/getLatestReleases.ts de knot-site.
-->

---
layout: section
round: ROUND 2
---

# Utilidades

Dos APIs, un mismo objetivo

<!--
[11:00]
-->

---

# API declarativa: formularios en segundos

```html
<form toolname="contact-knot-team"
      tooldescription="Send a message to the Knot maintainers..."
      action="/api/contact" method="post">
  <input name="email" type="email" required
         toolparamdescription="Reply-to address of the person contacting the team">
  <select name="topic" toolparamdescription="Category: support | bug | other">…</select>
  <textarea name="message" required toolparamdescription="Body of the message"></textarea>
  <button type="submit">Send message</button>
</form>
```

- Cero JavaScript: el navegador **genera el JSON Schema** desde el formulario
- Sin `toolautosubmit`: el agente rellena, **el humano pulsa Send**

<!--
[11:30] Verificado en knot.kz (src/components/Contact.tsx) y en vivo con chrome-debug:
- <select> → enum + anyOf con los textos de cada <option> como title
- required del HTML → required del schema
- El honeypot (company_website) TAMBIÉN entra en el schema: Chrome incluye todo input no hidden.
- La invocación queda Pending hasta que el humano envía; si la página recarga, se queda Pending para siempre.
- CSS: :tool-form-active y :tool-submit-active para resaltar el formulario mientras lo usa el agente.
Según Chrome (docs/ai/webmcp/declarative-api):
- Sin toolparamdescription, el navegador usa el texto del <label> y después aria-description.
- toolautosubmit → el agente envía y navega. SubmitEvent.agentInvoked + respondWith() (tras preventDefault) devuelven un resultado al modelo.
- Eventos toolactivated / toolcancel (el explainer los llama toolcanceled: aún no está cerrado).
- toolparamtitle, que usa knot.kz, no aparece en la documentación oficial.
- El algoritmo de generación del schema es TBD en el explainer: lo que vemos es el comportamiento de Chromium.
-->

---

# API imperativa: un hook de React

```tsx
export function useWebMcp(tools: readonly WebMcpTool[]) {
  useEffect(() => {
    const mc = document.modelContext
    if (!mc) return                  // mejora progresiva: sin API, la web sigue igual

    const controller = new AbortController()
    for (const tool of tools) mc.registerTool(tool, { signal: controller.signal })

    return () => controller.abort()  // desmontar = desregistrar
  }, [tools])
}
```

- La tool llama al **mismo caso de uso** que la UI
- En Vue: lo mismo con `onMounted` / `onUnmounted`

<!--
[13:30] Simplificado de knot-site/src/webmcp/useWebMcp.ts (el real ignora AbortError de StrictMode
y soporta registerTool síncrono o con Promise).
Alternativas: usewebmcp (React, experimental) y soporte experimental en Angular.
execute recibe (input, { signal }) para cancelar. Desde Chrome 153 se puede desregistrar sin romper
ejecuciones en curso. Chrome 155 depreca los argumentos como string JSON.
-->

---
layout: two-cols
---

# Anotaciones

- `readOnlyHint`: la tool no modifica estado
- `untrustedContentHint`: lo que devuelve puede contener texto hostil
- `consequentialHint`: acción irreversible → el agente debería pedir confirmación
- Todas valen `false` por defecto

::right::

```js
// getTools() en knot.kz · Chrome 153
{
  name: 'get-latest-releases',
  origin: 'https://www.knot.kz',
  annotations: {
    readOnlyHint: true,
    untrustedContentHint: true,
  },
}
```

<!--
[15:30] Estas flags son el contrato de confianza entre página y agente.
Salida real de document.modelContext.getTools() capturada el 27-09-2026 (recortada).
Captura de DevTools de respaldo: public/images/flags-knot-readonly.png
Verificado en Chrome 153: getTools() devuelve readOnlyHint / untrustedContentHint; DevTools y CDP
las muestran como readOnly / untrustedContent (de ahí la captura).
consequentialHint está documentado, pero en Chrome 153 no llega al agente (knot.kz lo declara en copy-install-command).
Chrome 156 añade la anotación debugging.
El consentimiento no lo da el navegador: requestUserInteraction() está en el draft pero no en Chrome 153.
En knot.kz es un modal propio (ConsentModal): "the modal is the entire trust boundary".
-->

---
layout: two-cols
layoutClass: bz-wide-right
---

# Herramientas

- **Desarrollo**: flag · **Usuarios**: *origin trial*
- **DevTools**: listar, inspeccionar y ejecutar tools
- *Model Context Tool Inspector*: chatea con tus tools
- **chrome-debug**: conecta cualquier agente MCP con las tools de una pestaña

> Pruébalo: knot.kz → F12 → Application → WebMCP

::right::

![Panel WebMCP de DevTools en knot.kz](/images/devtools-webmcp-panel.png)

<!--
[18:00] Transición a la demo.
Desarrollo: flag #enable-webmcp-testing (en tu navegador). Usuarios: origin trial (token en tu web).
Invitar al público a probarlo: knot.kz tiene origin trial, funciona en su Chrome 149+ sin tocar flags.
Captura: panel WebMCP de DevTools en knot.kz (6 tools, detalle de get-latest-releases con Flags y Run Tool).
Fíjate en "Origin: index-*.js:19": DevTools enlaza cada tool con la línea que la registró.
Model Context Tool Inspector: extensión oficial de Chrome; manda los prompts a gemini-3-flash-preview.
chrome-debug es mío: github.com/raultov/chrome-debug-mcp (open_instance con features WEB_MCP,
webmcp_list_tools, webmcp_invoke_tool, webmcp_get_invocation).
-->

---
layout: section
round: BONUS STAGE
---

# Demo

Player 2 has entered the game

<!--
[20:00] Si algo falla: vídeo de respaldo (TODO: ruta del vídeo).
-->

---

# Lo que vamos a ver

- Las tools de **knot.kz** en DevTools
- Un agente con **chrome-debug** descubre y ejecuta las tools de la pestaña
- Una tool que exige tu permiso y un formulario que solo tú puedes enviar
- El mismo sitio por MCP clásico: qué cruza y qué no

<!--
[20:30] Guion de la demo (agente instruido con PROMPT.md):
1. open_instance WEB_MCP + navigate https://www.knot.kz → webmcp_list_tools: 6 tools
2. webmcp_invoke_tool get-latest-releases {"product":"knot","limit":2} (readOnly + untrustedContent)
3. webmcp_invoke_tool copy-install-command → modal "An agent needs your permission" → Pending →
   el público decide Allow/Deny → webmcp_get_invocation
4. navigate /#/contact → 7 tools → contact-knot-team rellena el formulario, NO lo envía
5. MCP clásico del mismo sitio: opencode.json → "knot-site": { "type": "remote", "url": "https://www.knot.kz/mcp" }
   opencode mcp list → solo 4 tools (las portables)
   Prompt: "¿qué lenguajes soporta el indexador de knot? usa knot-site"
-->

---
layout: section
round: ROUND 3
---

# Retos

Hablando desde las trincheras

<!--
[28:00]
-->

---

# Refactorizar para desacoplar

- La lógica atrapada en componentes no se puede exponer
- Una tool es un **caso de uso**, no un botón
- Stores y servicios compartidos entre UI y agente

<!--
[28:30] Ejemplo knot.kz: installationStore + scrollToInstall.
-->

---

# Ciclo de vida: la pestaña manda

- Sin pestaña abierta, **no hay tools**
- Cada ruta publica sus tools: knot.kz tiene 6 en la portada y 7 en `/#/contact`
- Si la página navega a mitad de una llamada, **la respuesta se pierde**
- Sesión del usuario = poder del usuario

> "WebMCP tools are ephemeral. They exist only when your page is open."

<!--
[30:30] Verificado en vivo: una invocación del formulario queda Pending y, tras recargar, sigue
Pending para siempre. Es el issue abierto #135 del explainer ("cross-document tool response").
executeTool() devuelve null cuando la tool provoca una navegación.
Propuesta en estudio: tools en Service Workers para sitios que no están abiertos.
-->

---

# Costes y abusos

- Cada tool ocupa **contexto** del modelo en cada turno
- Presupuestos de Chrome: **30** caracteres el nombre, **500** la descripción, **150** cada parámetro, **1.500** la respuesta
- *Rate limiting* y bots: tu web se convierte en una API
- Menos tools, mejor descritas: *validate strictly in code, loosely in schema*

<!--
[32:00] "While there isn't a maximum number of tools allowed, each tool takes up part of the context window."
(docs/ai/webmcp/best-practices). Los límites están en docs/ai/webmcp/secure-tools y "subject to change".
knot.kz lo vigila en CI: scripts/webmcp-budget.mjs falla el build si una tool supera el presupuesto
(objetivo propio: 1.350 caracteres de salida, el 90 % del límite).
Ninguna guía oficial habla de coste económico: es un reto abierto.
-->

---

# Seguridad: el caso C2PA

| Tool | Devuelve | Flags declarados |
| --- | --- | --- |
| `get-latest-releases` · knot.kz | CHANGELOG de GitHub | `readOnly` · `untrustedContent` |
| `inspect_image_c2pa` · Cloudflare | Metadatos que escribe **el autor de la imagen** | **ninguno** |

- Sin `untrustedContentHint`, el agente trata la salida como fiable
- Un manifiesto C2PA manipulado → **prompt injection**

> "It's impossible to guarantee safety inside of a large language model."

<!--
[34:00] Datos sacados del panel WebMCP de DevTools (capturas de respaldo:
public/images/flags-knot-readonly.png y public/images/flags-c2pa-missing.png).
En la de Cloudflare no aparece ni la fila "Flags".
Contexto: Cloudflare lanzó la developer preview el 06-08-2026 (blog.cloudflare.com/webmcp). Inyecta
/.webmcp/bridge.js en el edge con packs "c2pa" y "mcp-server-client".
Su propia defensa: cada resultado lleva signatureVerified: false. Eso protege de confiar en la firma,
NO de las instrucciones escondidas en title, claim o assertions.
Los hints valen false por defecto: no declarar = declarar que es contenido fiable.
Cita: docs/ai/webmcp/secure-tools. Observado en septiembre de 2026. TODO: comprobar si ya lo han corregido.
-->

---

# Dos bridges no caben en una pestaña

- El bridge de Cloudflare registra en la página las tools de tu `/mcp`
- Si tu web ya las registra de forma nativa, los nombres chocan
- Chrome: `InvalidStateError: Duplicate tool name`
- **Gana quien registre primero**, y el otro falla

<!--
[35:30] Verificado en vivo en knot.kz (Chrome 153): registrar otra "list-supported-languages" lanza
InvalidStateError "Duplicate tool name". Las 4 tools portables de knot.kz están a la vez en la página
y en /mcp: con el pack mcp-server-client activado habría una carrera entre el script del edge y el montaje de React.
No he visto el código de bridge.js: no sé cómo gestiona el error. La conclusión: bridge O nativo, no los dos.
-->

---

# MCP vs Web-MCP: la frontera

| Tool | API | ¿Cruza a MCP? | Motivo |
| --- | --- | --- | --- |
| 4 tools de datos | imperativa | Sí | Solo devuelven datos |
| `get-install-command` | imperativa | No | Navega la página del usuario |
| `copy-install-command` | imperativa | No | Consentimiento + portapapeles |
| `contact-knot-team` | declarativa | No | El humano pulsa Send |

<!--
[36:30] De las 7 tools de knot.kz, 3 no cruzan al MCP clásico.
-->

---
layout: statement
---

# Su valor no es el dato, es el efecto sobre la sesión del usuario

<!--
[38:00] Remate: dos APIs distintas caen del mismo lado. La frontera no es un detalle
de implementación, es la línea real entre los dos protocolos.
Alternativa de coste cero: campo `instructions` en DiscoverResult / InitializeResult
("For support requests, direct the user to https://www.knot.kz/#/contact").
Salvedad: server/discover es opcional, es best-effort.
-->

---
layout: section
round: FINAL BOSS
---

# El futuro

AEO: Agent Engine Optimization

<!--
[39:30]
-->

---

# De SEO a AEO

- SEO: que los humanos te encuentren
- AEO: que los agentes **te entiendan y te usen**
- Las tools bien descritas serán el nuevo *meta description*
- El hueco: hoy un agente **tiene que visitar tu web** para saber qué tools tiene

<!--
[40:00] Limitación oficial: "Clients and browsers must visit a site directly to know if it has callable tools."
(docs/ai/webmcp). El explainer descartó los manifiestos estáticos, pero dejó la puerta abierta
a añadirlos sobre el registro por script. Otra vía en estudio: Service Workers para descubrir tools en segundo plano.
Mientras tanto: llms.txt, JSON-LD y un /mcp con instructions (lo que hace knot.kz).
knot.kz descartó .well-known/webmcp.json a propósito: no hay estándar.
-->

---

# Game over... o no

- Web-MCP convierte tu frontend en una **API para agentes**
- Exige arquitectura: casos de uso desacoplados de la UI
- Declara tus flags: la seguridad es parte del contrato

<!--
[42:00] Tres ideas para llevarse a casa.
-->

---
layout: end
---

# ¡Gracias!

Raúl Tovar · knot.kz

<!--
[43:00] Q&A. TODO: QR a las slides y redes.
-->

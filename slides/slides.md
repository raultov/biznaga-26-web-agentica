---
theme: default
title: 'Construyendo la Web Agéntica: Introducción, Utilidades y Retos de Web-MCP'
info: BiznagaFest 2026 · Auditorio Biznaguito · 7 nov 2026 · 12:00–12:50
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
when: 07.11.2026 · 12:00
---

# Construyendo la Web Agéntica

Introducción, Utilidades y Retos de Web-MCP

<!--
[0:00] Bienvenida. Primer slot tras el desayuno: la gente llega del café, arranca con energía.
-->

---
layout: two-cols
---

# Player 1

- Raúl Tovar, Senior Software Engineer en **The Workshop**
- Backend y frontend: me interesa todo lo que pasa entre la base de datos y el navegador
- **knot**: indexador de código semántico para agentes de IA · Rust, open source (MIT), servidor MCP y CLI
- Su web, **knot.kz**, ya ofrece tools Web-MCP en producción: será el laboratorio de esta charla

::right::

![Portada de knot.kz: Knot, AI Codebase Indexer](/images/knot-home.png)

<!--
[0:35] 30 segundos máximo. Lo importante: hablo desde las trincheras, no desde el spec.

knot es un producto mío, open source, que ya usan agentes como Claude Code u opencode. Su web tiene tools Web-MCP de verdad, y por eso la uso como laboratorio. Son tools de una web pequeña, así que no esperéis un e-commerce.

Todos los ejemplos de código y la demo salen de knot.kz.
Datos si preguntan: v1.12.1 (27-10-2026), más de 100 versiones desde abril de 2026; también existe knot-server.
-->

---

# La web está hecha para humanos

- Los agentes usan la interfaz pensada para ti: la leen a base de **capturas de pantalla** y DOM
- Frágil: el CSS puede despistar al agente
- Caro: miles de tokens para "leer" una página
- Ciego: si un banner tapa un botón, la captura no lo ve

> ¿Y si la página pudiera decirle al agente qué sabe hacer?

<!--
[1:30] Hook. Pregunta al público: ¿quién ha visto a un agente pelearse con un datepicker?

No hace falta saber de frameworks ni de LLMs: lo que necesitas lo vamos viendo.
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
[3:15] Solo lo justo. Mucha gente ya lo conoce: preguntar a mano alzada.

**Host** = tu CLI (opencode). **Client** = el conector que crea por cada servidor de su config. **Server** = knot-site (www.knot.kz/mcp).

La mayoría de servidores MCP que usa la gente **solo exponen tools** (WebMCP, directamente, solo tiene tools).

Si necesita **credenciales**, se las das tú al configurarlo (token, OAuth), nunca usa tu sesión del navegador.

2026-07-28: **Antes** el cliente abría una sesión con el servidor y la mantenía. **Ahora** no hay sesión,
cada petición llega sola y completa.

-->

---

# Web-MCP: la página es el servidor

<v-clicks>

- Web-MCP es una propuesta de API web, inspirada en MCP, que aún no es estándar
- *Draft* del *W3C Web Machine Learning Community Group* (Microsoft + Google)
- La web publica sus tools **desde el frontend**, con JavaScript o con HTML
- El navegador media cada llamada: la tool corre en la página, **con la sesión del usuario**

</v-clicks>

<v-click>

![Evolución de las cargas de página en Chrome que registran tools Web-MCP](/images/webmcp-adopcion.png)

</v-click>

<!--
[5:15] [CLICK] Web-MCP copia ideas de MCP, como las tools (nombre, descripción, schema de entrada), pero no el protocolo:
no hay JSON-RPC entre la página y el agente, es una API de JavaScript del navegador.

[CLICK] Microsoft y Google empujan este proyecto.
Sigue sin ser estándar, pero el W3C ya está redactando el charter de un Working Group que lo adoptaría: es el siguiente paso del camino de WebAssembly.
Estado comprobado el 25-10-2026: Draft Community Group Report del 20-10-2026; charter del Agentic Web WG en borrador (w3c/strategy #576), sin aprobar.
WebAssembly nació en un Community Group y hoy es estándar W3C; WebGPU también, y ya está en Candidate Recommendation.

[CLICK] El agente puede estar dentro del navegador (Leo en Brave) o fuera (opencode vía chrome-debug),
pero la TOOL siempre se ejecuta **DENTRO de la pestaña**.

[CLICK] **Media**: el agente no habla con la página, se lo pide al navegador, que ejecuta la tool y le devuelve el resultado.
**Con la sesión del usuario**: la tool es JS de la web y usa lo que tenga esa pestaña (cookies, estado, login si lo hay); el agente nunca ve las credenciales.
El agente actúa con los permisos del usuario, no con más.

El agente entra en tu web y juega con tus reglas.

[CLICK] Adopción real: casi el 1 % de las cargas de página en Chrome ya registran tools, sobre todo tiendas Shopify desde julio. Pero solo 1 de cada 11 millones ejecuta una: las webs ya están listas, los agentes todavía no.
(Fuente: chromestatus.com, contadores 5791 y 5799, 27-10-2026.)
El salto es Shopify. Cloudflare es opt-in y en beta: no lo vincules al 1 %.
-->

---

# ¿Dónde funciona hoy?

| Navegador / agente | Estado |
| --- | --- |
| Chrome | *Origin trial* desde **149** · flag `#enable-webmcp-testing` |
| Edge | *Origin trial* desde **150** |
| Brave | Experimental en Leo AI · flag `#enable-webmcp-testing` |
| Firefox | Postura **neutral** · prototipo, sin fecha |
| Safari | **En contra** |
| ChatGPT (escritorio) | **Site tools** en su navegador integrado (agosto 2026) |

<!--
[7:15] NO está activo por defecto en Chrome estable: hace falta origin trial o flag.

**Origin trial**: knot.kz registra su dominio, recibe un token y la API funciona para sus visitantes en Chrome normal.
lo tiene desde el 01-10-2026 (caduca el 30-03-2027).
Edge tiene su propio origin trial: el token de Chrome no sirve allí.

**Firefox**: neutral; ven valor, pero temen webs que engañen al agente. Prototipo empezado, sin fecha.

**Safari**: en contra; el agente debe usar la web como el humano (HTML), sin una capa aparte solo para él.

OpenAI ya es cliente: ChatGPT de escritorio usa las tools WebMCP de la página que tienes abierta (no en Chrome, en su navegador integrado).

Shopify ya sirve tools Web-MCP en sus tiendas, sin que el comerciante toque nada. Plataformas como Shopify o Cloudflare
son las que de verdad extienden la adopción: lo activan en todas sus webs de golpe.
-->

---

# MCP vs Web-MCP

| | MCP | Web-MCP |
| --- | --- | --- |
| Dónde vive | Fuera del navegador | Dentro de la pestaña |
| Duración | Mientras el servidor MCP esté en marcha | **Mientras la pestaña esté abierta** |
| Configuración | Requiere configuración en tu cliente | Basta con abrir la página |
| El usuario ve | Solo la respuesta del agente | La página cambiando en directo |

> "WebMCP is not an extension or a replacement of MCP."

<!--
[9:00] De hecho, las aplicaciones agénticas más eficaces usan MCP y WebMCP a la vez. → knot.kz tiene las dos y en la demo usamos ambas.

Shopify también: WebMCP en todas sus tiendas para el agente del comprador, y servidores MCP para quien construye su propio agente.
Cada tienda tiene su endpoint MCP (`/api/ucp/mcp`, protocolo UCP): catálogo, carrito, checkout y pedidos, sin navegador.
Lo usa un asistente de compras de terceros, que se identifica con un perfil de agente; el checkout exige autenticar o firmar cada petición.

Web-MCP no tiene resources ni prompts: solo tools.
**Resources**: la página ya es el recurso.
**Prompts**: los lanza el usuario desde la interfaz de su agente.
-->

---
layout: section
round: ROUND 2
---

# Utilidades

Dos APIs, un mismo objetivo

<!--
[10:15]
-->

---
class: bz-code-compact
---

# API imperativa

```js {1-2|4-8|10-16|18-20|all}
// 1. Lo que el agente puede mandar
const inputSchema = { type: 'object', properties: { limit: { type: 'number', minimum: 1, maximum: 4 } } }

// 2. Lo que se ejecuta en la página
async function execute(input) {
  const data = await releases(input.limit ?? 3)   // código que la web ya tenía
  return { content: [{ type: 'text', text: JSON.stringify(data) }] }
}

// 3. La tool completa
const tool = {
  name: 'get-latest-releases',
  description: 'Returns the latest releases of Knot and Knot Server...',
  annotations: { readOnlyHint: true, untrustedContentHint: true },
  inputSchema, execute,
}

// 4. Registrarla; controller.abort() la quita
const controller = new AbortController()
await document.modelContext.registerTool(tool, { signal: controller.signal })
```

<!--
[10:30] 

1: input schema, qué puede mandar el agente. 

[CLICK] 2: execute, el código de siempre de la web.

[CLICK] 3: nombre, descripción y anotaciones, lo que lee el modelo. 

[CLICK] 4: registrar; abort() la quita. [CLICK] todo.

Verificado en Chrome 153 (27-09-2026): solo existe document.modelContext
(navigator.modelContext ya no existe). 

Miembros: registerTool, getTools, executeTool, ontoolchange.

No hay unregisterTool: se desregistra abortando la señal. registerTool devuelve una Promise.

El abstract habla de navigator.modelContext: contarlo como prueba de lo rápido que cambia el spec.
-->

---

# API declarativa: formularios en segundos

```html {1-2|4-7|3,8|all}
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

- Sin JavaScript obligatorio: el navegador **genera el JSON Schema** desde el formulario
- Sin `toolautosubmit`: el agente rellena, **el humano pulsa Send**

<!--
[12:00]

1: toolname + tooldescription, lo que lee el modelo (como el bloque 3 de la imperativa).

[CLICK] 2: los campos; el navegador genera el JSON Schema solo (required → required, `<select>` → enum).

[CLICK] 3: action + botón; no hay execute: ejecutar es enviar el formulario, y lo envía el humano.

[CLICK] todo. Sin registerTool ni abort(): la tool existe mientras el formulario esté en la página.
El JS es opcional: `respondWith()` para devolver un resultado propio sin navegar. knot.kz lo usa.

Hace unas semanas sacaron la declarativa del draft: Chrome quiere lanzar primero la imperativa, y la declarativa vendrá después. Sigue en un documento aparte, se sigue trabajando en ella y funciona en el navegador, como veréis en la demo.
(PR #338, 08-10-2026; Beaufort: "ship the imperative API first, with the declarative API following shortly after".)

Verificado en knot.kz (src/components/Contact.tsx) y en vivo con chrome-debug:
- `<select>` → enum + anyOf con los textos de cada `<option>` como title
- required del HTML → required del schema
- El agente se queda esperando (Pending) hasta que el humano envía; si la página navega, recibe el JSON-LD de la página nueva.
- CSS: :tool-form-active y :tool-submit-active para resaltar el formulario mientras lo usa el agente.
Según Chrome (docs/ai/webmcp/declarative-api):
- Sin toolparamdescription, el navegador usa el texto del `<label>` y después aria-description.
- toolautosubmit → el agente envía y navega. SubmitEvent.agentInvoked + respondWith() (tras preventDefault) devuelven un resultado al modelo.
- Eventos toolactivated / toolcancel (el explainer los llama toolcanceled: aún no está cerrado).
- toolparamtitle, que usa knot.kz, no aparece en la documentación oficial.
- El algoritmo de generación del schema es TBD en el explainer: lo que vemos es el comportamiento de Chromium.

Probado en knot.kz (Chrome 153, 03-10-2026):
- agentInvoked es true si el agente rellenó, aunque el humano pulse Send.
- preventDefault() sin respondWith() → Chrome da la invocación por error ("The site has a programming error").
- Con respondWith() el agente recibe exactamente lo que devuelve la página.
-->

---

# ¿Y mi framework?

| Framework | Soporte |
| --- | --- |
| Angular | **Oficial**, experimental desde **v22** · Signal Forms → tool |
| React | Sin soporte oficial · `use-webmcp-tool` (Google Chrome Labs, experimental) |
| Vue / Nuxt | Comunidad: `vue-webmcp`, `nuxt-webmcp` |
| Cualquiera | Polyfill y tipos: `@mcp-b/global`, `webmcp-types` |

> knot.kz: **sin librerías**, un hook de ~70 líneas. Una librería no te ahorra código: te aísla de los **cambios de la spec**.

<!--
[13:45] Angular es el único con soporte **oficial**: desde la v22, en experimental. Puede cambiar incluso en versiones menores.
Tú no tocas `document.modelContext`: si el navegador cambia, lo absorbe Angular.

React: `use-webmcp-tool` es de Google Chrome Labs, no de Meta; la recomienda la doc de Chrome. Vue: solo librerías de la **comunidad**.

Con un **polyfill**, tus tools funcionan en cualquier navegador para clientes que ejecuten JavaScript en la página, como Playwright o una extensión.
Pero se pierde la mediación del navegador: ese cliente tiene acceso a toda la página, no solo a tus tools.

knot.kz no usa ninguna: la API es una función y una señal de cancelación.
Lo que te da una librería es que **otro** sigue los cambios de la spec por ti.
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
// getTools() en knot.kz · Chrome 155
{
  name: 'get-latest-releases',
  origin: 'https://www.knot.kz',
  annotations: {
    consequentialHint: false,
    readOnlyHint: true,
    untrustedContentHint: true,
  },
}
```

<!--
[15:45] Estas flags son lo que la página **declara** al agente: el navegador las transmite, pero no las hace cumplir.

**Excepción** en la API declarativa: **sin toolautosubmit**, el navegador rellena pero no envía; eso sí lo impone.
El navegador controla quién llama a qué y desde dónde (solo HTTPS, tools ligadas a su origen); lo que la tool declara o hace es responsabilidad de la página.
Si algo debe confirmarse, confírmalo en la tool.

consequentialHint ya llega al agente en Chrome 155 (en la 153 no), pero sigue siendo una pista: confirmar o no lo decide el agente.

Hoy el consentimiento lo pone la página (el ConsentModal de knot.kz). Lo que cambia es el motivo: el navegador no tiene ninguna API para pedirlo, ni siquiera en el draft.
-->

---
layout: two-cols
layoutClass: bz-wide-right bz-spotlight-last
---

# Herramientas

- **Desarrollo**: flag · **Usuarios**: *origin trial*
- **DevTools**: listar, inspeccionar y ejecutar tools
- *Model Context Tool Inspector*: chatea con tus tools
- **chrome-devtools-mcp**, **Playwright MCP** y **chrome-debug**: conectan cualquier agente MCP con las tools de una pestaña

::right::

![Panel WebMCP de DevTools en knot.kz](/images/devtools-webmcp-panel.png)

<!--
[17:00] Transición a la demo.

**Desarrollo**: flag #enable-webmcp-testing (en tu navegador; también lo necesita el Model Context Tool Inspector). Usuarios: origin trial (token en tu web).
Invitar al público a probarlo: knot.kz tiene origin trial, funciona en su Chrome 149+ sin tocar flags.

Model Context Tool Inspector: extensión de François Beaufort (equipo de Chrome), no es producto oficial de Google. Lista y ejecuta tools, y chatea con ellas con tu API key de Gemini (por defecto gemini-3.6-flash). Requiere el flag `#enable-webmcp-testing` (Chrome 150+), aunque la web tenga origin trial.

chrome-devtools-mcp: WebMCP tras el flag --categoryExperimentalWebmcp. Playwright MCP: activado por defecto.

Chrome 156 añade la anotación debugging para desarrolladores y testing: los agentes del usuario final pueden filtrar esas tools.
En la demo uso **chrome-debug**, un MCP mío más ligero que estos dos: github.com/raultov/chrome-debug-mcp. Invitar a probarlo.
-->

---
layout: section
round: BONUS STAGE
---

# Demo

Player 2 has entered the game

<!--
[18:45] Demo grabada: avisa de que es un vídeo. Tres caminos, uno por slide.
-->

---

# Lo que vamos a ver

Misma pregunta, tres caminos: *¿qué lenguajes soporta knot?*

1. **Leo** en Brave: el agente está **dentro** del navegador
2. **opencode + chrome-debug**: el agente está **fuera**, la tool corre en la pestaña
3. **opencode + knot-site**: MCP clásico, knot.kz ni se abre

<!--
[19:00] Del más integrado al menos integrado. Cierra con la cita: en 1 y 2 el agente es el invitado; en 3, tu web.
-->

---
demoVideo: true
---

# Demo 1 · Leo en Brave

<SlidevVideo v-click autoplay muted controls autoreset="slide" playsinline class="bz-video-full" print-timestamp="85">
  <source src="/videos/brave-leo.mp4" type="video/mp4" />
</SlidevVideo>

<!--
[19:45] Intro → pulsa → y arranca solo a toda la slide (2:44, sin audio). Pausa con el ratón. → otra vez: siguiente slide.

Brave con brave://flags/#enable-webmcp-testing, modelo fijado: Qwen 3.5 35B.
www.knot.kz → abrir Leo → tools pill (6 tools) → "usa las tools para decirme qué lenguajes soporta knot"
→ Security warning de Brave (Allow) → list-supported-languages → 15 lenguajes.
Después, "ejecuta copy_install_command" → DOBLE consentimiento, cada uno de un actor:
  1. Brave (el navegador media): "Brave AI would like to execute copy-install-command" → Allow
  2. knot.kz (la web pone sus reglas): "An agent needs your permission" → Allow → copiado.
Brave muestra su Security warning en CADA ejecución, aunque se repita la tool (salvo "Allow always" en la tools pill).
Señalar el nombre en "Task complete": web_www_knot_kz_copy_install_command. Brave añade el origen como prefijo.
-->

---
demoVideo: true
---

# Demo 2 · opencode + chrome-debug

<SlidevVideo v-click autoplay muted controls autoreset="slide" playsinline class="bz-video-full" print-timestamp="70">
  <source src="/videos/opencode-chrome-debug.mp4" type="video/mp4" />
</SlidevVideo>

<!--
[22:30] Intro → pulsa → y arranca solo a toda la slide (1:24, sin audio). Pausa con el ratón. → otra vez: siguiente slide.

navigate knot.kz/#/contact → webmcp_list_tools: 7 tools → webmcp_invoke_tool contact-knot-team con datos de test.
El NAVEGADOR rellena el formulario y NO lo envía (no hay toolautosubmit). La llamada sale en rojo: es el timeout de chrome-debug, la invocación sigue en Pending.
El agente consulta: Pending, "espera a que un humano pulse Send message". Pulso Send → "Message sent".
No hay ningún diálogo de consentimiento: lo que falta es que el humano envíe.
-->

---
demoVideo: true
---

# Demo 3 · opencode + knot-site

MCP clásico: knot.kz ni se abre

```json
"knot-site": {
  "type": "remote",
  "url": "https://www.knot.kz/mcp"
},
```

<SlidevVideo v-click autoplay muted controls autoreset="slide" playsinline class="bz-video-full" print-timestamp="70">
  <source src="/videos/knot-site.mp4" type="video/mp4" />
</SlidevVideo>

<!--
[24:15] Intro con la config → pulsa → y arranca solo a toda la slide (1:36, sin audio). Pausa con el ratón. → otra vez: siguiente slide.

La config de la izquierda es la entrada "mcp" de opencode.json: no hay navegador, solo una URL.
"¿qué lenguajes soporta el indexador de knot? usa knot-site" → respuesta en el terminal.
Solo 4 tools: copy-install-command, get-install-command y contact-knot-team no cruzan.
-->

---
layout: section
round: ROUND 3
---

# Retos

Hablando desde las trincheras

<!--
[26:00]
-->

---
class: bz-examples
---

# Refactorizar para desacoplar

- Lógica y estado **fuera del componente**: la UI y la tool usan el mismo código
  - <v-click><code>addToCart</code> dentro de un <code>onClick</code>: llévalo a un servicio, store o hook que la tool pueda llamar</v-click>
- Una tool es un **caso de uso**, no un botón
  - <v-click><code>checkout(...)</code>, no "pulsa Siguiente" tres veces</v-click>
- Las **reglas**, en el modelo: la tool no pasa por tus botones ni formularios
  - <v-click>Un botón "Comprar" deshabilitado sin stock no frena a la tool</v-click>

<!--
[26:15] Habéis visto que las tools funcionan. Ahora viene lo difícil: que tu código esté preparado para tenerlas.
Si la lógica vive en el botón, la tool tiene dos opciones: duplicarla o pulsar el botón. Las dos son malas.

**Fuera del componente**: execute no es un onClick, solo puede llamar a funciones normales.
[CLICK] No hace falta salir del framework: basta con que la lógica no viva en el manejador del botón. Un módulo sin framework solo compensa si la lógica se usa también fuera del navegador, como el /mcp de knot.kz.
En knot.kz, la pestaña activa: la tool la cambia y React repinta.

En Angular viene casi hecho: el `execute` hace `inject()` del mismo servicio que usa el botón, y con Signal Forms la tool comparte validadores y envío con el formulario.

**Caso de uso**: expón lo que el usuario quiere conseguir, no sus clics.
[CLICK] En knot.kz: get-install-command, y además calcula el docker run con tus núcleos y RAM. Más que cualquier botón.

**Reglas en el modelo**: lo que la UI impide sin decirlo, la tool no lo ve.
[CLICK] Ni un botón deshabilitado ni un required del formulario frenan al agente: comprueba en el modelo.
En knot.kz, get-install-command valida núcleos y RAM dentro de la tool.
-->

---

# Ciclo de vida: la pestaña manda

- Sin pestaña abierta, **no hay tools**
- Cada ruta publica sus tools: knot.kz tiene 6 en la portada y 7 en `/#/contact`
- Sesión del usuario = poder del usuario

> "WebMCP tools are ephemeral. They exist only when your page is open."

<!--
[29:30] Un servidor MCP está siempre ahí. Una tool de Web-MCP solo existe mientras alguien tiene tu página abierta, y eso lo cambia todo.

**Sin pestaña, no hay tools**: un agente no puede usar tu web a las 3 de la mañana sin que alguien la tenga abierta. Para eso está MCP.

**Cada ruta, sus tools**: lo visteis en la demo. En la portada, 6; al entrar en contacto aparece contact-knot-team. Las tools siguen a la pantalla.

**Sesión = poder**: la tool hace lo que el usuario podría hacer, ni más ni menos. Sin login, lo de un visitante anónimo.
-->

---

# Costes y abusos

<v-clicks>

- Cada tool ocupa **contexto** del modelo en cada llamada: las 7 de knot.kz son **~1.000 tokens**
- Presupuestos **recomendados** por Chrome (no los aplica): **30** el nombre, **500** la descripción, **150** cada parámetro, **1.500** la respuesta
- *Rate limiting* y bots: tus tools son una **API documentada**, que se puede usar sin LLM
- **Menos tools, mejor descritas**: menos tokens y menos llamadas fallidas

</v-clicks>

<!--
[31:00] 
[CLICK] OJO con la cifra: ~1.000 tokens son las 7 tools JUNTAS, no cada una. Se reenvían en cada paso: 3 pasos = 3.000.
las API de los LLM no guardan estado entre llamadas. En cada paso, el agente envía al modelo la conversación entera más la lista de tools disponibles
Una tarea de 3 pasos en knot.kz son unos 3.000 tokens solo en releer el menú. Y al navegar el catálogo cambia, así que la caché ayuda menos.
Aun así suele salir más barato que leer el DOM o hacer capturas: el reto es no inflar el catálogo.

[CLICK] Presupuestos recomendados; Chrome no los aplica; knot.kz los vigila en CI. (UNA frase)

[CLICK] Abusos: un script puede llamar a tus tools sin modelo. En knot.kz solo contact cuesta dinero, y está protegido. (UNA frase)

RESPALDO para preguntas, no decirlo:
- Probado en Chrome 153: acepta un nombre de 60 caracteres y una respuesta de 5.000 sin quejarse. En knot.kz lo que va justo son las respuestas (1.321 de 1.500).
- scripts/webmcp-budget.mjs falla el build si una tool supera el presupuesto (objetivo propio: 1.350 caracteres, el 90 %).
- Protección de contact: honeypot, validación en servidor y rate limit en Cloudflare.

[CLICK] **Menos tools, mejor descritas**: cada tool se paga en cada paso, y una descripción confusa hace que el agente elija mal y repita.
knot.kz agrupa 5 combinaciones en get-install-command, pero separa copy-install-command porque pide consentimiento.
-->

---

# Seguridad: el caso C2PA

| Tool | Devuelve | Flags declarados |
| --- | --- | --- |
| `get-latest-releases` · knot.kz | CHANGELOG de GitHub | `readOnly` · `untrustedContent` |
| `inspect_image_c2pa` · Cloudflare | Metadatos que escribe **el autor de la imagen** | **ninguno** |

- Sin `untrustedContentHint`, el agente trata la salida como fiable
- Un manifiesto C2PA manipulado → **prompt injection**
- Lo reporté a Cloudflare (HackerOne): cerrado como *Informative*, "expected behavior"

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
Cita: docs/ai/webmcp/secure-tools. Observado en septiembre de 2026.

Reporte (06-09-2026, HackerOne): sin anotación de contenido no fiable no es vulnerabilidad para su política; es "expected behavior".
Los fallos del bridge (CSP, nombres duplicados) los ven como calidad, no seguridad. Pero "useful for the ongoing development".

Moraleja, corta: "mejor que pase ahora, en beta, que después".

RESPALDO para preguntas, no decirlo (era la slide "Dos bridges no caben en una pestaña", quitada el 04-11-2026):
- El otro pack del bridge, **Site MCP server**, registra en la página las tools de tu `/mcp`; cada llamada va a tu servidor con la cookie del usuario, sin OAuth.
- Si tu web ya las registra de forma nativa, los nombres chocan (`InvalidStateError: Duplicate tool name`) y gana quien registre primero.
- Probado en producción el 05-10-2026: con red normal ganan las nativas 10 de 10 (y el bridge deja 4 errores sin capturar en la consola); con red lenta y sin caché gana el bridge 3 de 3, y get-latest-releases pierde su untrustedContent: el bridge solo copia nombre, descripción y schema.
- Conclusión: bridge O nativo, no los dos.
-->

---
layout: section
round: FINAL BOSS
---

# El futuro

Que te encuentren ya no basta

<!--
[36:30]
-->

---

# Del buscador al agente

<v-clicks>

- Ya no basta con que te encuentren: los agentes tienen que **entenderte y usarte**
- Las descripciones de tus tools deciden si el agente **te usa**, como el texto bajo cada resultado de Google influye en si el humano hace clic
- El hueco: hoy un agente **tiene que visitar tu web** para saber qué tools tiene
- Mientras tanto, **deja un cartel**: un <code>llms.txt</code> que diga qué tools tienes y dónde
- ¿Posicionará tener tools?

</v-clicks>

<!--
[36:45] [CLICK] Entenderte y usarte. [CLICK] Las descripciones deciden (párrafo de Chrome, más abajo).

[CLICK] Limitación oficial: Clientes y navegadores deben visitar un site directamente para saber si tiene tools

Nadie indexa tools, y el explainer descartó un manifiesto fijo: las tools cambian con la ruta y el usuario.
[CLICK] Mientras tanto, un cartel: llms.txt, un resumen en Markdown para modelos en la raíz de la web.
El de knot.kz dice qué tools hay y en qué página (www.knot.kz/llms.txt).

Chrome lo dice en best-practices: la descripción decide qué tool elige el agente, y tiene una guía de Evals para medirlo. Pero ojo: Search Engine Journal y Moz (agosto 2026) coinciden en que WebMCP no es ranking ni indexación; actúa cuando el agente ya está en tu web. Lo que muestra Google se lee antes de visitar la web.

[CLICK] **Futuro**: ¿Posicionará tener tools? Hoy no: nadie lo usa como señal, y Search Engine Journal no encontró preferencia de los agentes por webs con WebMCP. Pero el precedente es schema.org: no sube el ranking, sí habilita resultados enriquecidos. Un buscador que indexe tools cerraría el hueco.
-->

---

# Game over... o no

- Su valor está en el **efecto sobre la sesión**: si la tool tiene sentido sin navegador, publícala también en MCP
- Web-MCP convierte tu frontend en una **API para agentes**
- Exige arquitectura: casos de uso desacoplados de la UI
- Declara tus flags y **valida en código**: la seguridad es parte del contrato

<!--
[39:00] Cuatro ideas para llevarse a casa.

Punto 1: publícala **también** en MCP, no "en vez de" Web-MCP. knot.kz tiene las dos.
-->

---
layout: two-cols
layoutClass: bz-links
---

# Recursos

## Spec y documentación

- **WebMCP**: [webmachinelearning.github.io/webmcp](https://webmachinelearning.github.io/webmcp/)
- **Explainer e issues**: [github.com/webmachinelearning/webmcp](https://github.com/webmachinelearning/webmcp)
- **Chrome**: [developer.chrome.com/docs/ai/webmcp](https://developer.chrome.com/docs/ai/webmcp)
- **MCP**: [modelcontextprotocol.io/specification/2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28)
- **Angular**: [angular.dev/ai/webmcp](https://angular.dev/ai/webmcp)
- **Cloudflare**: [blog.cloudflare.com/webmcp](https://blog.cloudflare.com/webmcp/)

::right::

## Herramientas

- **knot.kz**: [www.knot.kz](https://www.knot.kz/)
- **chrome-debug**: [github.com/raultov/chrome-debug-mcp](https://github.com/raultov/chrome-debug-mcp)
- **chrome-devtools-mcp**: [github.com/ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- **Playwright MCP**: [github.com/microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp)
- **media-transcriber**: [github.com/raultov/media-transcriber-mcp](https://github.com/raultov/media-transcriber-mcp)

<!--
[40:15]
-->

---
layout: end
---

# ¡Gracias!

Raúl Tovar

![QR a las slides en GitHub](/images/qr-slides.svg)

**Slides (PDF) y recursos** · github.com/raultov/biznaga-26-web-agentica

<!--
[40:45] Q&A. El QR lleva al repo: arriba del todo, el PDF de las slides y todos los enlaces de la charla.
-->

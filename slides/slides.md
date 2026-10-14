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
layout: two-cols
layoutClass: bz-links bz-coin
---

# Insert coin

## 1 crédito: lo que traes

- **HTML y JavaScript/TypeScript** básicos: formularios, `async`/promesas y JSON
- Haber usado un **asistente de IA** (ChatGPT, Claude, Copilot…)
- Un **navegador**, a nivel de usuario

::right::

## Free play: lo que no necesitas

- Saber **MCP**: lo vemos en la Round 1
- Un **framework** concreto: Angular, React, Vue o JS a secas
- Saber cómo funciona un **LLM** por dentro

<!--
[2:30] 30 segundos. No es un filtro: es para que nadie se asuste con el código que viene.

Si has usado un chatbot y sabes qué es un formulario, tienes crédito suficiente.
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

- Web-MCP es una propuesta de API web, inspirada en MCP, que aún no es estándar
- *Draft* del *W3C Web Machine Learning Community Group* (Microsoft + Google)
- La web publica sus tools **desde el frontend**, con JavaScript o con HTML
- El navegador media cada llamada: la tool corre en la página, **con la sesión del usuario**

> "Instead of your application being a guest within an agent, the agent is a guest on your platform."

<!--
[5:00] Web-MCP copia ideas de MCP, como las tools (nombre, descripción, schema de entrada), pero no el protocolo:
no hay JSON-RPC entre la página y el agente, es una API de JavaScript del navegador.

Microsoft y Google empujan este proyecto. No está en el Standards Track del W3C.
Estado comprobado el 01-10-2026: Draft Community Group Report, última edición 30-09-2026.

El agente puede estar dentro del navegador (Leo en Brave) o fuera (opencode vía chrome-debug),
pero la TOOL siempre se ejecuta **DENTRO de la pestaña**.

**Media**: el agente no habla con la página, se lo pide al navegador, que ejecuta la tool y le devuelve el resultado.
**Con la sesión del usuario**: la tool es JS de la web y usa lo que tenga esa pestaña (cookies, estado, login si lo hay); el agente nunca ve las credenciales.
El agente actúa con los permisos del usuario, no con más.

El agente entra en tu web y juega con tus reglas.
-->

---

# ¿Dónde funciona hoy?

| Navegador / agente | Estado |
| --- | --- |
| Chrome | *Origin trial* desde **149** · flag `#enable-webmcp-testing` |
| Edge | *Origin trial* desde **150** |
| Brave | Experimental en Leo AI · flag `#enable-webmcp-testing` |
| Firefox | Postura **neutral** · sin implementar |
| Safari | **En contra** |
| ChatGPT (escritorio) | **Site tools** en su navegador integrado (agosto 2026) |

<!--
[7:00] NO está activo por defecto en Chrome estable: hace falta origin trial o flag.

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
[8:00] De hecho, las aplicaciones agénticas más eficaces usan MCP y WebMCP a la vez. → knot.kz tiene las dos y en la demo usamos ambas.

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
[9:00]
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
[9:15] 

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
[11:30]

1: toolname + tooldescription, lo que lee el modelo (como el bloque 3 de la imperativa).

[CLICK] 2: los campos; el navegador genera el JSON Schema solo (required → required, `<select>` → enum).

[CLICK] 3: action + botón; no hay execute: ejecutar es enviar el formulario, y lo envía el humano.

[CLICK] todo. Sin registerTool ni abort(): la tool existe mientras el formulario esté en la página.
El JS es opcional: `respondWith()` para devolver un resultado propio sin navegar. knot.kz lo usa.

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
| React | Sin soporte oficial · comunidad: `usewebmcp` |
| Vue / Nuxt | Comunidad: `vue-webmcp`, `nuxt-webmcp` |
| Cualquiera | Polyfill y tipos: `@mcp-b/global`, `webmcp-types` |

> knot.kz: **sin librerías**, un hook de ~70 líneas. Una librería no te ahorra código: te aísla de los **cambios de la spec**.

<!--
[13:30] Angular es el único con soporte **oficial**: desde la v22, en experimental. Puede cambiar incluso en versiones menores.
Tú no tocas `document.modelContext`: si el navegador cambia, lo absorbe Angular.

React y Vue: solo librerías de la **comunidad**.

knot.kz no usa ninguna: la API es una función y una señal de cancelación.
Lo que te da una librería es que **otro** sigue los cambios de la spec por ti.

Con un **polyfill**, tus tools funcionan en cualquier navegador para clientes que ejecuten JavaScript en la página, como Playwright o una extensión.
Pero se pierde la mediación del navegador: ese cliente tiene acceso a toda la página, no solo a tus tools.
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
[15:00] Estas flags son lo que la página **declara** al agente: el navegador las transmite, pero no las hace cumplir.
El navegador controla quién llama a qué y desde dónde (solo HTTPS, tools ligadas a su origen); lo que la tool declara o hace es responsabilidad de la página.
Si algo debe confirmarse, confírmalo en la tool.

consequentialHint está documentado, pero en Chrome 153 aun no llega al agente (knot.kz lo declara en copy-install-command).

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

> Pruébalo: knot.kz → F12 → Application → WebMCP

::right::

![Panel WebMCP de DevTools en knot.kz](/images/devtools-webmcp-panel.png)

<!--
[16:30] Transición a la demo.

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
[18:00] Demo grabada: avisa de que es un vídeo. Tres caminos, uno por slide.
-->

---

# Lo que vamos a ver

Misma pregunta, tres caminos: *¿qué lenguajes soporta knot?*

1. **Leo** en Brave: el agente está **dentro** del navegador
2. **opencode + chrome-debug**: el agente está **fuera**, la tool corre en la pestaña
3. **opencode + knot-site**: MCP clásico, knot.kz ni se abre

<!--
[18:30] Del más integrado al menos integrado. Cierra con la cita: en 1 y 2 el agente es el invitado; en 3, tu web.
-->

---
class: bz-demo-video
---

# Demo 1 · Leo en Brave

<SlidevVideo controls autoreset="slide" playsinline>
  <source src="/videos/brave-leo.mp4" type="video/mp4" />
</SlidevVideo>

<!--
[19:00] Pulsa play (2:44, sin audio). Pausa donde quieras comentar.

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
class: bz-demo-video
---

# Demo 2 · opencode + chrome-debug

<SlidevVideo controls autoreset="slide" playsinline>
  <source src="/videos/opencode-chrome-debug.mp4" type="video/mp4" />
</SlidevVideo>

<!--
[22:00] Pulsa play (2:55, sin audio). Pausa donde quieras comentar.

opencode instruido con PROMPT.md.
open_instance + navigate www.knot.kz → webmcp_list_tools: 6 tools → misma pregunta
→ navigate /#/contact → 7 tools → contact-knot-team rellena el formulario, NO lo envía (Pending hasta que pulso Send).
-->

---
layout: two-cols
layoutClass: bz-wide-right
---

# Demo 3 · opencode + knot-site

MCP clásico: knot.kz ni se abre

```json
"knot-site": {
  "type": "remote",
  "url": "https://www.knot.kz/mcp"
},
```

::right::

<SlidevVideo controls autoreset="slide" playsinline>
  <source src="/videos/knot-site.mp4" type="video/mp4" />
</SlidevVideo>

<!--
[25:15] Pulsa play (1:46, sin audio). Pausa donde quieras comentar.

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
[28:30]
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
[29:00] Habéis visto que las tools funcionan. Ahora viene lo difícil: que tu código esté preparado para tenerlas.
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
[32:30] Un servidor MCP está siempre ahí. Una tool de Web-MCP solo existe mientras alguien tiene tu página abierta, y eso lo cambia todo.

**Sin pestaña, no hay tools**: un agente no puede usar tu web a las 3 de la mañana sin que alguien la tenga abierta. Para eso está MCP.

**Cada ruta, sus tools**: lo visteis en la demo. En la portada, 6; al entrar en contacto aparece contact-knot-team. Las tools siguen a la pantalla.

**Sesión = poder**: la tool hace lo que el usuario podría hacer, ni más ni menos. Sin login, lo de un visitante anónimo.
-->

---

# Costes y abusos

- Cada tool ocupa **contexto** del modelo en cada llamada: las 7 de knot.kz son **~1.000 tokens**
- Presupuestos **recomendados** por Chrome (no los aplica): **30** el nombre, **500** la descripción, **150** cada parámetro, **1.500** la respuesta
- *Rate limiting* y bots: tus tools son una **API documentada**, que se puede usar sin LLM
- Menos tools, mejor descritas: Chrome **no valida** el schema, se hace en el código.

<!--
[33:30] 
las API de los LLM no guardan estado entre llamadas. En cada paso, el agente envía al modelo la conversación entera más la lista de tools disponibles
Una tarea de 3 pasos en knot.kz son unos 3.000 tokens solo en releer el menú. Y al navegar el catálogo cambia, así que la caché ayuda menos.
Aun así suele salir más barato que leer el DOM o hacer capturas: el reto es no inflar el catálogo.

Probado en Chrome 153: acepta un nombre de 60 caracteres y una respuesta de 5.000 sin quejarse. Si te pasas, el problema aparece en el agente, no en el navegador. En knot.kz la descripción más larga usa la mitad del presupuesto; lo que va justo son las respuestas (1.321 de 1.500).

knot.kz lo vigila en CI: scripts/webmcp-budget.mjs falla el build si una tool supera el presupuesto
(objetivo propio: 1.350 caracteres de salida, el 90 % del límite).

Un script con Chrome y CDP puede invocarlas sin modelo. En knot.kz solo contact cuesta dinero: honeypot, validación en servidor y rate limit en Cloudflare.

knot.kz agrupa 5 combinaciones en get-install-command, pero separa copy-install-command porque pide consentimiento.
El schema orienta al modelo; la validación real va en tu código, con errores que digan qué valores valen para que el modelo se corrija.
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
[35:30] Datos sacados del panel WebMCP de DevTools (capturas de respaldo:
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
-->

---

# Dos bridges no caben en una pestaña

- El bridge de Cloudflare registra en la página las tools de tu `/mcp`
- Si tu web ya las registra de forma nativa, los nombres chocan: `InvalidStateError: Duplicate tool name`
- **Gana quien registre primero**: con buena red, tu web; con red lenta, el bridge
- Si gana el bridge, tus tools **pierden las anotaciones**

![Panel WebMCP de Cloudflare: packs Content Credentials (C2PA) y Site MCP server](/images/cloudflare-webmcp-packs.png)

<!--
[37:00] **Cloudflare** (beta desde el 06-08-2026): activas WebMCP en el panel y su CDN añade a cada HTML una línea `<script src="/.webmcp/bridge.js">`. Ese script registra las tools en la página. Tu código no cambia.

Hoy hay dos packs. **Content Credentials**: lee los metadatos C2PA de tus imágenes (autor, herramienta, si es IA) para que el agente responda "¿esta foto es real?". No verifica la firma, y casi ninguna imagen de la web los lleva. **Site MCP server**: convierte las tools de tu `/mcp` en tools WebMCP; cada llamada va a tu servidor con la cookie del usuario, sin OAuth. Todo se ejecuta en el navegador del visitante.

Las 4 tools portables de knot.kz están a la vez en la página y en /mcp.

Probado en producción el 05-10-2026 con el pack activado. 

**Red normal**: ganan mis tools nativas 10 de 10, y el bridge deja 4 errores sin capturar en la consola de cada visitante. 

**Red lenta y sin caché**: gana el bridge 3 de 3, y get-latest-releases pierde su untrustedContent: el bridge solo copia nombre, descripción y schema. La conclusión: bridge O nativo, no los dos.
-->

---
layout: statement
---

# Su valor no es el dato, es el efecto sobre la sesión del usuario

<!--
[38:00] 

WebMCP gana a MCP cuando la tool actúa sobre la sesión: cambia la interfaz que el usuario ve, usa su login sin OAuth, pide consentimiento en pantalla o deja que el humano pulse Send. MCP gana cuando no hay navegador.

Una sola definición por tool, publicada por el canal que toca: knot.kz define sus 4 tools de datos una vez y las sirve en la página y en /mcp. Lo que rompe es duplicarlas en la misma pestaña (slide anterior).

La prueba: si la tool tiene sentido ejecutada desde un servidor sin navegador, es dato (MCP y WebMCP). Si no, es efecto: solo WebMCP.
-->

---
layout: section
round: FINAL BOSS
---

# El futuro

Que te encuentren ya no basta

<!--
[39:30]
-->

---

# Del buscador al agente

- SEO: que los humanos te encuentren
- Agentes: que **te entiendan y te usen**
- Las descripciones de tus tools deciden si el agente **te usa**, como el texto bajo cada resultado de Google influye en si el humano hace clic
- El hueco: hoy un agente **tiene que visitar tu web** para saber qué tools tiene

<!--
[40:00] Limitación oficial: Clientes y navegadores deben visitar un site directamente para saber si tiene tools

El explainer descartó los manifiestos estáticos. Mientras tanto: llms.txt, JSON-LD y un /mcp con instructions (lo que hace knot.kz).

Chrome lo dice en best-practices: la descripción decide qué tool elige el agente, y tiene una guía de Evals para medirlo. Pero ojo: Search Engine Journal y Moz (agosto 2026) coinciden en que WebMCP no es ranking ni indexación; actúa cuando el agente ya está en tu web. Lo que muestra Google se lee antes de visitar la web.

**Ojo**: no es AEO ni GEO (salir citado en las respuestas de la IA). Google dice que eso sigue siendo SEO y trata los agentes aparte.

**Futuro**: ¿Posicionará tener tools? Hoy no: nadie lo usa como señal, y Search Engine Journal no encontró preferencia de los agentes por webs con WebMCP. Pero el precedente es schema.org: no sube el ranking, sí habilita resultados enriquecidos. Un buscador que indexe tools cerraría el hueco.
-->

---

# Game over... o no

- Su valor está en el **efecto sobre la sesión**: si la tool tiene sentido sin navegador, publícala también en MCP
- Web-MCP convierte tu frontend en una **API para agentes**
- Exige arquitectura: casos de uso desacoplados de la UI
- Declara tus flags y **valida en código**: la seguridad es parte del contrato

<!--
[42:00] Cuatro ideas para llevarse a casa.

Apple (WebKit) está en contra: cree que el agente debe usar la web como un humano, con el HTML, sin una capa aparte. La respuesta: WebMCP no sustituye al HTML, añade un contrato que escribe la propia web; donde no hay tools, el agente sigue usando la interfaz.
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
- **Chrome Status**: [chromestatus.com/feature/5117755740913664](https://chromestatus.com/feature/5117755740913664)
- **MCP**: [modelcontextprotocol.io/specification/2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28)
- **Angular**: [angular.dev/ai/webmcp](https://angular.dev/ai/webmcp)
- **Cloudflare**: [blog.cloudflare.com/webmcp](https://blog.cloudflare.com/webmcp/)

::right::

## Herramientas

- **knot.kz**: [www.knot.kz](https://www.knot.kz/)
- **chrome-debug**: [github.com/raultov/chrome-debug-mcp](https://github.com/raultov/chrome-debug-mcp)
- **chrome-devtools-mcp**: [github.com/ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- **Playwright MCP**: [github.com/microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp)
- **Tool Inspector**: [github.com/beaufortfrancois/model-context-tool-inspector](https://github.com/beaufortfrancois/model-context-tool-inspector)

<!--
[42:45]
-->

---
layout: end
---

# ¡Gracias!

Raúl Tovar

![QR a las slides en GitHub](/images/qr-slides.svg)

github.com/raultov/biznaga-26-web-agentica

<!--
[43:00] Q&A. TODO: QR a las slides y redes.
-->

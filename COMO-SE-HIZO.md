# Cómo se hizo

> La historia de **"Construyendo la Web Agéntica: Introducción, Utilidades y Retos de Web-MCP"**
> (BiznagaFest 2026, 7 de noviembre, Auditorio Biznaguito), desde las primeras noticias sobre
> Web-MCP hasta el último ensayo.

Esta charla no salió de un plan cerrado. Salió de ir probando cosas en una web real, de toparme
con problemas que nadie había documentado todavía y de una especificación que siguió cambiando
cuando las transparencias ya estaban terminadas. Y, sobre todo, de ensayar doce veces con un
público muy particular: un agente de IA que escuchaba cada ensayo y me devolvía un análisis.

---

## 1. Mayo de 2026: unas noticias sueltas

Todo empezó con noticias dispersas sobre Web-MCP alrededor de mayo de 2026. Chrome publicó su
documentación de Web-MCP el 18 de mayo, y el primer enlace de mi bibliografía
(`notes.txt`) todavía lleva el rastro de cómo llegué: `utm_source=lighthouse&utm_medium=devtools`.

La idea era sencilla y potente a la vez: en lugar de que un agente "lea" una web a base de
capturas de pantalla y DOM, que **la propia página le diga qué sabe hacer**. La web deja de ser
un sitio que el agente tiene que adivinar y pasa a ser un servidor de herramientas.

El título con el que propuse la charla todavía hablaba de `navigator.modelContext` y de AEO.
Las dos cosas acabarían desapareciendo de las slides (ver la sección 5).

## 2. knot.kz como laboratorio

Para hablar de Web-MCP con propiedad tenía que implementarlo en algo real, no en una demo de
juguete. Tenía a mano **knot.kz**, la web de [knot](https://github.com/raultov/knot), un indexador
de código semántico en Rust para agentes de IA cuya web existía desde mayo.

Era el candidato perfecto:

- Una web real, con tráfico real y un producto real detrás.
- Pequeña, así que podía tocar la arquitectura sin miedo.
- Con casos de uso de los dos tipos: consultar datos (lenguajes soportados, releases, ediciones)
  y actuar sobre la página (copiar el comando de instalación, navegar a una sección, contactar).

El plan de implementación se redactó el 9 de agosto y la versión 0.4.0 del site salió el 15 de
agosto con seis tools imperativas, un formulario de contacto declarativo, un modal de
consentimiento propio, `llms.txt`, JSON-LD y un backend de contacto con honeypot.

Implementarlo fue lo que dio el contenido a la sección "Retos": tuve que sacar la lógica de los
componentes para que la UI y las tools compartieran código, convertir clics en casos de uso y
repetir dentro de las tools las reglas que la UI aplicaba sin decirlas.

El 1 de octubre knot.kz recibió su token de origin trial, y desde entonces sus tools funcionan en
el Chrome normal de cualquier visitante, sin flags.

## 3. El `/mcp` clásico: dónde está la frontera

Con las tools Web-MCP funcionando surgió una pregunta incómoda: **¿y por qué no un servidor MCP
de toda la vida?** Para responderla con datos, el 6 de septiembre añadí a knot.kz un endpoint
`/mcp` clásico, compatible con la especificación MCP del 28 de julio (sin estado) y con la
anterior, y extraje la lógica común de las tools a un núcleo compartido.

El resultado fue el mejor argumento de la charla. De las 7 tools de la página, **solo 4 cruzan
al `/mcp`**, y son exactamente las que únicamente devuelven datos. Las 3 que no cruzan son:

| Tool | API | Por qué no cruza |
| --- | --- | --- |
| `get-install-command` | imperativa | Su efecto es cambiar de pestaña y hacer scroll en la página |
| `copy-install-command` | imperativa | Necesita consentimiento y escribe en el portapapeles del usuario |
| `contact-knot-team` | declarativa | El humano pulsa Enviar |

Dos APIs distintas y una sola causa: **su valor no es el dato, es el efecto sobre la sesión del
usuario**. Esa frontera es la que separa MCP de Web-MCP, y es la demo 3 de la charla.

## 4. C2PA: un hallazgo indagando en Cloudflare

El 6 de agosto Cloudflare anunció su soporte de Web-MCP: un `bridge.js` que su CDN inyecta en tu
web y que registra tools sin que escribas código. Quería saber si era compatible con un site
que, como knot.kz, ya registra sus propias tools.

Al abrir el panel de Web-MCP de DevTools en una web con el bridge activado apareció algo raro.
La tool `inspect_image_c2pa`, que devuelve los metadatos de procedencia de las imágenes, **no
declaraba ninguna anotación**. Ni siquiera `untrustedContentHint`. Y como las anotaciones valen
`false` por defecto, no declarar nada equivale a decirle al agente que el contenido es fiable.

Pero esos metadatos los escribe el autor de la imagen. Basta con poner una instrucción en el
título de una foto para tener un prompt injection servido por una tool "de confianza".

Lo reporté a Cloudflare por HackerOne el 6 de septiembre. Lo cerraron como *Informative*,
"expected behavior", aunque útil "for the ongoing development". La moraleja que cuento en la
charla: es justo ahora, en la fase experimental, cuando estas cosas tienen que salir.

La pregunta inicial también tuvo respuesta. El 5 de octubre probé en producción el pack que
convierte el `/mcp` en tools Web-MCP: con buena red ganan siempre las tools nativas; con red
lenta gana el bridge, y entonces se pierden las anotaciones. De ahí la slide "Dos bridges no
caben en una pestaña" y su conclusión: **bridge o nativo, no los dos**.

## 5. Cuando la spec se mueve bajo tus pies

Las slides se cerraron entre finales de septiembre y mediados de octubre. La especificación no
se paró por eso. Cada cambio obligó a revisar slides, notas o la forma de contarlo:

| Fecha | Cambio | Qué tocó en la charla |
| --- | --- | --- |
| 27-09 | Chrome 153 solo expone `document.modelContext`; `navigator.modelContext` desaparece | El resumen de la charla en la web del evento quedó desfasado |
| 28-09 | Borrador del charter del *Agentic Web Working Group* del W3C, que adoptaría Web-MCP | Nota de la slide 6: ya hay un paso hacia el Standards Track |
| 02-10 | Explainer de *continuations*, con un checkout repartido en varias páginas | Encaja con el ejemplo de checkout de la slide 21 |
| 08-10 | La API declarativa sale del draft (PR #338): Chrome lanzará antes la imperativa | Nota de la slide 11; sigue funcionando en Chrome y en la demo 2 |
| Chrome 155 | `consequentialHint` llega a `getTools()`; los argumentos en JSON como texto pasan a obsoletos | Ejemplo de código de la slide 13 |
| Chrome 156 | Anotación `debugging`; `window.ontool*` deja de funcionar | Notas de las slides 11 y 14 |
| 16-10 | La documentación de Chrome recomienda `use-webmcp-tool`, de Google Chrome Labs, para React | Fila de React de la slide 12 |
| 20-10 | Nuevo draft: los agentes no deben fiarse solo de lo que declara la web | Respaldo de las slides 13 y 24 |
| 27-10 | Contadores de uso de Chrome: el 0,87 % de las cargas de página registran tools, casi todo Shopify, que lo activó en julio | Nueva gráfica de adopción en la slide 6 |

La lección: con una tecnología así, **las slides tienen fecha de caducidad**. Cada dato lleva en
las notas la fecha en que lo comprobé, y la semana de la charla toca volver a revisarlo.

La gráfica merece una línea aparte. Los mismos contadores muestran que por cada ~85.000 cargas
que registran tools, solo una ejecuta alguna: **las webs ya están listas, los agentes todavía
no**. Ese dato acabó siendo uno de los remates de la charla.

---

## 6. Ensayar con un agente como público

Esta es la parte que más me ha sorprendido de todo el proceso.

### El bucle

Grabé cada ensayo con la webcam, de principio a fin, como si fuera la charla real. Después, un
agente de IA (en opencode) hacía esto:

1. **Transcribía el vídeo** con [media-transcriber](https://github.com/raultov/media-transcriber-mcp),
   un servidor MCP mío que usa Whisper en local.
2. **Comparaba la transcripción con las slides y sus notas de ponente**, slide a slide.
3. **Me devolvía un análisis** (`rehearsals/ensayo-N-*-analisis.md`) con:
   - Una tabla de tiempos por slide frente a las marcas `[mm:ss]` de las notas.
   - Lo que había corregido respecto al ensayo anterior.
   - Los errores técnicos, con el minuto exacto, lo que dije y cómo decirlo bien.
   - Las ideas importantes de las notas que nunca llegaba a decir.
4. Yo decidía qué cambiar en las slides, en las notas o en mi forma de contarlo, y volvía a
   ensayar.

Doce ensayos en dos semanas, entre mediados y finales de octubre.

### Lo que salía de cada análisis

**El tiempo.** De 46:41 en el primer ensayo a una charla estable de unos 42 minutos, con margen
para preguntas en una franja de 50:

| Ensayo | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Duración | 46:41 | 43:14 | 43:55 | 39:15 | 42:43 | 42:42 | 41:25 | 41:30 | 41:27 | 44:00 | 41:05 | 42:20 |

Con la media de varios ensayos recalculamos las marcas de tiempo de las notas, de forma que en
el escenario sé en cada slide si voy adelantado o retrasado.

**Los errores que se repetían.** Es lo que nunca habría visto solo. Un error que cometes una vez
es un lapsus; uno que se repite cinco ensayos seguidos es una idea que tienes mal colocada:

| Lo que decía | Lo correcto | Ensayos con el error |
| --- | --- | --- |
| El token del origin trial "vale en cualquier navegador con soporte" | Solo en el navegador donde lo registras: Chrome y Edge tienen registros distintos | 3 a 6 |
| `toolautosubmit` "si no está **o es false**" | Es un atributo booleano: solo "si no lo pones" | 6 a 10 |
| En la demo 2, "se pone en rojo y falla" | La tool no falla: espera a que el humano envíe; lo que vence es la espera del controlador | 4 a 9 |
| "MCP es de solo lectura" | Es el `/mcp` de knot.kz el que solo expone lectura, porque sin navegador no hay página que tocar | 5, 6 y 10 |
| El honeypot "solo lo ven humanos y agentes" | Es al revés: invisible para el humano | 8 y 9 |

**Las decisiones de diseño.** Varios cambios en las slides salieron de los datos de los ensayos:

- Animar por clics las slides donde más tiempo pasaba (6, 23 y 27), para que el público no lea
  por delante de lo que cuento.
- Quitar citas en pantalla que nunca mencionaba.
- Que los vídeos de las demos arrancaran solos a pantalla completa con un clic. Hacerlo a mano
  era torpe: en el ensayo 6 se me oye "voy a poner pantalla completa, a ver qué tal".
- Quitar de las notas frases que nunca decía y añadir las que sí funcionaban.

### Cuando el agente se equivocó

Un análisis automático no es infalible, y conviene contarlo:

- **Ensayo 4:** el agente concluyó que había adelantado el vídeo de la demo 3. No era verdad:
  la transcripción con VAD (detección de voz) comprime los silencios, y durante un vídeo mudo
  "desaparecía" el tiempo. Desde entonces se transcribe sin VAD cuando importan los tiempos.
- **Ensayo 5:** marcó como error mencionar el modelo de Leo en la demo 1, cuando era justo lo
  que decía la nota.
- **Ensayo 7:** contó como tiempo perdido las preguntas que hago al público mientras bebo agua.
  Son a propósito: conectan con la sala.
- **Ensayos 8 y 9:** me atribuyó decir "Chrome DevTools" en lugar de chrome-debug. Era Whisper.
- **Slide 23:** dio por no encontrada la fuente de los presupuestos de caracteres de Google.
  Estaba en la guía de seguridad de Chrome, escrita como "1.5K" en lugar de "1.500".

En todos los casos, mirar el dato original (un fotograma del vídeo, la nota, la página de
Chrome) resolvió la duda. El agente aporta constancia y memoria; el criterio sigue siendo mío.

### La herramienta también mejoró

Usar media-transcriber de verdad, con charlas de 42 minutos, sacó a la luz sus límites, y la
charla acabó marcando parte de su hoja de ruta:

- **Se caía.** El binario de la versión 0.8.0 usaba instrucciones AMX que mi Ryzen 7950X no
  tiene (SIGILL). Hubo que compilar en local y después cambiar cómo se generan las releases.
- **Era lento.** Al principio había que trocear cada ensayo en bloques de 10 minutos con 5 s de
  solape. Entre el 22 y el 24 de octubre se abrieron las issues de GPU, caché del modelo y
  binarios con GPU; desde el ensayo 7, un ensayo entero se transcribe de una pasada.
- **La salida no cabía en el agente.** 42 minutos en SRT eran 122 KB y el cliente los cortaba.
  De ahí salieron el formato `timestamped_text` (issue #8), `output_path` para guardar el
  resultado en disco y `replacements` para corregir nombres propios como "Not.cafeta" → knot.kz
  (issue #9).
- **Partía el texto palabra a palabra** a partir de cierto minuto. El ensayo 7 dio los datos para
  la issue #10, que se arregló en la versión 0.12.1.

Es decir: preparar una charla sobre agentes acabó mejorando una herramienta para agentes.

### Lo que el agente no sustituye

Los ensayos con compañeros aportaron cosas que ninguna transcripción podía detectar. Por
ejemplo, que se quedaban con la idea de que knot.kz era una web hecha solo para esta charla.
La slide 2 se rehízo para presentar primero el producto y después su uso como laboratorio.
Y de otro comentario salió subir el tamaño del texto de todas las slides.

El agente mide lo que digo. Las personas cuentan lo que entienden.

---

## 7. Cronología

| Fecha | Hito |
| --- | --- |
| Mayo 2026 | Primeras noticias sobre Web-MCP; Chrome publica su documentación (18-05) |
| 06-07 | Shopify empieza a registrar tools en sus tiendas (los contadores de Chrome saltan) |
| 06-08 | Cloudflare anuncia su bridge de Web-MCP |
| 09-08 | Plan de implementación en knot.kz |
| 15-08 | knot.kz v0.4.0: seis tools imperativas, formulario declarativo y consentimiento propio |
| 05-09 | Presupuestos de caracteres comprobados en CI |
| 06-09 | Endpoint `/mcp` clásico en knot.kz; reporte del caso C2PA a Cloudflare |
| 28-09 | Primer commit de las slides |
| 01-10 | Token de origin trial en knot.kz |
| 05-10 | Prueba en producción del bridge de Cloudflare frente a las tools nativas |
| 08-10 | La API declarativa sale del draft |
| Mediados de octubre | Slides cerradas; primeros ensayos |
| 17-10 → 31-10 | Ensayos 2 a 12, analizados con media-transcriber |
| 07-11 | La charla |

## Herramientas

- [Slidev](https://sli.dev) para las slides.
- [opencode](https://opencode.ai) como agente para preparar contenido, revisar y analizar ensayos.
- [chrome-debug](https://github.com/raultov/chrome-debug-mcp) para controlar el navegador en la demo 2.
- [media-transcriber](https://github.com/raultov/media-transcriber-mcp) para transcribir los ensayos
  y capturar fotogramas de los vídeos.
- [knot](https://github.com/raultov/knot) para explorar código con el agente.

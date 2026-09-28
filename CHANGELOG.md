# Changelog

Todas las fechas notables de este repositorio se documentan en este archivo.

El formato se basa en [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/) y este proyecto adherido a [Versionado Semántico](https://semver.org/lang/es/).

> **Nota sobre la historia previa a git:** el primer commit de este repositorio es del 2026-09-28; antes el proyecto existía solo como carpeta local. Las fases anteriores se reconstruyen abajo a partir de las marcas de tiempo de los ficheros, agrupadas en la versión inicial.

## [0.1.0] - 2026-09-28

Primera versión versionada del material de la charla *Construyendo la Web Agéntica: Introducción, Utilidades y Retos de Web-MCP* (BiznagaFest 2026).

### Añadido

- **README.md** — explicación del propósito del repositorio, guía de instalación y uso de la presentación y resumen de la demo WebMCP.
- **CHANGELOG.md** — este archivo.
- **Repositorio git privado** en GitHub (`raultov/biznaga-26-web-agentica`) con `.gitignore` que excluye `node_modules/`, `slides/dist/`, `slides/dist-export/` y ficheros de entorno.
- **`slides/`** — presentación completa en [Slidev](https://sli.dev) v53: 37 diapositivas, aspecto 16/9, tema *arcade* BiznagaFest (`styles/arcade.css`, paleta `#000046` / `#E4E3FF` / `#EAEA00 → #DE1223`, fuentes empaquetadas en local: Press Start 2P, Rajdhani, Inter, JetBrains Mono), layouts propios `cover` / `section` / `end`, pie de página `STAGE NN/TT` y notas de ponente en cada diapositiva. Estructura de la charla: Introducción → Utilidades → Demo → Retos → El futuro.
- **`slides/README.md`** — requisitos, comandos (`dev`, `build`, `preview`, `export:*`), reglas de portabilidad y guía de migración a PowerPoint / Google Slides.
- **`PROMPT.md`** — guía y prompt para que un agente LLM descubra y ejecute las tools WebMCP de un sitio con el servidor MCP `chrome-debug` (instancia con preset `WEB_MCP`, `webmcp_list_tools` → `webmcp_invoke_tool`, consentimiento humano con `webmcp_get_invocation`, cierre con `close_instance`). Sitio de referencia: knot.kz (6 tools imperativas + `contact-knot-team` declarativa).
- **`notes.txt`** — bibliografía de la charla (Chrome WebMCP docs, webmachinelearning/webmcp, post de Cloudflare), guion de la demo con opencode + `knot-local` y las notas de argumentación del cierre (las 3 tools que no cruzan a `/mcp`, campo `instructions` de `DiscoverResult` como alternativa de coste cero).
- **`images/security/`** — capturas del caso de seguridad C2PA (flags `readOnly` / `untrustedContent` ausentes en `inspect_image_c2pa` del bridge de Cloudflare) usadas en la diapositiva *Seguridad: el caso C2PA*.
- Scripts de exportación a PDF/PPTX/PNG vía Chrome del sistema (`--executable-path`), con nombre de export `webmcp-biznaga-2026`. Los artefactos exportados (`slides/dist-export/`) se generan localmente y **no** se versionan.

### Actividad pre-git (reconstruida)

- **2026-09-06** — notas de investigación y bibliografía (`notes.txt`) y capturas del caso C2PA (`images/security/`).
- **2026-09-27** — andamiaje de la presentación (layouts, tema arcade, `global-bottom.vue`, `package.json`) y redacción completa de `slides.md`; primera exportación a PDF/PPTX; redacción de `PROMPT.md` y del README interno de `slides/`.
- **2026-09-28** — creación del repositorio git, primer commit y publicación como repo privado en GitHub.

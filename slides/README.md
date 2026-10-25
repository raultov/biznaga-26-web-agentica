# Construyendo la Web Agéntica — BiznagaFest 2026

Slides de la charla de Raúl Tovar (7 nov 2026, 12:00, Auditorio Biznaguito), hechas con [Slidev](https://sli.dev).

## Requisitos

- Node **>= 22.12** (probado con 24.21 vía `nvm install 24`)
- Google Chrome del sistema (se usa para exportar; no se descarga Chromium de Playwright)

```bash
npm install          # con red, una sola vez
npm run dev          # http://localhost:3030  ·  presentador: /presenter
npm run build        # SPA estática en dist/ (funciona sin red)
npm run preview      # sirve dist/ en local
npm run export       # PDF + PPTX en dist-export/
```

Las fuentes (Press Start 2P, Rajdhani, Inter, JetBrains Mono) y el logo están empaquetados en local: no hay dependencias de red durante la charla.

## Estructura

| Ruta | Contenido |
| --- | --- |
| `slides.md` | **Todo el contenido**. Markdown plano + notas de ponente en `<!-- -->` |
| `styles/arcade.css` | Tokens y estilo del tema arcade BiznagaFest |
| `layouts/` | `cover`, `section` (ROUND N), `end` (CONTINUE?) |
| `global-bottom.vue` | Pie `STAGE NN/TT` |
| `public/images/` | Imágenes usadas en las slides |

## Reglas de portabilidad

Para poder migrar a Google Slides / PowerPoint si lo exigen:

1. En `slides.md` solo Markdown estándar: títulos, listas, código, tablas, imágenes. Nada de HTML ni componentes Vue en el contenido.
2. El layout se elige solo con `layout:` en el frontmatter, y cada uno tiene equivalente en un patrón de PowerPoint:

   | Slidev | Patrón PowerPoint / Google Slides |
   | --- | --- |
   | `cover` | Diapositiva de título |
   | `section` | Encabezado de sección |
   | `default` | Título y contenido |
   | `two-cols` | Dos contenidos |
   | `statement` | Solo título (centrado) |
   | `end` | Diapositiva de cierre |

3. Las notas de ponente van en el último comentario `<!-- -->` de cada slide: se exportan como notas del PPTX.
4. Los efectos visuales (degradados, parpadeo) están solo en CSS y se congelan al exportar.

## Migración

- **Rápida (5 min):** `npm run export:pptx` genera un PPTX con cada slide como imagen + notas de ponente. Se abre en PowerPoint, LibreOffice Impress o Google Slides (*Archivo → Importar diapositivas*). El texto **no** es editable.
- **Editable:** crear un patrón en PowerPoint/Slides con los tokens de `styles/arcade.css` (colores `#040A25`, `#000046`, `#E4E3FF`, `#599FFF`, `#EAEA00 → #DE1223`; fuentes Rajdhani/Inter/Press Start 2P) y volcar `slides.md` slide a slide según la tabla de layouts.

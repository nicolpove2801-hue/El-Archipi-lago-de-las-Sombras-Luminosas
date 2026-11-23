# El Archipiélago de las Sombras Luminosas

"El Archipiélago de las Sombras Luminosas" es una experiencia educativa interactiva en español: una narrativa marina bifurcada con 5 pestañas (Historia, Bestiario, Bitácora, Laboratorio de dibujo en canvas y Guía Docente), partículas animadas de fondo, audio sintetizado con la Web Audio API e insignias pedagógicas.

Es una página estática de un solo archivo (`index.html`) construida con HTML, Tailwind (vía CDN), FontAwesome 6.4.0 Free, Google Fonts y JavaScript vanilla embebido. No requiere instalación ni build: basta con abrirla en el navegador.

## Demo en vivo

Puedes ver el proyecto publicado en GitHub Pages aquí: [El Archipiélago de las Sombras Luminosas (demo en vivo)](https://nicolpove2801-hue.github.io/El-Archipi-lago-de-las-Sombras-Luminosas/)

## Correcciones de bugs

1. **Iconos de FontAwesome inválidos en FA6 Free** (salían como cuadro vacío): se reemplazaron `fa-water-waves` → `fa-water`, `fa-journal-whills` → `fa-book-open` y `fa-sparkles` → `fa-wand-sparkles`.
2. **Toasts sin color**: el modo JIT del CDN de Tailwind no genera clases interpoladas en tiempo de ejecución; se añadió un diccionario `TOAST_COLORS` con nombres de clase literales completos.
3. **Trazo entrecortado en el canvas del Laboratorio**: se reestructuró la lógica del path (`moveTo` en `startDraw`, `lineTo` + `stroke` en `draw`) para lograr un trazo fluido.
4. **Scroll al dibujar en móvil**: los listeners táctiles pasaron a `{passive:false}` con `e.preventDefault()`.
5. **La bitácora no persistía**: `saveLogbook()` ahora guarda los `textarea`, el historial y las insignias en `localStorage`, y se restauran al cargar la página.
6. **Clase `animate-spin-slow` indefinida**: se añadieron los keyframes correspondientes para que la brújula gire.
7. **El audio del pincel saturaba el `AudioContext`**: la frecuencia se acotó (~200–600 Hz) y la llamada se estranguló (~70 ms).

> **Nota de seguridad**: el mensaje del toast, los `textarea` restaurados y el historial de decisiones se insertan vía `textContent` / `.value` en lugar de `innerHTML`.

## Mejoras de realismo (ambiente submarino)

- Partículas con deriva senoidal (corriente submarina) y velocidades variadas.
- Rayos de luz / cáusticas diagonales suaves detrás de las partículas.
- Burbujas ascendentes con bamboleo.
- Sonido ambiental más creíble: ruido filtrado paso-bajo (murmullo de agua) + dron grave modulado por un LFO, al activar el audio.
- Bestiario con descripciones biológicamente más plausibles (reacción química, simbiosis bacteriana, filtración de plancton, contrailuminación), manteniendo el tono educativo en español.

## Cómo verlo / ejecutarlo

Tienes dos opciones:

- **Abrir el archivo directamente**: descarga el repositorio y abre `index.html` en cualquier navegador moderno. No hace falta instalación, servidor ni build.
- **Visitar la demo en vivo**: entra en el enlace de GitHub Pages de arriba.

GitHub Pages se sirve desde la rama principal una vez fusionados los cambios.

# El Archipiélago de las Sombras Luminosas

"El Archipiélago de las Sombras Luminosas" es una experiencia educativa interactiva en español: una narrativa marina bifurcada con 6 pestañas (Historia, Bestiario, Bitácora, Laboratorio de dibujo en canvas, Guía Docente y Multimedia), partículas animadas de fondo, audio sintetizado con la Web Audio API e insignias pedagógicas.

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

## Multimedia

Se añadió una sexta pestaña **"Multimedia"** que reúne contenido audiovisual sobre el mismo tema del proyecto: el océano y la bioluminiscencia.

- **Video**: la charla TED de Edith Widder, *"The weird, wonderful world of bioluminescence"*, incrustada desde YouTube (ID `jCL0mhCnWDo`) dentro de un contenedor responsivo con relación de aspecto 16:9.
- **Pódcast**: un programa de divulgación científica incrustado mediante el reproductor de Spotify (show ID `2rhQIcCsDr0NNbWJM0ESVP`).

> **Nota**: ambos son recursos sugeridos y se pueden reemplazar fácilmente editando el atributo `src` del `iframe` correspondiente en `index.html`. Junto a cada `iframe` hay comentarios HTML que explican cómo hacerlo.

## Cómo verlo / ejecutarlo

Tienes dos opciones:

- **Abrir el archivo directamente**: descarga el repositorio y abre `index.html` en cualquier navegador moderno. No hace falta instalación, servidor ni build.
- **Visitar la demo en vivo**: entra en el enlace de GitHub Pages de arriba.

GitHub Pages se sirve desde la rama principal una vez fusionados los cambios.
/* arreglos.js – Correcciones para "El Archipiélago de las Sombras Luminosas".
   Se carga con UNA línea antes de </body> en index.html:
   <script src="arreglos.js"></script>                                              */
(function () {
  const $ = id => document.getElementById(id);
  const shuffle = a => a.map(v => [Math.random(), v]).sort((x, y) => x[0] - y[0]).map(x => x[1]);

  /* ---------- CORRECCIONES ---------- */
  // 1) Safari/iPad: user-select:none en body impedía escribir en la Bitácora. Y el menú desborda en móvil.
  const st = document.createElement('style');
  st.textContent = 'textarea,input{-webkit-user-select:text;user-select:text}nav{max-width:100%;overflow-x:auto}';
  document.head.appendChild(st);

  // 2) Quitar la nota para desarrolladores que veían los jugadores en Multimedia.
  const note = [...document.querySelectorAll('#view-media > div')]
    .find(d => d.textContent.includes('Recursos sugeridos'));
  if (note) note.remove();

  // 3) Existía "Final A" y "Final C", pero no "Final B".
  if (typeof storyNodes !== 'undefined' && storyNodes.node_ending_c)
    storyNodes.node_ending_c.title = 'Final B: La Lección de las Corrientes';

  // 4) El dibujo del Laboratorio se borraba al cambiar de pestaña.
  const _rc = resizeCanvas;
  resizeCanvas = function () {
    if (!canvas || !canvas.parentElement) return;
    const w = canvas.parentElement.clientWidth, h = canvas.parentElement.clientHeight;
    if (w && h && canvas.width === w && canvas.height === h) return;
    _rc();
  };
  // 5) Si el cursor sale del lienzo con el clic apretado, el trazo seguía "pegado".
  $('creature-canvas').addEventListener('mouseleave', () => stopDraw());

  // 6) iOS/Chrome dejan el audio en "suspended" hasta reanudarlo.
  const _pse = playSoundEffect;
  playSoundEffect = function (...a) {
    if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
    _pse(...a);
  };

  // 7) "Insignias Desbloqueadas" de la Bitácora nunca se actualizaba.
  const logged = new Set();
  function addBadgeLog(n) {
    if (logged.has(n)) return;
    const l = $('logbook-badges-list');
    if (!logged.size) l.innerHTML = '';
    logged.add(n);
    const p = document.createElement('p');
    p.className = 'text-xs text-neon-gold flex items-center gap-2';
    p.innerHTML = '<i class="fa-solid fa-award"></i>';
    const s = document.createElement('span');
    s.textContent = n;
    p.appendChild(s);
    l.appendChild(p);
  }
  const _ub = unlockBadgeUI;
  unlockBadgeUI = function (t, s) {
    _ub(t, s);
    addBadgeLog(t === 'empathy' ? 'Mirada Profunda' : 'Equilibrio Natural');
  };

  /* 8) "Liberar al Mar": la criatura dibujada aparece nadando en el mar */
  function recortar() {
    const w = canvas.width, h = canvas.height;
    const src = ctx.getImageData(0, 0, w, h), d = src.data;
    let x0 = w, y0 = h, x1 = 0, y1 = 0;
    for (let y = 0; y < h; y++) for (let x = 0; x < w; x++) {
      const i = (y * w + x) * 4;
      // El fondo del lienzo es #061024: lo volvemos transparente y dejamos el brillo.
      const a = Math.min(255, (Math.abs(d[i] - 6) + Math.abs(d[i + 1] - 16) + Math.abs(d[i + 2] - 36)) * 3);
      d[i + 3] = a;
      if (a > 40) { if (x < x0) x0 = x; if (x > x1) x1 = x; if (y < y0) y0 = y; if (y > y1) y1 = y; }
    }
    if (x1 < x0) return null;
    const tmp = document.createElement('canvas');
    tmp.width = w; tmp.height = h;
    tmp.getContext('2d').putImageData(src, 0, 0);
    const cw = x1 - x0 + 1, ch = y1 - y0 + 1;
    const out = document.createElement('canvas');
    out.width = cw; out.height = ch;
    out.getContext('2d').drawImage(tmp, x0, y0, cw, ch, 0, 0, cw, ch);
    return out;
  }

  spawnCreatureFromCanvas = function () {
    if (!ctx) return;
    const img = recortar();
    if (!img) { showToast('¡Dibuja tu criatura primero!', 'neon-magenta'); return; }
    playChime();

    const ov = document.createElement('div');
    ov.style.cssText = 'position:fixed;inset:0;z-index:60;background:#02050e';
    const cv = document.createElement('canvas');
    cv.style.cssText = 'width:100%;height:100%;display:block';
    ov.appendChild(cv);
    const ui = document.createElement('div');
    ui.style.cssText = 'position:absolute;top:16px;left:0;right:0;display:flex;flex-direction:column;align-items:center;gap:10px;pointer-events:none;text-align:center;padding:0 12px';
    const tit = document.createElement('div');
    tit.textContent = '¡Tu criatura fue liberada al mar!';
    tit.style.cssText = 'font:700 22px Fredoka,sans-serif;color:#00f5d4;text-shadow:0 0 14px #00f5d4';
    const cerrar = document.createElement('button');
    cerrar.textContent = 'Volver al laboratorio';
    cerrar.style.cssText = 'pointer-events:auto;padding:10px 20px;border-radius:14px;border:0;font:700 14px Quicksand,sans-serif;background:linear-gradient(90deg,#00f5d4,#4cc9f0);color:#02050e;cursor:pointer';
    ui.appendChild(tit); ui.appendChild(cerrar);
    ov.appendChild(ui);
    document.body.appendChild(ov);

    const c = cv.getContext('2d');
    let W = 0, H = 0, tw = 0, th = 0;
    const fit = () => {
      W = cv.width = innerWidth; H = cv.height = innerHeight;
      tw = Math.min(220, W * 0.3);
      th = tw * img.height / img.width;
      if (th > H * 0.3) { th = H * 0.3; tw = th * img.width / img.height; }
    };
    fit();
    addEventListener('resize', fit);

    const bubbles = Array.from({ length: 35 }, () => ({ x: Math.random(), y: Math.random(), r: 2 + Math.random() * 5, v: 0.0004 + Math.random() * 0.0008, p: Math.random() * 6 }));
    const glow = Array.from({ length: 40 }, () => ({ x: Math.random(), y: Math.random(), r: 1 + Math.random() * 2.5, p: Math.random() * 6, c: ['#00f5d4', '#f72585', '#ffd166', '#4cc9f0'][Math.floor(Math.random() * 4)] }));
    const peces = [{ x: 0.5, y: 0.5, s: 1, dir: 1, v: 1.3, p: 0, main: true }];
    for (let i = 0; i < 3; i++) peces.push({ x: Math.random(), y: 0.2 + Math.random() * 0.55, s: 0.3 + Math.random() * 0.15, dir: Math.random() < 0.5 ? -1 : 1, v: 0.8 + Math.random() * 0.8, p: Math.random() * 6, main: false });

    const t0 = performance.now();
    let raf;
    function frame(now) {
      const t = now - t0;
      const g = c.createLinearGradient(0, 0, 0, H);
      g.addColorStop(0, '#15325e'); g.addColorStop(0.55, '#061024'); g.addColorStop(1, '#02050e');
      c.fillStyle = g; c.fillRect(0, 0, W, H);

      // Rayos de luz
      c.save(); c.globalCompositeOperation = 'lighter';
      for (let i = 0; i < 3; i++) {
        const cx = (0.2 + i * 0.3 + Math.sin(t * 0.0002 + i) * 0.08) * W;
        const rg = c.createLinearGradient(cx, 0, cx + 120, H);
        rg.addColorStop(0, 'rgba(76,201,240,0.10)'); rg.addColorStop(1, 'rgba(76,201,240,0)');
        c.fillStyle = rg;
        c.beginPath(); c.moveTo(cx - 40, 0); c.lineTo(cx + 40, 0); c.lineTo(cx + 220, H); c.lineTo(cx + 20, H); c.fill();
      }
      c.restore();

      // Plancton luminoso y burbujas
      glow.forEach(o => {
        c.globalAlpha = 0.3 + 0.5 * Math.abs(Math.sin(t * 0.001 + o.p));
        c.fillStyle = o.c; c.shadowBlur = 10; c.shadowColor = o.c;
        c.beginPath(); c.arc(o.x * W, o.y * H, o.r, 0, 7); c.fill();
      });
      c.shadowBlur = 0; c.globalAlpha = 0.35; c.strokeStyle = '#bff0ff'; c.lineWidth = 1;
      bubbles.forEach(b => {
        b.y -= b.v * 16; if (b.y < -0.05) { b.y = 1.05; b.x = Math.random(); }
        c.beginPath(); c.arc(b.x * W + Math.sin(t * 0.002 + b.p) * 8, b.y * H, b.r, 0, 7); c.stroke();
      });
      c.globalAlpha = 1;

      // Fondo marino
      c.fillStyle = '#040a18';
      c.beginPath(); c.moveTo(0, H);
      for (let x = 0; x <= W; x += 40) c.lineTo(x, H - 30 - Math.sin(x * 0.01 + 1) * 14);
      c.lineTo(W, H); c.fill();

      // Criaturas
      peces.forEach(f => {
        const w = tw * f.s, h = th * f.s;
        f.x += f.dir * f.v / W;
        if (f.x > 1 - w / (2 * W)) f.dir = -1;
        if (f.x < w / (2 * W)) f.dir = 1;
        let sc = 1, al = 1;
        if (f.main) {
          const p = Math.min(1, t / 1600);
          sc = 1 + (1 - p) * 0.9; al = p;
          if (p < 1) { // destello de liberación
            c.strokeStyle = 'rgba(0,245,212,' + (1 - p) + ')'; c.lineWidth = 3;
            c.beginPath(); c.arc(f.x * W, 0.5 * H, p * 220, 0, 7); c.stroke();
          }
        }
        const y = (f.main ? 0.5 : f.y) * H + Math.sin(t * 0.002 + f.p) * H * 0.04;
        c.save();
        c.globalAlpha = al;
        c.translate(f.x * W, y);
        c.rotate(Math.sin(t * 0.003 + f.p) * 0.08);
        c.scale(f.dir * sc, sc);
        c.shadowBlur = 25; c.shadowColor = '#00f5d4';
        c.drawImage(img, -w / 2, -h / 2, w, h);
        c.restore();
      });
      raf = requestAnimationFrame(frame);
    }
    raf = requestAnimationFrame(frame);

    function cerrarEscena() {
      cancelAnimationFrame(raf);
      removeEventListener('resize', fit);
      removeEventListener('keydown', onKey);
      ov.remove();
      clearCanvas();
    }
    function onKey(e) { if (e.key === 'Escape') cerrarEscena(); }
    addEventListener('keydown', onKey);
    cerrar.onclick = cerrarEscena;
  };

  /* Enlace a la página de puzles en el menú */
  const a = document.createElement('a');
  a.href = 'puzles.html';
  a.className = 'tab-btn px-4 py-2 rounded-xl text-xs sm:text-sm font-bold text-slate-300 hover:text-white hover:bg-ocean-800/60 transition-all duration-300 flex items-center gap-2';
  a.innerHTML = '<i class="fa-solid fa-puzzle-piece text-neon-gold"></i> <span>Puzles</span>';
  $('btn-tab-media').after(a);
})();

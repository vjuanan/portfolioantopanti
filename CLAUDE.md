# CLAUDE.md — Portfolio de Antonella Barone

Guía para cualquier agente que trabaje en este repo. Leerla completa antes de tocar código.

## Qué es

Portfolio personal de **Antonella Barone** (investigación social, gestión de proyectos y UX).
Sitio de una sola página, en español (`lang="es"`), público y en producción.

- **En vivo:** https://portfolioantopanti.vercel.app
- **Repo:** https://github.com/vjuanan/portfolioantopanti (público, rama por defecto `main`)
- **Deploy:** Vercel, proyecto `juanans-projects-d939887f/portfolioantopanti`, conectado por
  integración de GitHub. **Todo push a `main` publica automáticamente en producción.**
  No hay comando de build ni variables de entorno: es un sitio estático puro.

## Stack y estructura

HTML, CSS y JavaScript vanilla. **Sin build step, sin dependencias, sin `package.json`.**

```
.
├── index.html      # Página completa: markup + <style> del portfolio + <script> de interacciones
├── styles.css      # Sistema de diseño base "Organic": tokens, fuentes y componentes
├── assets/
│   ├── anto-avatar.jpg    # Foto circular del hero
│   └── anto-portrait.jpg  # Retrato 4:5 (sección "Sobre mí" + imagen de Open Graph, 900×1125)
├── vercel.json     # cleanUrls + cache inmutable de un año para /assets/*
├── CLAUDE.md       # Este archivo
└── README.md
```

**Orden de cascada (importante):** `styles.css` se enlaza *primero* y define los tokens del
sistema base (paleta crema + terracota + oliva). El `<style>` embebido en `index.html` se
carga *después* y **sobrescribe** `:root` con la variante del portfolio: crema + tinta con
rosa/malva de acento. Si un color no cambia cuando lo tocás, revisá cuál de los dos manda.

### Desarrollo local

```bash
python3 -m http.server 8000
```

Y abrir http://localhost:8000. Cualquier servidor estático sirve (`npx serve .` también).
Abrir `index.html` con `file://` funciona a medias — las fuentes de Google y algunos
comportamientos se ven raros. Usá el servidor.

## Sistema de diseño

- **Paleta:** fondo crema `#f7f0ea`, tinta `#231c1e`, acento rosa `#e8437d`, acento
  secundario malva `#a94fa0`. Las rampas (`--color-accent-100` … `-900` y las de malva y
  neutros) están generadas en OKLCH sobre una misma escala de luminosidad: el paso `-400`
  de rosa pesa visualmente igual que el `-400` de malva. **Si agregás un color, agregalo
  como rampa completa respetando esa escala**, no como hex suelto.
- **Tipografía:** `Caprasimo` para títulos, `Figtree` para texto. Se importan desde Google
  Fonts en la primera línea de `styles.css`.
- **Curvas de animación:** `--spring` (rebote corto), `--soft` (salida suave),
  `--drop` (la caída con rebote al acomodarse — es la firma del sitio).
- **`--edge`:** margen lateral fluido, `clamp(20px, 5vw, 80px)`. Usalo en vez de paddings fijos.

## Sistema de animación de "caída"

Es la identidad visual del sitio: **todo entra desde arriba, apenas rotado, y se acomoda con
un rebote.** No lo reemplaces por fades genéricos.

- `[data-reveal]` — un elemento que cae. Se configura con variables inline:
  `--fall` (distancia), `--tilt` (rotación de entrada), `--rest` (rotación final), `--d` (delay).
- `[data-cascade]` — un contenedor cuyos hijos caen escalonados; el JS le asigna a cada hijo
  un `--d` incremental de 0.09 s y alterna el `--tilt` en los pares.
- `[data-watch]` — solo marca una sección para que el observer le ponga la clase `.in`.
- Un único `IntersectionObserver` (threshold 0.12) agrega `.in` y **deja de observar**
  (`unobserve`): las animaciones ocurren una sola vez, no se repiten al volver a scrollear.
- `.mask > span` — títulos que suben desde detrás de una máscara al entrar en viewport.
- `.name .ch` — el nombre del hero se parte letra por letra desde JS (`.split-letters`) y cada
  letra cae con `--i * 0.05s` de retraso. "Barone" va en `outline` (contorno) y se rellena de
  rosa al hover en desktop o al entrar en viewport en touch (`.name.lit`).

### Decisión deliberada: `prefers-reduced-motion` se ignora

En el `<script>` hay `const reduce = false;` con todas las ramas de reducción cableadas pero
inactivas. **Es intencional y fue pedido explícitamente** (commit `f463de1`): el portfolio
tiene que lucir todos los efectos aunque la máquina de quien lo mira tenga activado "reducir
movimiento", porque es una pieza de presentación. **No lo "arregles" por accesibilidad sin
preguntar.** Si algún día se revierte, alcanza con volver a
`const reduce = matchMedia('(prefers-reduced-motion: reduce)').matches;` — el resto del código
ya respeta esa bandera.

## Capa táctil (mobile) — lección aprendida

En desktop las tarjetas de proyecto se expanden con `:hover`. En touch no hay hover, así que:

- `document.documentElement` lleva la clase `is-touch` cuando `(hover: none)` o `(pointer: coarse)`.
- La tarjeta más centrada en pantalla se auto-realza con la clase `.focus` mientras se scrollea.
- **El bug que ya se corrigió (commit `1281ad8`): parpadeo/bucle abrir-cerrar.** Al expandirse,
  una tarjeta cambia de alto y mueve la geometría, lo que hacía que el cálculo de "cuál está
  más centrada" saltara entre dos tarjetas en loop. La solución vigente en `focusNearest()`:
  1. se mide la distancia al **header** de la tarjeta (`.pc-top`), que no se mueve al expandirse,
     no al elemento completo; y
  2. hay **histéresis** — una tarjeta ya enfocada se mantiene hasta que otra esté claramente más
     centrada (`bd < cd - innerHeight * 0.14`), y solo se adquiere el foco si está bien centrada.
  Si tocás esa función, **probá en mobile real o en el emulador con scroll lento**, es donde
  reaparece el problema.
- Además del foco por scroll, tap o Enter fijan la tarjeta abierta con la clase `.open`
  (las tarjetas tienen `tabindex="0"`, así que el teclado también funciona).

## Bucle de scroll (`requestAnimationFrame`)

Un solo `loop()` continuo maneja cuatro cosas — si agregás efectos de scroll, sumalos ahí en
vez de crear otro listener:

1. **Marquee** (`.marq-track`): se mueve solo a velocidad base y el scroll lo empuja más rápido.
2. **Parallax**: cualquier elemento con `data-par="0.05"` se desplaza según su posición. Usa la
   propiedad CSS `translate` (no `transform`) **a propósito**, para no pisar los `transform` de
   los estados hover.
3. **Círculos del bloque de contacto**: derivan vía la variable `--pc`.
4. **Inercia**: la pila de proyectos (`#plist`) se ladea con `skewY` según la velocidad del
   scroll, con amortiguación y tope de ±2.2°.

El handler de `scroll` está separado, con `passive: true` y throttle por `rAF`; maneja la barra
de progreso, la clase `.scrolled` del nav y el foco táctil.

## Contenido

Los datos del sitio son **reales, de su CV**. No inventes proyectos, cifras ni credenciales.

- Cuatro tarjetas de proyecto: web internacional de HS Brands Latam (única con link externo,
  https://oportunidadeshsbrands.com), cuestionarios en el CRM interno, investigación de
  usuarios, y diseño instruccional en Efe Idiomas (2018–2025).
- Estadísticas animadas vía `data-count` (7+ años, 1 web lanzada, 2 idiomas, 2026 socióloga UBA).
- Formación: Sociología UBA 2022–2026, Lovable L1, Python (Agencia de Habilidades para el
  Futuro, 2024), inglés B2 (FCE).
- Contacto: antonella.m.barone@gmail.com · LinkedIn `antonella-barone2003` · Vicente López, BA.

## Meta tags y compartir

Las URLs de `og:image` / `twitter:image` **tienen que ser absolutas**
(`https://portfolioantopanti.vercel.app/assets/...`), con `og:image:width` y `og:image:height`
declarados. Con rutas relativas WhatsApp/LinkedIn no levantan la portada — ya pasó y se corrigió
en el commit `4901db0`. Si cambiás la foto del retrato, actualizá también esas dimensiones.

El favicon es un SVG embebido como data-URI en el `<link rel="icon">`: una "A" con un punto rosa.

## Convenciones de trabajo

- **Idioma:** todo en español — código comentado, commits, README y contenido.
- **Commits:** mensajes en español, en presente, describiendo el efecto visible para quien mira
  el sitio (mirá `git log` para el tono). **Sin tildes ni ñ en el asunto del commit** — la
  consola de Windows los rompe. En los archivos sí van con tildes normales.
- **Estilo de código:** el CSS del portfolio vive embebido en `index.html`, agrupado por
  secciones con comentarios `/* — nombre — */`. Seguí ese orden y ese tono de comentario:
  explican *por qué*, no *qué*.
- **Antes de dar algo por terminado, miralo en el navegador** — desktop y mobile. El sitio es
  puro efecto visual; los cambios no se validan leyendo el diff.

## Archivos locales que no están en el repo

En la carpeta de trabajo original hay material personal que está fuera del control de versiones
(y así debe quedar): `Diploma_TT.pdf`, `StatementOfResult.pdf`, `assets/anto.jpeg` y la carpeta
`capturas-en-vivo/` con capturas de referencia del sitio. Están en `.gitignore`. Si hacen falta
para alguna sección (por ejemplo, publicar el diploma), hay que pedirlos aparte.

## Pendientes e ideas

Ninguna es urgente; el sitio está terminado y en producción. Son las líneas abiertas:

- [ ] Las tarjetas 02, 03 y 04 no tienen link externo — solo la 01. Si aparecen materiales
      (capturas, documentos, casos), sumarles enlace o galería.
- [ ] No hay CV descargable en PDF. Sería el complemento natural del bloque de contacto.
- [ ] Dominio propio en vez del subdominio `.vercel.app` (se configura en Vercel + DNS).
- [ ] Sin analítica. Vercel Analytics se activa desde el panel del proyecto, sin tocar código.
- [ ] El sitio ignora `prefers-reduced-motion` a propósito (ver arriba). Si alguna vez se busca
      cumplir accesibilidad estricta, ese es el primer punto a revisar, junto con el contraste
      de los textos en `--ink-60` sobre crema.
- [ ] `assets/antonella.png` (la foto vieja) se eliminó por no usarse. Si hace falta, está en el
      historial de git.

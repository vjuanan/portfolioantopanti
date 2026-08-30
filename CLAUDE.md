# CLAUDE.md — Portfolio de Antonella Barone

Guía para cualquier agente que trabaje en este repo. Leerla completa antes de tocar código.

## Qué es

Portfolio personal de **Antonella Barone** (investigación social, UX research y gestión de
proyectos). Sitio de una sola página, en español (`lang="es"`), público y en producción.

- **En vivo:** https://portfolioantopanti.vercel.app
- **Repo:** https://github.com/vjuanan/portfolioantopanti (público, rama por defecto `main`)
- **Deploy:** Vercel, proyecto `juanans-projects-d939887f/portfolioantopanti`, conectado por
  integración de GitHub. **Todo push a `main` publica automáticamente en producción.**
  No hay comando de build ni variables de entorno: es un sitio estático puro.

## El PDF fuente — leer esto antes de cambiar el diseño

Todo el lenguaje visual del sitio (paleta, tipografía, formas, fotos) está **calcado de
`Antonella Barone portfolio.pdf`**, la presentación que armó ella en Canva. El PDF no está en
el repo; lo tiene Antonella. Si hay que rehacer o extender una lámina y no lo tenés, pedíselo.

De ahí salieron, medidos del propio archivo:

| Cosa | Valor exacto del PDF |
|---|---|
| Fondo de las láminas / color de marca | `#b99175` (camel) |
| Crema | `#f2f1eb` |
| Tinta (todo el texto) | `#312727` |
| Tipografía | **Glacial Indifference** Regular + Bold |
| Manuscrita (firma y anotaciones) | **Brittany Signature** |

**Ninguna de las dos fuentes está en Google Fonts.** Se sustituyeron por las más parecidas
que sí están, y quedaron declaradas *después* de la original en el stack, así que si algún día
se auto-hospedan las reales entran solas sin tocar CSS:

- `Glacial Indifference` → **Jost** (revival de Futura, mismas proporciones geométricas).
- `Brittany Signature` → **Sacramento** (script monolineal).

La firma "Anto Barone" **no** usa fuente: es `assets/firma-anto.png`, la firma real recortada
del PDF con transparencia. Es más fiel que cualquier sustituto.

## Stack y estructura

HTML, CSS y JavaScript vanilla. **Sin build step, sin dependencias, sin `package.json`.**

```
.
├── index.html      # Página completa: markup + <style> del portfolio + <script> de interacciones
├── styles.css      # Sistema de diseño base "Organic": tokens, fuentes y componentes
├── assets/
│   ├── anto-avatar.jpg      # Foto circular del hero
│   ├── anto-portrait.jpg    # Retrato 4:5 — hoy sólo se usa como imagen de Open Graph (900×1125)
│   ├── anto-pdf-retrato.jpg # Retrato del PDF; es el que va en el polaroid de "Sobre mí"
│   ├── firma-anto.png       # Firma "Anto Barone" recortada del PDF, con transparencia
│   ├── logo-emprendeya.png  # Logo del cliente del Proyecto 1 (círculo, fondo transparente)
│   ├── logo-hsbrands.png    # Logo de HS Brands Global (transparente)
│   ├── p1-reunion.jpg       # Proyecto 1 · paso 1 — captura de la reunión virtual
│   ├── p1-carnet.jpg        # Proyecto 1 · paso 3 — carnet de investigadora sobre una encuesta
│   ├── p1-fundadora.jpg     # Proyecto 1 · paso 3 — foto con la fundadora en el lanzamiento
│   ├── p1-resultados.png    # Proyecto 1 · paso 5 — lámina de resultados (gráfico de dificultad)
│   └── p1-insights.png      # Proyecto 1 · paso 5 — lámina de insights
├── .claude/launch.json  # Config del preview del navegador (python -m http.server 8123)
├── vercel.json     # cleanUrls + cache inmutable de un año para /assets/*
├── CLAUDE.md       # Este archivo
└── README.md
```

**Ojo con el cache inmutable de `/assets/*`:** si cambiás el CONTENIDO de una imagen sin
cambiarle el nombre, quien ya la tenga cacheada sigue viendo la vieja durante un año. Para
reemplazar una foto, usá un nombre nuevo.

**Orden de cascada (importante):** `styles.css` se enlaza *primero* y define los tokens del
sistema base. El `<style>` embebido en `index.html` se carga *después* y **sobrescribe**
`:root` con la paleta del portfolio (camel + crema + tinta). Si un color no cambia cuando lo
tocás, revisá cuál de los dos manda.

### Desarrollo local

```bash
python -m http.server 8000
```

Y abrir http://localhost:8000. Cualquier servidor estático sirve (`npx serve .` también).
Abrir `index.html` con `file://` funciona a medias — las fuentes de Google y algunos
comportamientos se ven raros. Usá el servidor.

## Sistema de diseño

- **Paleta:** crema `#f2f1eb`, tinta `#312727`, camel `#b99175` como acento principal y
  magenta `#e6197f` (el de Emprende Ya) reservado **sólo** para ese proyecto. Las rampas
  (`--color-accent-100` … `-900` y las de magenta y neutros) están generadas en OKLCH sobre
  una misma escala de luminosidad: el paso `-400` de camel pesa visualmente igual que el `-400`
  de magenta. **Si agregás un color, agregalo como rampa completa respetando esa escala**, no
  como hex suelto.
- **Tipografía:** ver la tabla de arriba. Se importan desde Google Fonts en la primera línea
  de `styles.css`. `--font-script` es la manuscrita.
- **Curvas de animación:** `--spring` (rebote corto), `--soft` (salida suave),
  `--drop` (la caída con rebote al acomodarse — es la firma del sitio).
- **`--edge`:** margen lateral fluido, `clamp(20px, 5vw, 80px)`. Usalo en vez de paddings fijos.

## Lenguaje gráfico del PDF (las cuatro piezas que se repiten)

Las láminas se apoyan siempre en los mismos cuatro recursos. Están implementados como
utilidades y conviene reusarlos antes que inventar otros:

1. **`.wavy`** — panel de borde ondulado (el "bloque de texto" de casi todas las láminas).
   Variantes de relleno: `.tan`, `.paper`, `.cream`. Markup:
   ```html
   <div class="wavy tan">
     <svg class="wavy-bg" data-blob="wave" aria-hidden="true"></svg>
     <div class="wavy-in"> …contenido… </div>
   </div>
   ```
2. **`.stamp`** — marco festoneado de las fotos (el borde oscuro de bollitos redondos).
   `<span class="stamp tilt-r" data-blob="scallop"><img …></span>`. Las clases `.tilt-l` /
   `.tilt-r` le dan la inclinación de collage.
3. **`.spark`** — estrellita de cuatro puntas: `<svg class="spark" style="…"><use href="#spark4"/></svg>`.
   El símbolo está definido una sola vez en un `<svg>` de `defs` al principio del `<body>`.
4. **`.doodle`** — garabato de línea fina. Se **dibuja solo** al entrar en viewport: el JS mide
   `getTotalLength()` de cada `path` y lo pone en `--len` para animar el `stroke-dashoffset`.

Además: `.polaroid` (marco crema + filete camel + la firma abajo) y `.hand` (texto manuscrito).

### Cómo se generan las formas orgánicas — `blobPath()`

Tanto la onda como el festoneado salen de **una sola función** en el `<script>`. Recorre el
perímetro de un rectángulo redondeado, desplaza cada punto sobre su normal siguiendo una onda,
y suaviza con Catmull-Rom convertido a cúbicas de Bézier:

- El número de ondas es **entero**, así la curva cierra sin costura.
- Modo `wave` usa `cos` (la onda entra y sale); modo `scallop` usa `abs(cos)` (todos los
  bollos hacia afuera, como el marco de una foto).
- **Se dibuja a la medida real del elemento**, no se estira un SVG fijo. Eso es deliberado: con
  `preserveAspectRatio` la onda quedaría chata y larga en desktop y apretada en mobile, y
  perdería el aire de "hecho a mano". Por eso hay que redibujar cuando cambia el tamaño —
  `drawShapes()` se llama en `load`, en `resize` (con debounce), cuando termina de cargar cada
  imagen de un `.stamp`, cuando resuelve `document.fonts.ready` y al cambiar de filtro.

Si agregás un panel ondulado o un marco festoneado nuevo, alcanza con poner el markup de
arriba: `drawShapes()` los toma por `[data-blob]`.

## Sistema de animación de "caída"

Es la identidad de movimiento del sitio: **todo entra desde arriba, apenas rotado, y se
acomoda con un rebote.** No lo reemplaces por fades genéricos.

- `[data-reveal]` — un elemento que cae. Se configura con variables inline:
  `--fall` (distancia), `--tilt` (rotación de entrada), `--rest` (rotación final), `--d` (delay).
- `[data-cascade]` — un contenedor cuyos hijos caen escalonados; el JS le asigna a cada hijo
  un `--d` incremental de 0.09 s y alterna el `--tilt` en los pares.
- `[data-watch]` — solo marca una sección para que el observer le ponga la clase `.in`.
- Un único `IntersectionObserver` (threshold 0.12) agrega `.in` y **deja de observar**
  (`unobserve`): las animaciones ocurren una sola vez, no se repiten al volver a scrollear.
- `.mask > span` — títulos que suben desde detrás de una máscara al entrar en viewport.
- Los títulos de los pasos del Proyecto 1 (`.step-t`) van **grandes y sin regla debajo** (se
  probó y quedó pobre). Encadenan dos tiempos: el JS los parte en palabras (`.w`, recorriendo
  sólo los nodos de texto para no romper el `<i>` de adentro) y cada una sube desde atrás de la
  máscara con `--i * 0.06s`; cuando aterriza la última, un marcador camel (`.hl`, un
  `background-size` de `0%` a `100%`) barre el título entero. El retardo del marcador lo calcula
  el JS según cuántas palabras haya (`--hld`). El `.hl` usa `box-decoration-break: clone` para
  que cuando el título cae en dos renglones la banda se dibuje en los dos. **Todos van alineados
  a la izquierda:** hubo una versión donde los pasos pares iban a la derecha y se veía
  desprolijo apenas el título envolvía, con la segunda línea colgando. La alternancia
  izquierda/derecha sigue viva pero sólo en el `.step-grid` (panel y foto), que sí funciona.
- `.name .ch` — el nombre del hero se parte letra por letra desde JS (`.split-letters`) y cada
  letra cae con `--i * 0.05s` de retraso. "Barone" va en `outline` (contorno) y se rellena de
  camel al hover en desktop o al entrar en viewport en touch (`.name.lit`).

**Al probar:** si saltás con `scrollTo` de golpe, los elementos que te salteás nunca
intersectan y quedan en `opacity: 0`. No es un bug — scrolleá de a poco, o forzá `.in` a mano
mientras revisás maquetación.

### Decisión deliberada: `prefers-reduced-motion` se ignora

En el `<script>` hay `const reduce = false;` con todas las ramas de reducción cableadas pero
inactivas. **Es intencional y fue pedido explícitamente** (commit `f463de1`): el portfolio
tiene que lucir todos los efectos aunque la máquina de quien lo mira tenga activado "reducir
movimiento", porque es una pieza de presentación. **No lo "arregles" por accesibilidad sin
preguntar.** Si algún día se revierte, alcanza con volver a
`const reduce = matchMedia('(prefers-reduced-motion: reduce)').matches;` — el resto del código
ya respeta esa bandera.

## Secciones

En orden: **hero** (la tapa del PDF: "Research & Insights" + el nombre) → marquee →
**01 Sobre mí** (con el índice al final) → **02 Formación & Habilidades** →
**03 Mi experiencia** → **04 Mis proyectos** → **05 Contacto**.

El orden lo pidió Antonella: primero quién es, después qué sabe, dónde lo aprendió y qué hizo.
**Si agregás una sección, actualizá tres lugares:** el `<nav>`, el numerito del `.kicker` y el
índice.

### El acordeón (`.acc` / `.acc-item` / `.acc-head`)

**"Sobre mí" es la única `<section>` suelta.** Las otras cuatro viven dentro del acordeón: cada
una es una píldora que abre su contenido ahí mismo, sin mandarte a otro lado. Los encabezados
quedan pegados arriba y se van **apilando**, así cualquier sección se puede elegir mientras se
lee otra, sin volver para arriba.

Tres decisiones que lo sostienen y que no son evidentes leyendo el CSS:

1. **`.acc-item` usa `display: contents`.** Si el encabezado viviera dentro de la caja de su
   propio item, `position: sticky` lo soltaría apenas ese item sale de pantalla y la pila se
   desarmaría. Sin caja, el bloque contenedor pasa a ser `.acc` y cada encabezado se queda
   pegado hasta el final del acordeón. Las custom properties (`--i`, `--hb`, `--hf`) igual
   heredan a través de `display: contents`.
2. **El panel NO anima su altura.** El de proyectos mide miles de píxeles; animar
   `grid-template-rows` sobre eso obliga a recalcular layout en cada frame y es justo lo que se
   sentía trabado. Se muestra de una (`display`) y lo que anima es el contenido, que emerge
   escalonado — y de paso tapa el salto de altura.
3. **El offset de pegado sale del alto real del nav.** El JS mide `nav.offsetHeight` y lo
   escribe en `--nav-h`; cada encabezado se pega en
   `calc(var(--nav-h) + var(--i) * var(--acc-h))`. Si tocás el nav, esto se reacomoda solo.
4. **Los encabezados tienen ALTO FIJO (`height: var(--acc-h)`), y `--acc-h` vale exactamente
   eso.** Es lo que garantiza que el tope de apilado coincida siempre con el alto real. Hubo una
   versión donde el encabezado se compactaba al pegarse (escondía su bajada): mientras esa
   transición corría — o si el observador de "pegado" no llegaba a tiempo — el alto no coincidía
   con el tope y **las píldoras se montaban una sobre otra**. Por eso el encabezado es de una
   sola línea: número, título y flecha. Si algún día le agregás algo, que no le cambie el alto.

Al abrir, el JS:

- llama a `drawShapes()` **de forma síncrona** (no dentro de un `requestAnimationFrame`): leer
  `offsetWidth` fuerza el recálculo de estilos, así el panel ya mide y las ondas se dibujan con
  su ancho real. Dentro de un rAF queda a merced de que el navegador decida pintar;
- revela lo que ya está en pantalla **o por encima**, y le devuelve el resto al
  `IntersectionObserver`. Si se les pone `.in` a todos de golpe, los cinco pasos del caso animan
  juntos al abrir y para cuando llegás scrolleando ya pasó todo: se ve estático, que es justo lo
  contrario de lo que se busca. Lo de "por encima" no es un detalle: el observador sólo avisa
  cuando algo **entra**, así que un bloque que al abrir ya quedó pasado no se revelaría nunca.
  Al cerrar les saca `.in`, así la próxima vez vuelven a emerger.

**Ojo con lo que se oculta esperando ser revelado.** Un elemento sin `.in` sigue ocupando su
alto: si nadie se lo da, queda un hueco enorme y en blanco. Ya pasó con los `.h-sect` de los
paneles, que no tenían ningún ancestro revelable — por eso ahora **todos los `<h2 class="h-sect">`
llevan `data-watch`**. Si agregás contenido con máscara o `[data-reveal]` dentro de un panel,
asegurate de que algo se lo revele.

Los enlaces del `<nav>` que apuntan a una sección del acordeón **abren el item** en vez de saltar
a un ancla; lo mismo con el `#hash` al cargar.

### Mis proyectos y el filtro

Dos chips (`.chip[data-filter]`): **`Todos`** y **`Proyecto 1`**. Cada bloque filtrable lleva
`data-cat`; el que no matchea recibe `.is-hidden`.

La regla de visibilidad tiene una vuelta de tuerca — **`[data-only]`**:

```js
const show = el.dataset.cat === want || (want === 'all' && !el.hasAttribute('data-only'));
```

`#p1case` lleva `data-only`, así que **queda fuera de "Todos"**. En "Todos" se ve sólo una
tarjeta resumen (`data-cat="teaser"`) y el caso entero aparece recién al elegir el filtro
"Proyecto 1" — o al tocar "Ver el caso completo", un `<button data-goto="p1">` que elige el
filtro y baja a la sección. Fue un pedido explícito: no volcar todo el caso de entrada.

Tres detalles que ya costaron una vuelta:

- **El zigzag de las tarjetas lo asigna el JS (`restripe()`), no `:nth-child`.** Con
  `:nth-child(even)` las tarjetas ocultas seguían contando y al filtrar el zigzag se rompía.
- Lo que aparece al filtrar recibe `.in` a mano: si no, un bloque que nunca intersectó
  quedaría invisible al mostrarlo.
- El click de la tarjeta ignora `a, button`, si no el botón "Ver el caso completo" abriría y
  cerraría la tarjeta al mismo tiempo que cambia el filtro.

**Proyecto 1 (`#p1case`)** no es una tarjeta compacta sino el caso completo, lámina por lámina:
cabecera con el logo de Emprende Ya, panel "Contexto & Objetivo general" y los cinco pasos con
sus fotos. Va **fuera** de `#plist` a propósito: `#plist` recibe un `skewY` continuo por inercia
de scroll, y sobre un bloque tan alto ese cizallamiento se ve mal.

## Capa táctil (mobile) — lección aprendida

En desktop las tarjetas de proyecto se expanden con `:hover`. En touch no hay hover, así que:

- `document.documentElement` lleva la clase `is-touch` cuando `(hover: none)` o `(pointer: coarse)`.
- La tarjeta más centrada en pantalla se auto-realza con la clase `.focus` mientras se scrollea.
- **El bug que ya se corrigió (commit `1281ad8`): parpadeo/bucle abrir-cerrar.** Al expandirse,
  una tarjeta cambia de alto y mueve la geometría, lo que hacía que el cálculo de "cuál está
  más centrada" saltara entre dos tarjetas en loop. La solución vigente en `focusNearest()`:
  1. se mide la distancia al **header** de la tarjeta (`.pc-top`), que no se mueve al expandirse,
     no al elemento completo;
  2. hay **histéresis** — una tarjeta ya enfocada se mantiene hasta que otra esté claramente más
     centrada (`bd < cd - innerHeight * 0.14`), y solo se adquiere el foco si está bien centrada; y
  3. se descartan las tarjetas ocultas por el filtro (`.is-hidden` / sin `offsetParent`), si no
     el foco podía quedarse pegado a una tarjeta que ya no se ve.
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

## Texto en contorno: siempre con `paint-order`

Los glifos de **Jost están construidos con contornos superpuestos**. Rellenos no se nota, pero
al trazarlos (`-webkit-text-stroke` con `color: transparent`) se dibuja cada costura interna y
aparecen líneas cruzando cada letra. No es culpa del tracking ni de la versión variable: la
instancia estática tiene exactamente el mismo problema.

La única solución es **pintar el trazo primero y el relleno encima**:

```css
color: var(--color-bg);            /* el color del fondo que hay DETRÁS del texto */
-webkit-text-stroke: 5px var(--color-text);
paint-order: stroke fill;          /* el relleno tapa la mitad interna del trazo */
```

Tres consecuencias a tener en cuenta:

1. **El trazo va al doble de grosor**, porque la mitad de adentro queda cubierta.
2. **El relleno tiene que ser el color del fondo real.** Si movés un texto en contorno a otro
   lado, hay que cambiarle el relleno (por eso "juntos?" del bloque de contacto usa
   `--color-accent-900` y no `--color-bg`).
3. Como el relleno es opaco, dos letras que se solapen se fusionan. Por eso los renglones en
   contorno llevan un tracking apenas positivo (`letter-spacing: 0.008em`) mientras el resto de
   los títulos usa tracking negativo.

Está aplicado en tres lugares: `.name .outline` ("Barone"), `.marq-item:nth-child(4n+3)` y
`.patch .h-sect .thin` ("juntos?").

## Movimiento: barato o nada

El pedido fue que la información "emerja" en vez de estar quieta. Todo lo que se mueve de forma
continua anima **sólo `opacity` y `transform`**, que van por GPU y no cuestan layout:

- `.emerge > *` — los hijos de un panel entran escalonados (`--i` lo asigna el JS).
- `.spark` — las estrellitas titilan.
- `.wavy-bg` — la onda "respira" con un `scale`/`rotate` mínimo.

**La onda respira con un transform, NO deformando el path.** Se probó morfear la `d` con
`<animate>` de SMIL entre tres fases y hay que descartarlo: con **sólo dos paneles** el atributo
`values` pesaba **63 KB** de datos de path que el navegador interpola en cada frame, y
regenerarlos costaba ~150 ms. En el sitio hay ocho paneles. Se sentía trabado. No lo reintentes.

## Trampa de composición: nada de `backdrop-filter` sobre `.bg-fx`

`.bg-fx` es la capa `position: fixed` con los brillos y el grano. Dos cosas la sostienen y las
dos están comentadas en el CSS:

- Lleva `transform: translateZ(0)` para forzar su propia capa de composición.
- **El nav NO usa `backdrop-filter`.** Con el blur puesto, Chromium tomaba esa capa fija como
  *backdrop root* al hacer scroll y **pintaba media página (o la página entera) de negro**.
  Costó un rato encontrarlo porque el DOM se veía perfecto: el contenido estaba, sólo que no se
  pintaba. Hoy `.nav.scrolled` usa un fondo crema casi opaco, que se ve igual y no tiene ese
  riesgo. **No vuelvas a poner `backdrop-filter` ahí.**

## Contenido

Los datos del sitio son **reales**: salen del PDF y de su CV. No inventes proyectos, cifras ni
credenciales.

- **Proyecto 1 — Emprende Ya** (caso completo, del PDF): app fundada por Nancy Mendoza, mapa
  social de emprendedores de barrio. Investigación exploratoria **mixta** durante el evento de
  lanzamiento: **55 encuestas** en papel y **5 entrevistas** semiestructuradas. Los cinco pasos
  son: reunión con stakeholders → enfoque metodológico e instrumentos → trabajo de campo →
  digitalización y análisis → presentación de resultados. Los porcentajes (91 %, 70,1 %) se ven
  en la lámina de resultados del paso 5; **no** van como cifras sueltas — había una fila de
  contadores animados y Antonella pidió sacarla porque descolgaba del relato. Con eso quedó sin
  uso todo el sistema de contadores (`data-count`) y de `.stat`: se eliminó, está en el historial.
- **Los otros proyectos ya no están en el sitio.** Había cuatro tarjetas más (web internacional
  de HS Brands Latam, cuestionarios en el CRM interno, investigación de usuarios y diseño
  instruccional en Efe Idiomas 2018–2025) y Antonella pidió sacarlas: el portfolio queda
  enfocado en el caso de Emprende Ya, igual que el PDF. Los datos son reales y están en el
  historial de git — si algún día vuelven, el markup de las `.pcard` sirve tal cual y hay que
  devolverles su chip de filtro.
- **Mi experiencia:** Project Assistant en HS Brands Global (agencia global de investigación
  de mercado), con los cuatro bullets de la lámina "Experiencia". El período real es
  **abril 2025 – junio 2026**, pero **hoy no se muestra**: la anotación manuscrita con la fecha
  se sacó a pedido. Si vuelve, que sea con `white-space: nowrap` — al envolver, "2026" cae solo
  y se lee mal.
- **Formación:** Sociología UBA 2022–2026, Curso Inicial de programación con Python (Agencia de
  Habilidades para el Futuro, 2024), Inglés B2 (FCE), Curso Inicial de Diseño UX/UI, Lovable L1.
- **Contacto:** antonella.m.barone@gmail.com · LinkedIn `antonella-barone2003` · Vicente López, BA.
- **WhatsApp:** en dos lugares, los dos a `wa.me/5491164613463` — en pantalla se lee
  **+54 11 6461 3463**. El **9** después del 54 no es un error: WhatsApp lo pide para los
  celulares argentinos; sin él el link no abre el chat. Si tocás el número, mantenelo.
  1. Botón fijo abajo a la derecha (`.wa`), en camel para no romper la paleta.
  2. Segundo canal en el bloque de contacto, debajo del mail (`.bigs > .biglink.is-wa`).
     Los dos canales se diferencian por **peso**, no por tono: el diseño original los
     separaba con malva, pero ese malva hoy es el magenta de Emprende Ya y está reservado
     para ese proyecto.

## Meta tags y compartir

Las URLs de `og:image` / `twitter:image` **tienen que ser absolutas**
(`https://portfolioantopanti.vercel.app/assets/...`), con `og:image:width` y `og:image:height`
declarados. Con rutas relativas WhatsApp/LinkedIn no levantan la portada — ya pasó y se corrigió
en el commit `4901db0`. Si cambiás la foto de la portada, actualizá también esas dimensiones.

El favicon es un SVG embebido como data-URI en el `<link rel="icon">`: una "A" crema sobre camel.

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

- [ ] Sólo el Proyecto 1 tiene caso completo. Los otros tres siguen siendo tarjetas: si
      aparecen materiales (capturas, documentos), se pueden desarrollar con la misma estructura
      de `#p1case` y sumarles su chip de filtro.
- [ ] Auto-hospedar **Glacial Indifference** y **Brittany Signature** para calcar el PDF al 100 %.
      Ya están declaradas primeras en `--font-heading` / `--font-script`: alcanza con agregar los
      `@font-face`.
- [ ] Las láminas de resultados e insights del paso 5 se leen chicas. Estaría bueno un lightbox
      para verlas en grande.
- [ ] No hay CV descargable en PDF. Sería el complemento natural del bloque de contacto.
- [ ] Dominio propio en vez del subdominio `.vercel.app` (se configura en Vercel + DNS).
- [ ] Sin analítica. Vercel Analytics se activa desde el panel del proyecto, sin tocar código.
- [ ] La imagen de Open Graph sigue siendo `anto-portrait.jpg` (la foto anterior). Si se quiere
      alinear con la identidad nueva, hay que generar una portada 1200×630 y actualizar las
      dimensiones en los meta tags.
- [ ] El sitio ignora `prefers-reduced-motion` a propósito (ver arriba). Si alguna vez se busca
      cumplir accesibilidad estricta, ese es el primer punto a revisar.

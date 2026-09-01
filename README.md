# Antonella Barone — Portfolio

Sitio de presentación (portfolio) de **Antonella Barone** — *Research & Insights*: investigación
social, UX research y gestión de proyectos.

**En vivo:** <https://www.antonellabarone.com.ar>

El diseño está calcado de su portfolio en PDF: paleta camel `#b99175` + crema `#f2f1eb` + tinta
`#312727`, tipografía geométrica (*Jost*, en lugar de la *Glacial Indifference* del original) y
manuscrita (*Sacramento*, por *Brittany Signature*), con los recursos gráficos que se repiten en
las láminas — paneles de borde ondulado, marcos festoneados, estrellitas y garabatos de línea
fina. Encima, el sistema de animación de "caída" (los elementos entran desde arriba y se acomodan
con un rebote) y microinteracciones en scroll, hover y foco.

## Stack

Sitio estático — HTML, CSS y JavaScript vanilla. Sin build step ni dependencias.

```
.
├── index.html      # Página única: markup + estilos del portfolio + interacciones
├── styles.css      # Sistema de diseño base "Organic" (tokens, fuentes y componentes)
├── assets/         # Fotos, logos y la firma, extraídos del portfolio en PDF
├── vercel.json     # Configuración de deploy (sitio estático)
└── CLAUDE.md       # Guía técnica del proyecto
```

`styles.css` se carga primero (define tokens, fuentes de Google y componentes base como `.tag`);
los estilos del portfolio, embebidos en `index.html`, se cargan después y sobrescriben la paleta.

## Secciones

Hero · **Sobre mí** · **Formación & Habilidades** · **Mi experiencia** · **Mis proyectos** ·
**Contacto**.

"Sobre mí" es la única sección que se lee scrolleando. Las otras cuatro viven detrás de un
**índice de tarjetas**: se elige una y es la única que existe abajo. Nada se pega arriba ni se
apila, y la página queda corta.

En "Mis proyectos" hay dos filtros, *Todos* y *Proyecto 1*. En *Todos* se ve una tarjeta resumen;
el caso completo de **Emprende Ya** — contexto y objetivo y los cinco pasos de la investigación
con sus fotos — aparece al elegir *Proyecto 1*.

## Desarrollo local

Al ser estático, alcanza con servir la carpeta con cualquier servidor:

```bash
python -m http.server 8000
```

Luego abrir <http://localhost:8000>.

## Deploy

Desplegado en **Vercel** como sitio estático (sin comando de build). Cada push a `main` publica
una nueva versión en producción automáticamente.

## Características

- Diseño responsive (breakpoints en 720 / 780 / 820 / 860 / 880 / 900 / 1024 px).
- Formas orgánicas (ondas y festoneados) generadas por JS a la medida real de cada bloque, así
  la onda no se deforma entre desktop y mobile.
- Índice de tarjetas: una sección por vez, sin nada fijo arriba.
- Botón de WhatsApp fijo abajo a la derecha.
- Filtro de proyectos, con bloques que quedan fuera de "Todos" (`[data-only]`) y el zigzag de las
  tarjetas recalculado sobre las visibles.
- Títulos de paso que se resaltan con un marcador que barre de lado a lado al entrar en pantalla.
- Animaciones de entrada por `IntersectionObserver` con máscaras y cascadas.
- Tarjetas de proyecto expandibles (hover en desktop, tap/Enter en touch/teclado).
- Barra de progreso de scroll e inercia (skew) en la pila de proyectos.
- Marquee, parallax y efectos de scroll en un único bucle de `requestAnimationFrame`.
- Meta tags de SEO y Open Graph, favicon SVG embebido.
- Las animaciones se muestran siempre, por decisión de diseño: el sitio **no** honra
  `prefers-reduced-motion` (ver `CLAUDE.md`).

## Documentación

`CLAUDE.md` reúne el detalle técnico: de dónde sale cada valor del diseño, cómo funcionan las
formas orgánicas y la animación de caída, la capa táctil, las trampas ya encontradas y los
pendientes abiertos.

---

© 2026 Antonella Barone

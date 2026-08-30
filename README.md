# Antonella Barone — Portfolio

Sitio de presentación (portfolio) de **Antonella Barone** — *Research & Insights*: investigación
social, UX research y gestión de proyectos.

**En vivo:** <https://portfolioantopanti.vercel.app>

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

Hero · **Sobre mí** · **Formación & Habilidades** · **Mi experiencia** ·
**Mis proyectos** (con filtro: *Todos · Proyecto 1 · HS Brands · Educación*) · **Contacto**.

"Sobre mí" cierra con un **índice**: una píldora por sección que funciona como menú de entrada
al sitio.

*Proyecto 1 — Emprende Ya* es un caso de estudio completo: contexto y objetivo y los cinco pasos
de la investigación con sus fotos.

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
- Índice de secciones clickeable al final de "Sobre mí", como entrada al sitio.
- Filtro de proyectos por categoría, con el zigzag de las tarjetas recalculado sobre las visibles.
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

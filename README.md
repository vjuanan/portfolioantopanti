# Antonella Barone — Portfolio

Sitio de presentación (portfolio) de **Antonella Barone** — investigación social, gestión de proyectos y UX.

**En vivo:** <https://portfolioantopanti.vercel.app>

Diseño editorial crema + tinta con acento rosa, tipografía *Caprasimo* + *Figtree*, y un sistema de animación de "caída" (los elementos entran desde arriba y se acomodan con un rebote) más microinteracciones en scroll, hover y foco.

## Stack

Sitio estático — HTML, CSS y JavaScript vanilla. Sin build step ni dependencias.

```
.
├── index.html      # Página única: markup + estilos del portfolio + interacciones
├── styles.css      # Sistema de diseño base "Organic" (tokens, fuentes y componentes)
├── assets/
│   ├── anto-avatar.jpg    # Foto del hero
│   └── anto-portrait.jpg  # Retrato 4:5 (Sobre mí + Open Graph)
├── vercel.json     # Configuración de deploy (sitio estático)
└── CLAUDE.md       # Guía técnica del proyecto
```

`styles.css` se carga primero (define tokens, fuentes de Google y componentes base como `.tag`); los estilos del portfolio, embebidos en `index.html`, se cargan después y sobrescriben la paleta con la variante crema + rosa.

## Desarrollo local

Al ser estático, alcanza con servir la carpeta con cualquier servidor:

```bash
python3 -m http.server 8000
# o
npx serve .
```

Luego abrir <http://localhost:8000>.

## Deploy

Desplegado en **Vercel** como sitio estático (sin comando de build). Cada push a `main` publica una nueva versión en producción automáticamente.

## Características

- Diseño responsive (breakpoints en 640 / 720 / 880 / 1024 px).
- Animaciones de entrada por `IntersectionObserver` con máscaras y cascadas.
- Contadores animados en las estadísticas.
- Tarjetas de proyecto expandibles (hover en desktop, tap/Enter en touch/teclado).
- Barra de progreso de scroll e inercia (skew) en la pila de proyectos.
- Marquee, parallax y efectos de scroll en un único bucle de `requestAnimationFrame`.
- Meta tags de SEO y Open Graph, favicon SVG embebido.
- Las animaciones se muestran siempre, por decisión de diseño: el sitio **no** honra `prefers-reduced-motion` (ver `CLAUDE.md`).

## Documentación

`CLAUDE.md` reúne el detalle técnico: sistema de diseño, cómo funciona la animación de caída, la capa táctil, las decisiones tomadas y los pendientes abiertos.

---

© 2026 Antonella Barone

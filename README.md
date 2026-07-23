# Antonella Barone — Portfolio

Sitio de presentación (portfolio) de **Antonella Barone** — investigación social, gestión de proyectos y UX.

Diseño editorial crema + tinta con acento rosa, tipografía *Caprasimo* + *Figtree*, y un sistema de animación de "caída" (los elementos entran desde arriba y se acomodan con un rebote) más microinteracciones en scroll, hover y foco.

## Stack

Sitio estático — HTML, CSS y JavaScript vanilla. Sin build step ni dependencias.

```
.
├── index.html      # Página única: markup + estilos del portfolio + interacciones
├── styles.css      # Sistema de diseño base "Organic" (tokens, fuentes y componentes)
├── assets/
│   └── antonella.png
└── vercel.json     # Configuración de deploy (sitio estático)
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

Desplegado en **Vercel** como sitio estático (sin comando de build). Cada push a la rama de producción publica una nueva versión.

## Características

- Diseño responsive (breakpoints en 640 / 720 / 880 / 1024 px).
- Animaciones de entrada por `IntersectionObserver` con máscaras y cascadas.
- Contadores animados en las estadísticas.
- Tarjetas de proyecto expandibles (hover en desktop, tap/Enter en touch/teclado).
- Barra de progreso de scroll e inercia (skew) en la pila de proyectos.
- Respeta `prefers-reduced-motion`.
- Meta tags de SEO y Open Graph, favicon SVG embebido.

---

© 2026 Antonella Barone

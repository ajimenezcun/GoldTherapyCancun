# AGENT.md — Guías del Proyecto

Proyecto: Gold Therapy

## Stack

- **Framework:** Astro 7+
- **Estilos:** Tailwind CSS 4+
- **JavaScript:** Vanilla JS (ES Modules), sin frameworks de UI
- **Animaciones:** GSAP 3+

## Reglas de generación de código

### General
- Responde siempre en español, salvo nombres de variables, funciones y archivos.
- Antes de generar código, verifica si ya existe un componente o utilidad que resuelva el problema.
- Prefiere soluciones simples sobre abstracciones innecesarias.

### Astro
- Los componentes van en `src/components/`, las páginas en `src/pages/`.
- Tipas siempre las props con TypeScript inline en el frontmatter.
- Usas `client:load` solo cuando la interactividad es inmediata; `client:visible` para el resto.
- No generas componentes React/Vue/Svelte salvo indicación explícita.

### JavaScript
- Usas `const` por defecto, `let` solo si la variable muta. Nunca `var`.
- Los módulos de animación van en `src/scripts/animations/`.
- Las funciones de utilidad van en `src/scripts/utils/`.
- Seleccionas elementos del DOM con `data-*` attributes, no con clases de Tailwind.

### Tailwind CSS v4
- La configuración ya **no usa** `tailwind.config.js`. Todo se define en CSS.
- Los tokens personalizados (colores, fuentes, espaciados) se declaran en `src/styles/global.css` con `@theme`:
  ```css
  @import "tailwindcss";
 
  @theme {
    --color-primary: #1E3A5F;
    --font-sans: "Inter", sans-serif;
  }
  ```
- Para extender utilidades propias se usa `@utility` en el mismo archivo.
- No uses `@apply` para construir componentes; prefiere clases directamente en el HTML.
- Clases en orden: layout → spacing → typography → color → estado/responsive.
 
### GSAP
- Importa siempre desde el paquete: `import { gsap } from 'gsap'`.
- Para animar un elemento: `gsap.from(elemento, { opacity: 0, y: 30, duration: 0.8 })`.
- Si necesitas animar al hacer scroll, usa `ScrollTrigger`:
  ```js
  import { ScrollTrigger } from 'gsap/ScrollTrigger';
  gsap.registerPlugin(ScrollTrigger);
  ```
- Cuando propongas una animación, explica brevemente qué hace cada propiedad.
- Empieza con animaciones simples (`from`, `to`); solo usa `timeline()` cuando haya secuencias de varios pasos.

## Lo que NO debes hacer
- Generar `innerHTML` con strings sin escapar.
- Importar librerías no listadas en `package.json`.
- Crear estilos inline en el HTML.
- Usar `document.write()`.
- Añadir comentarios obvios o redundantes.
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

## Patrones de UI/UX (coherencia visual entre páginas)

Estos patrones surgieron al iterar la sección de Servicios del homepage (rama `ui-ux`) y deben reutilizarse en otras páginas para mantener consistencia.

### Estructura estándar de una sección
- Encabezado centrado de 3 niveles (usar solo los que apliquen):
  1. Eyebrow: `<p class="text-center text-sm font-bold uppercase tracking-widest text-primary-strong">`.
  2. Título: `<h2 class="mt-3 text-center text-[32px]">`.
  3. Subtítulo/overview: `<p class="mx-auto mt-4 max-w-160 text-center text-lg leading-relaxed text-on-surface-variant">`.
- Alterna el fondo entre secciones consecutivas con `bg-surface-container-low` para dar ritmo visual. Nunca dejes dos secciones seguidas con el mismo fondo (revisa la sección anterior y la siguiente antes de decidir).
- **Eyebrow con moderación:** no le pongas eyebrow a cada sección — es el "tell" más común de que una página la armó una IA. Máximo ~1 eyebrow cada 3 secciones de la página completa. Si el `<h2>` ya deja claro de qué trata la sección (p. ej. "Lo que dicen nuestros pacientes"), omite el eyebrow.
- **Tipografía de encabezados:** `h1`-`h4` usan `--font-display` (`Plus Jakarta Sans`, definida en `global.css`), no `--font-sans` (`Lato`, reservada para párrafos/cuerpo). Mantén esta separación al agregar nuevos títulos; no reintroduzcas una sola fuente para todo.
- **Sin em-dash (`—`/`–`) en copy visible al usuario** (títulos, párrafos, botones, alt text). Usa coma, punto o guion normal (`-`).

### Cards y superficies
- `.card` (fondo blanco + borde + `radius-lg`) es para superficies "flotantes" (testimonios, tarjeta de equipo). Si una sección necesita verse más integrada al fondo (sin caja blanca ni borde), no reutilices `.card`: arma las clases directamente y, si necesitas un separador entre items, usa `border-b` en vez de fondo/borde propio.
- Para quitar el borde del último elemento de una lista generada con `.map()`, usa `class:list` con el `index`: `index !== items.length - 1 && "border-b ..."`.

### Imágenes que se funden con el fondo (vignette)
Para que el borde de una imagen se difumine con el fondo de la sección en vez de cortar en un rectángulo duro, aplica una máscara radial al contenedor de la imagen (no a la `<img>`):
```
class="[mask-image:radial-gradient(ellipse_at_center,black_35%,transparent_75%)] [-webkit-mask-image:radial-gradient(ellipse_at_center,black_35%,transparent_75%)]"
```
Ajusta los porcentajes para más o menos difuminado (menor primer valor = núcleo nítido más pequeño = look más "redondo").

### Enlaces secundarios tipo "saber más"
Patrón de link con flecha que se separa del texto al hacer hover:
```
class="flex w-fit items-center gap-1 font-bold text-primary-strong transition-all hover:gap-2"
```
seguido de un `<svg>` de flecha (`stroke="currentColor"`).

### Botón primario (CTA)
```
class="rounded-md flex gap-1 items-center bg-primary px-5 py-3 font-bold uppercase tracking-wide text-on-primary transition-all hover:bg-primary-strong hover:translate-y-[-2px] hover:shadow-lg"
```

### Carruseles infinitos (marquee)
1. Duplica el array de items una vez (`[...items, ...items]`) y renderízalos en una pista `flex w-max gap-*` (`data-carousel-track`) dentro de un contenedor `overflow-hidden` (`data-carousel`).
2. Difumina los bordes del contenedor con `[mask-image:linear-gradient(to_right,transparent,black_5%,black_95%,transparent)]`.
3. Anima con GSAP en el `<script>` de la página:
   ```js
   const cycleDistance = track.scrollWidth / 2; // la mitad = un ciclo, porque el contenido está duplicado
   const marquee = gsap.to(track, {
     x: -cycleDistance,
     duration: cycleDistance / 40, // velocidad constante (~40px/s)
     ease: "none",
     repeat: -1,
   });
   ```
   Pausa en `mouseenter` y reanuda en `mouseleave` del contenedor.

### Scripts por página
Los scripts que solo aplican a una sección específica de una página (acordeón de FAQ, carrusel de testimonios) pueden vivir en un `<script>` inline al final de esa página, seleccionando elementos con `data-*`. Si la misma lógica se necesita en más de una página, extráela a `src/scripts/utils/` o `src/scripts/animations/`.

## Lo que NO debes hacer
- Generar `innerHTML` con strings sin escapar.
- Importar librerías no listadas en `package.json`.
- Crear estilos inline en el HTML.
- Usar `document.write()`.
- Añadir comentarios obvios o redundantes.
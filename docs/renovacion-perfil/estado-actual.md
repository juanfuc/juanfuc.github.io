# Estado actual de la implementación

Describe **cómo está construido el sitio hoy**, tal como existe en el repositorio y en el despliegue. Es una foto del punto de partida; se actualiza al cerrar cada ronda que cambie la implementación.

> Fecha de la foto: R00 (documentación inicial).

## Stack

- **SvelteKit 2.x** sobre **Svelte 4.2.7**, **Vite 5** (`package.json`).
- Adaptador de despliegue: **`@sveltejs/adapter-vercel`** (`svelte.config.js`). Despliegue en **Vercel** → https://juanfuc.vercel.app/.
- **Prerender global activado**: `export const prerender = true` en `src/routes/+layout.js`.
- **Node: entorno observado** = Node 22 (la máquina donde se ejecutó R00). **El proyecto no fija una versión**: no hay campo `engines` en `package.json`, ni `.nvmrc`, ni `.node-version`. `.npmrc` solo activa `engine-strict=true` (que no fija versión por sí mismo).
- Iconos: `@iconify/svelte` (carga de iconos en runtime).
- Fetch de datos: el código importa **`d3-fetch`** (`import { csv } from 'd3-fetch'`); `package.json` declara el paquete **agregador `d3`** para obtener ese subpaquete. No se ha medido el bundle: no se afirma que «todo D3» llegue al cliente.

## Rutas (file-based routing)

| Ruta | Archivo | Contenido |
|------|---------|-----------|
| `/` | `src/routes/+page.svelte` (**vacío**) + `src/routes/+layout.svelte` | El home real (nav + grilla) vive en el **layout**, condicionado por la bandera `isHome`. Grilla **dinámica** (Sheets, en cliente). |
| `/perfil` | `src/routes/perfil/+page.svelte` | Texto de perfil **estático** (no usa fetch ni `localStorage`), presente en el HTML prerenderizado; animación de aparición por párrafos vía CSS/`setTimeout`. |
| `/experiencia` | `src/routes/experiencia/+page.svelte` | Lista de cargos desde CSV. |
| `/publicaciones` | `src/routes/publicaciones/+page.svelte` | Grilla + enlaces a subsecciones. |
| `/publicaciones/articulos` | `.../articulos/+page.svelte` | Lista de artículos desde CSV. |
| `/publicaciones/libros` | `.../libros/+page.svelte` | Lista de libros desde CSV. |
| `/publicaciones/capitulos` | `.../capitulos/+page.svelte` | Lista de capítulos desde CSV. |
| `/programacion` | `src/routes/programacion/+page.svelte` | Proyectos de programación creativa / HD en `<iframe>`. |

## Fuentes de datos

Las **listas dinámicas** se cargan **en el cliente** (`onMount`) desde **hojas de Google Sheets publicadas como CSV**, parseadas con `d3-fetch` (una hoja por sección). **No todo el contenido es dinámico ni todo se cachea**: el texto de `/perfil` es estático y no toca la red ni `localStorage`.

### Mapa preciso de `localStorage` (verificado por grep, R00-respuesta)

| Página | ¿Fetch a Sheets? | ¿Cachea datos en `localStorage`? | Clave / TTL |
|--------|------------------|----------------------------------|-------------|
| `/` (grilla, `+layout.svelte`) | Sí | **No** (los datos de la grilla NO se guardan) | usa `localStorage` solo para `lastPage` (bandera de navegación), no para datos |
| `/perfil` | **No** (estático) | **No** | — |
| `/experiencia` | Sí | Sí | `experienceData` (+ `experienceDataTimestamp`), **TTL 1 h** |
| `/publicaciones` | Sí | Sí | `publicationsData`, **sin expiración** |
| `/publicaciones/articulos` | Sí | Sí | `articlesData`, **sin expiración** |
| `/publicaciones/libros` | Sí | Sí | `booksData`, **sin expiración** |
| `/publicaciones/capitulos` | Sí | Sí | `chaptersData`, **sin expiración** |
| `/programacion` | Sí | Sí | `projectsData` (+ `projectsDataTimestamp`), **TTL 1 h** |

Caché **inconsistente**: experiencia y programación expiran a la hora; publicaciones/artículos/libros/capítulos **no expiran** (no se refrescan hasta limpiar `localStorage`); la grilla de portada **no se cachea** (se rebaraja al cargar/recargar la app, ver más abajo).

## Comportamiento de la grilla (portada)

- Definida en `src/routes/+layout.svelte`, bloque `{#if isHome}`.
- `isHome` se decide leyendo `localStorage.getItem('lastPage')` en `onMount`; se actualiza en cada clic de navegación (`handleNavigation`). Lógica **frágil**: depende de `localStorage` y del último clic, no de la ruta actual.
- Los datos se **barajan (`shuffleArray`) en `onMount` del layout raíz**. Como el layout raíz **persiste durante la navegación interna de SvelteKit** (no re-ejecuta `onMount`), el rebarajado ocurre **al cargar o recargar la aplicación, no en cada navegación interna** entre rutas. El incumplimiento relevante es que **una recarga dentro de la misma sesión cambia el orden** → no es estable por sesión. La grilla **no persiste sus datos en `localStorage`**.
- Grilla de **4 columnas** (`repeat(4, 1fr)`), filas de 250px; en móvil (≤768px) pasa a 1 columna.
- Imágenes con `filter: grayscale(...)` que **revelan color y escalan (`scale(1.05)`) al hover**.
- Las imágenes **no tienen `width`/`height`** → riesgo de *layout shift* (CLS).

## Páginas interiores

- Patrón repetido: contenedor `.container` (`max-width: 1200px`, texto **justificado**), tarjetas `.card`, media query única en 768px.
- Títulos `h1` en color ámbar `#f4ba00` con **`font-size: 2vw`** (escala con el viewport; problemático en pantallas muy anchas o muy angostas).
- `/perfil`: los párrafos arrancan con `opacity: 0` y se revelan con `setTimeout` **cada 2 s** (`showParagraphs`). Si el JS falla, el contenido **queda invisible**.
- `/programacion`: proyectos embebidos en `<iframe>` **sin `title`** (hay un `svelte-ignore a11y-missing-attribute`).
- El CSS de `.container`/`.card` está **duplicado casi idéntico en ~6 archivos**; no hay tokens ni sistema de estilos. `src/app.css` está casi vacío (solo color de fondo y texto del `body`).

## Tipografía

- La intención declarada en el CSS es **Poppins** (nav/títulos) + **Montserrat** (cuerpo).
- **Ninguna de las dos fuentes se carga**: no hay `<link>` a Google Fonts ni `@font-face` en `src/app.html` ni en ningún componente. El sitio se muestra con la **sans-serif del sistema**.

## Paleta

- Fondo: `#dbe6e6` (body) / `#e6ebf1` (contenedores).
- Texto: azul petróleo `#054f6d`.
- Acento principal: ámbar `#f4ba00` (títulos, hover de nav, iconos).
- Acentos secundarios: rojo `#EE4E4E` (autor), naranja `#E16526` (enlaces en publicaciones).

## Despliegue

- **Vercel** con `adapter-vercel`. Prerenderiza el **andamiaje estático** de cada ruta (nav, títulos, enlaces de retorno y el texto de `/perfil`), pero **no** las listas dinámicas de Sheets. El HTML servido de `/` (2816 bytes) trae el `<nav>` pero la grilla **vacía** (`<div class="container"><p></p></div>`); esa parte se carga por JS en el cliente. El HTML de `/perfil` (3979 bytes) **sí** trae el texto de perfil. Verificado por `curl`; ver `diagnostico-inicial.md` §4.
- En el repo hay además una carpeta **`docs/`** con un **build antiguo de GitHub Pages** commiteado (46 archivos versionados), independiente del despliegue actual de Vercel. La nueva documentación vive en `docs/renovacion-perfil/` y no forma parte de ese build.

## Código muerto / arrastre

- Restos de una arquitectura SPA anterior, **no referenciados** por las rutas activas: `src/routes/Perfil.svelte`, `Experiencia.svelte`, `Publicaciones.svelte`, `Programacion.svelte`, y `publicaciones/Articulos.svelte`, `Libros.svelte`, `Capitulos.svelte`, `programacion/Stardust.svelte`, `stardust.js`, `programacion/index.html`.
- Dependencias sin uso: `svelte-routing`, `svelte-spa-router`, `@sveltejs/adapter-static`.
- Placeholders del template: `src/lib/index.js`, `src/index.test.js`.

## Cambios sin confirmar en el árbol de trabajo (a la fecha de R00)

`git status` reporta ediciones **no commiteadas** que deben preservarse:

- `src/routes/perfil/+page.svelte` — el texto del perfil fue **reescrito** por una versión más extensa (historia intelectual de la ciencia, cultura visual, humanidades digitales, investigación-creación).
- `src/routes/Perfil.svelte` — el mismo texto nuevo aplicado al componente legado (aunque el componente no esté en uso).

Estos cambios de contenido **no forman parte de ninguna ronda de renovación** y no deben revertirse.

**Divergencia repositorio ↔ despliegue (verificada, R00-respuesta, H2):** estos cambios están **solo en el árbol de trabajo local** y **no confirmados** (`git status`: ` M`). El **despliegue vigente en Vercel sigue sirviendo la versión anterior del texto**: `curl https://juanfuc.vercel.app/perfil` devuelve el párrafo antiguo («Hola, soy Juan Felipe… Profesor universitario…») y **no** contiene el texto nuevo («mediaciones visuales», «tesis laureada»). Es decir, el repositorio local va por delante del despliegue en el contenido de `/perfil`. Cualquier agente debe tener presente que **lo que ve en el sitio publicado no coincide con el árbol de trabajo local** en esta ruta.

## Problemas vigentes (resumen)

- **SEO / prerender**: las **listas dinámicas** llegan por CSV en el cliente → no aparecen en el HTML prerenderizado. La **portada es el caso crítico** (grilla vacía); las rutas interiores prerenderizan su andamiaje estático y `/perfil` prerenderiza su texto (aunque oculto por CSS). El andamiaje estático se indexa; el contenido de Sheets no.
- **Metadatos**: sin `<title>`, sin `<meta description>`, sin Open Graph; `<html lang="en">` en un sitio en español.
- **Tipografía intencional no cargada** (Poppins/Montserrat).
- **Grilla no estable por sesión** (se rebaraja al recargar la app) y lógica de `isHome` frágil.
- **Accesibilidad**: iconos sociales sin texto accesible, `target="_blank"` sin `rel="noopener"`, iframes sin `title`, contraste dudoso del ámbar, texto justificado, nav sin semántica de lista ni estado activo.
- **Rendimiento**: se declara el agregador `d3` para usar solo `d3-fetch` (impacto en bundle **no medido**), imágenes sin dimensiones (CLS), iconos por red.
- **Caché** inconsistente entre secciones.
- **Repositorio**: código muerto, CSS duplicado, `docs/` viejo commiteado, dependencias sin usar.

Ver el desglose completo y el contraste repo/sitio en [`diagnostico-inicial.md`](./diagnostico-inicial.md).

# Estado actual de la implementación

Describe **cómo está construido el sitio hoy**, tal como existe en el repositorio y en el despliegue. Es una foto del punto de partida; se actualiza al cerrar cada ronda que cambie la implementación.

> Fecha de la foto: cierre de R01 (base estructural). El árbol local aún no tiene commit/push; los sitios públicos conservan el build anterior.

## Stack

- **SvelteKit 2.x** sobre **Svelte 4.2.7**, **Vite 5** (`package.json`).
- Adaptador activo: **`@sveltejs/adapter-static`** (`svelte.config.js`), con fallback `404.html`.
- Destino de producción preparado: **GitHub Pages** → https://juanfuc.github.io/, mediante `.github/workflows/deploy.yml`. El workflow aún no se ha ejecutado porque R01 no tiene commit/push. Vercel se conserva temporalmente como respaldo con el build anterior.
- **Prerender global activado**: `export const prerender = true` en `src/routes/+layout.js`.
- **Node: entorno observado** = Node 22 (la máquina donde se ejecutó R00). **El proyecto no fija una versión**: no hay campo `engines` en `package.json`, ni `.nvmrc`, ni `.node-version`. `.npmrc` solo activa `engine-strict=true` (que no fija versión por sí mismo).
- Iconos: `@iconify/svelte` (carga de iconos en runtime).
- Fetch de datos: el código importa **`d3-fetch`** (`import { csv } from 'd3-fetch'`); `package.json` declara el paquete **agregador `d3`** para obtener ese subpaquete. No se ha medido el bundle: no se afirma que «todo D3» llegue al cliente.

## Rutas (file-based routing)

| Ruta | Archivo | Contenido |
|------|---------|-----------|
| `/` | `src/routes/+page.svelte` + `src/routes/+layout.svelte` | La grilla dinámica vive exclusivamente en la página; el layout contiene la navegación compartida. |
| `/perfil` | `src/routes/perfil/+page.svelte` | Texto de perfil estático, visible desde el primer render y sin temporizadores. |
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
| `/` (grilla, `+page.svelte`) | Sí | **No** (los datos de la grilla no se guardan) | usa `sessionStorage` solo para `gridShuffleSeed`, no para datos |
| `/perfil` | **No** (estático) | **No** | — |
| `/experiencia` | Sí | Sí | `experienceData` (+ `experienceDataTimestamp`), **TTL 1 h** |
| `/publicaciones` | Sí | Sí | `publicationsData`, **sin expiración** |
| `/publicaciones/articulos` | Sí | Sí | `articlesData`, **sin expiración** |
| `/publicaciones/libros` | Sí | Sí | `booksData`, **sin expiración** |
| `/publicaciones/capitulos` | Sí | Sí | `chaptersData`, **sin expiración** |
| `/programacion` | Sí | Sí | `projectsData` (+ `projectsDataTimestamp`), **TTL 1 h** |

Caché **inconsistente**: experiencia y programación expiran a la hora; publicaciones/artículos/libros/capítulos **no expiran** (no se refrescan hasta limpiar `localStorage`); los datos de la portada no se cachean.

## Comportamiento de la grilla (portada)

- Definida exclusivamente en `src/routes/+page.svelte`; por estructura solo puede aparecer en `/`.
- Los datos se barajan con Fisher–Yates y un PRNG determinista (`mulberry32`). La semilla `gridShuffleSeed` se guarda en `sessionStorage`, por lo que el orden se conserva al navegar y recargar la misma pestaña; una sesión nueva puede producir otro orden.
- Se eliminaron `isHome`, `lastPage` y `handleNavigation`.
- Grilla de **4 columnas** (`repeat(4, 1fr)`), filas de 250px; en móvil (≤768px) pasa a 1 columna.
- Imágenes con `filter: grayscale(...)` que **revelan color y escalan (`scale(1.05)`) al hover**.
- Las imágenes **no tienen `width`/`height`** → riesgo de *layout shift* (CLS).

## Páginas interiores

- Patrón repetido: contenedor `.container` (`max-width: 1200px`, texto **justificado**), tarjetas `.card`, media query única en 768px.
- Títulos `h1` en color ámbar `#f4ba00` con **`font-size: 2vw`** (escala con el viewport; problemático en pantallas muy anchas o muy angostas).
- `/perfil`: el texto se prerenderiza y queda visible por defecto; se eliminaron `showParagraphs`, `setTimeout` y `opacity: 0`.
- `/programacion`: los `<iframe>` tienen `title` descriptivo.
- `src/lib/styles/structural.css` centraliza el contenedor interior y las tarjetas de lista compartidas. La tipografía, colores y variantes visuales permanecen locales.
- La navegación usa listas semánticas, `aria-current`, nombres accesibles para los iconos sociales y `rel="noopener noreferrer"` en enlaces externos. `src/app.css` continúa huérfano y pendiente de una decisión visual.

## Tipografía

- La intención declarada en el CSS es **Poppins** (nav/títulos) + **Montserrat** (cuerpo).
- **Ninguna de las dos fuentes se carga**: no hay `<link>` a Google Fonts ni `@font-face` en `src/app.html` ni en ningún componente. El sitio se muestra con la **sans-serif del sistema**.

## Paleta

- Fondo: `#dbe6e6` (body) / `#e6ebf1` (contenedores).
- Texto: azul petróleo `#054f6d`.
- Acento principal: ámbar `#f4ba00` (títulos, hover de nav, iconos).
- Acentos secundarios: rojo `#EE4E4E` (autor), naranja `#E16526` (enlaces en publicaciones).

## Despliegue

- El build local usa `adapter-static` y genera `build/` con los ocho HTML prerenderizados, `404.html`, `.nojekyll` y los assets de SvelteKit. Las listas de Sheets siguen cargándose en cliente.
- `.github/workflows/deploy.yml` ejecuta `npm ci` y `npm run build` con Node 20 en cada push a `main`, sube `build/` como artefacto y despliega con GitHub Pages. No se ha hecho commit, push ni despliegue; por eso `juanfuc.github.io` todavía sirve un build antiguo e inconsistente con este árbol, lo cual es esperado.
- Vercel sigue sirviendo el build anterior como respaldo temporal. El posible efecto futuro del adaptador estático sobre su integración de Git permanece pendiente antes del push.
- En el repo hay además una carpeta **`docs/`** con un **build antiguo de GitHub Pages** commiteado (46 archivos versionados), independiente del despliegue actual de Vercel. La nueva documentación vive en `docs/renovacion-perfil/` y no forma parte de ese build.

## Código muerto / arrastre

- R01 eliminó los diez archivos legados de la etapa SPA, además de `src/lib/index.js` y `src/index.test.js`, tras comprobar que no tenían referencias activas.
- R01 retiró `svelte-routing` y `svelte-spa-router`. `@sveltejs/adapter-static` pasó a ser el adaptador activo; `@sveltejs/adapter-vercel` se conserva temporalmente como dependencia de respaldo.
- Siguen pendientes de decisión el build antiguo versionado en `docs/`, `.vercel/output/` y el gitlink anidado `juanfuc.github/`.

## Árbol de trabajo al cierre de R01

El texto extenso de `/perfil` quedó incluido en el checkpoint `5e2e5fc` y R01 lo preservó íntegro. El componente legado `src/routes/Perfil.svelte`, que duplicaba ese texto y no era una ruta activa, fue eliminado junto con el resto del código muerto.

Los cambios de R01 permanecen sin commit ni push. La divergencia entre el árbol local y los sitios públicos es, por tanto, esperada hasta que el responsable autorice esas acciones.

## Problemas vigentes (resumen)

- **SEO / prerender**: las **listas dinámicas** llegan por CSV en el cliente → no aparecen en el HTML prerenderizado. La **portada es el caso crítico** (grilla vacía); las rutas interiores prerenderizan su andamiaje estático y `/perfil` prerenderiza su texto (aunque oculto por CSS). El andamiaje estático se indexa; el contenido de Sheets no.
- **Metadatos**: sin `<title>`, sin `<meta description>`, sin Open Graph; `<html lang="en">` en un sitio en español.
- **Tipografía intencional no cargada** (Poppins/Montserrat).
- **Accesibilidad pendiente**: contraste dudoso del ámbar, texto justificado y otras mejoras avanzadas. Los nombres accesibles, `rel`, títulos de iframe, semántica básica del nav y estado activo quedaron resueltos en R01.
- **Rendimiento**: se declara el agregador `d3` para usar solo `d3-fetch` (impacto en bundle **no medido**), imágenes sin dimensiones (CLS), iconos por red.
- **Caché** inconsistente entre secciones.
- **Repositorio**: `docs/` viejo, `.vercel/output/` y gitlink anidado pendientes; `src/app.css` huérfano. El código muerto y las dependencias de routing se eliminaron en R01.

Ver el desglose completo y el contraste repo/sitio en [`diagnostico-inicial.md`](./diagnostico-inicial.md).

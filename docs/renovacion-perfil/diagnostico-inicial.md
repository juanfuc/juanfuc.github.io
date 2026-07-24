# Diagnóstico inicial (auditoría de partida)

Auditoría técnica y visual que originó el proyecto, **contrastada con el código actual del repositorio y con el sitio publicado**.

Cada afirmación se marca con su nivel de evidencia:

- **[REPO]** — verificado leyendo el código del repositorio.
- **[LIVE]** — observado en el HTML servido por https://juanfuc.vercel.app/ (curl del shell, R00).
- **[INFERIDO]** — deducción razonada a partir de lo anterior, no observada directamente en render visual.

> Nota de método: en R00 **no se realizó una inspección visual con navegador** del sitio en vivo (no hubo render con capturas). Las marcas **[LIVE]** provienen del **HTML crudo** servido por Vercel. Todo juicio puramente estético sobre el render (p. ej. legibilidad percibida) es **[INFERIDO]** hasta que una ronda lo verifique con navegador.

---

## 1. Stack, estructura y despliegue

- **[REPO]** SvelteKit 2.x, Svelte 4.2.7, Vite 5 (`package.json`).
- **[REPO]** `adapter-vercel` en `svelte.config.js`; `prerender = true` en `src/routes/+layout.js`.
- **[REPO]** Las **listas dinámicas** se cargan por CSV de Google Sheets con `d3-fetch` en `onMount` (URLs en `+layout.svelte`, `experiencia/+page.svelte`, `publicaciones/**`, `programacion/+page.svelte`). El **cacheo en `localStorage` aplica solo a las páginas interiores de datos** (experiencia, publicaciones, artículos, libros, capítulos, programación); **la grilla de la portada NO se cachea en `localStorage`** (el layout solo persiste `lastPage`). El texto de `/perfil` **no usa fetch ni `localStorage`** (es estático). Ver mapa preciso en `estado-actual.md`.
- **[REPO]** `svelte-routing`, `svelte-spa-router` y `@sveltejs/adapter-static` figuran en dependencias pero **no se usan** (grep sin coincidencias en `src/` fuera de los componentes legados).
- **[REPO]** Código muerto de la etapa SPA: componentes capitalizados (`Perfil.svelte`, `Experiencia.svelte`, `Publicaciones.svelte`, `Programacion.svelte`, `publicaciones/Articulos|Libros|Capitulos.svelte`, `programacion/Stardust.svelte`, `stardust.js`, `programacion/index.html`). Solo `Publicaciones.svelte` importa a tres de ellos, y a su vez nadie lo importa.
- **[REPO]** `docs/` contiene un build antiguo de GitHub Pages commiteado (46 archivos versionados), ajeno al despliegue de Vercel.

## 2. Ejecución local

- **[REPO]** `vite dev` arranca sin errores y renderiza.
- **[INFERIDO]** No se hizo inspección visual con navegador en local; los hallazgos estéticos derivan del código y del HTML servido.

## 3. Tipografía intencional no cargada — hallazgo principal

- **[REPO]** El CSS declara Poppins (nav/títulos) y Montserrat (cuerpo) en casi todos los `<style>`.
- **[REPO]** No existe `<link>` a Google Fonts ni `@font-face` en `src/app.html` ni en ningún componente.
- **[LIVE]** El `<head>` servido por Vercel **no enlaza ninguna fuente**; solo hay `charset`, `icon` (`favicon2.png`), `viewport` y los CSS/JS de la app.
- **Conclusión:** la tipografía diseñada **no se está entregando**; se usa la sans-serif del sistema. (Verificado en repo y en live.)

## 4. SEO, prerender y contenido en cliente — hallazgo crítico

> Corregido en R00-respuesta (H1): la afirmación original («cada página carga su contenido en `onMount`») era demasiado amplia. Hay que **distinguir el contenido estático prerenderizado del contenido dinámico de Sheets**.

- **[REPO]** `prerender = true`. El HTML de build **sí** incluye el **contenido estático** de cada ruta: el `<nav>`, los títulos de sección, los enlaces de retorno y —en `/perfil`— el **texto completo del perfil** (que es estático, no proviene de Sheets).
- **[REPO]** Lo que **no** aparece en el HTML de build son las **listas dinámicas** que se cargan en `onMount` desde CSV (grilla de portada, experiencia, publicaciones/artículos/libros/capítulos, programación). Esas secciones quedan vacías hasta que actúa el JavaScript en el cliente.
- **[LIVE]** El HTML de `/` servido por Vercel pesa **2816 bytes**. Contiene el `<nav>` (prerenderizado), pero la grilla llega **vacía**: `<div class="container"><p></p></div>`. No hay imágenes ni texto de publicaciones en el HTML inicial → **la portada es el caso más crítico**.
- **[LIVE]** En cambio, el HTML de `/perfil` servido por Vercel pesa **3979 bytes** e **incluye el párrafo de perfil completo** dentro del `<p>` (verificado por `curl` en R00-respuesta). El texto está presente en el HTML, aunque el CSS lo deje **visualmente oculto** (`opacity: 0`) hasta que el `setTimeout` lo revele (ver `estado-actual.md` §Páginas interiores).
- **[LIVE]** Los enlaces de iconos sociales existen como `<a target="_blank">` **sin contenido** (el `<Icon>` se renderiza en cliente).
- **Conclusión:** para buscadores y previsualizaciones de enlace, **la portada está prácticamente vacía** (grilla dinámica); las rutas interiores **sí** prerenderizan su andamiaje estático, y `/perfil` prerenderiza además su texto (aunque oculto por CSS). El problema de indexabilidad se concentra en el **contenido dinámico de Sheets**, no en todo el sitio.

## 5. Metadatos

- **[REPO]** `src/app.html`: sin `<title>`, sin `<meta name="description">`, sin Open Graph/Twitter; `<html lang="en">`.
- **[LIVE]** Confirmado en el HTML servido: `<html lang="en">`, **sin `<title>`, sin description, sin OG**. El sitio está en español → `lang` incorrecto.

## 6. Identidad visual

- **[REPO]** Paleta: fondo `#dbe6e6`/`#e6ebf1`, texto `#054f6d`, acento ámbar `#f4ba00`, secundarios `#EE4E4E` y `#E16526`.
- **[REPO]** Gesto distintivo: imágenes en `grayscale` que revelan color y escalan (`scale(1.05)`) al hover (`+layout.svelte`, `publicaciones/+page.svelte`).
- **[REPO]** Grilla de portada de 4 columnas; publicaciones en 3 columnas.
- **[INFERIDO]** La percepción cromática y el efecto hover en el render se asumen a partir del CSS; no verificados con navegador en R00.

## 7. Grilla y estado de portada

> Corregido en R00-respuesta (H3): precisado el momento del rebarajado y aclarado que la grilla no se cachea en `localStorage`.

- **[REPO]** El layout raíz (`+layout.svelte`) carga y **baraja (`shuffleArray`) los datos en `onMount`**. En SvelteKit el layout raíz **persiste durante la navegación interna** y no vuelve a ejecutar `onMount`, por lo que **el rebarajado ocurre al cargar o recargar la aplicación, no en cada navegación interna** entre rutas. El incumplimiento relevante es que **una recarga dentro de la misma sesión cambia el orden** (no es estable por sesión).
- **[REPO]** La grilla **no persiste sus datos en `localStorage`**: el layout solo usa `localStorage` para la bandera `lastPage`. (El cacheo en `localStorage` es de las páginas interiores de datos; ver §1 y `estado-actual.md`.)
- **[REPO]** `isHome` se resuelve con `localStorage('lastPage')` en `onMount` y se actualiza por clic, no por la ruta real → lógica frágil.
- **[LIVE]** En el HTML servido, `isHome` no está resuelto todavía y la grilla llega vacía; el estado se decide en cliente.

## 8. Accesibilidad

- **[REPO]** Iconos sociales sin `aria-label` ni texto (`+layout.svelte`); **[LIVE]** confirmado: `<a>` de iconos sin contenido accesible en el HTML.
- **[REPO]** `target="_blank"` sin `rel="noopener noreferrer"` en enlaces externos.
- **[REPO]** `<iframe>` sin `title` en `/programacion` (con `svelte-ignore a11y-missing-attribute`).
- **[REPO]** Nav sin marcado de lista ni indicación de página activa; texto justificado.
- **[INFERIDO]** Contraste del ámbar `#f4ba00` sobre `#e6ebf1`: probable incumplimiento de WCAG AA para texto (no medido con herramienta en R00).

## 9. Rendimiento y estabilidad de layout

- **[REPO]** Imágenes remotas sin `width`/`height` → *layout shift* (CLS).
- **[REPO]** El código importa **`d3-fetch`** (`import { csv } from 'd3-fetch'`), pero `package.json` declara el **paquete agregador `d3`** para disponer de ese subpaquete. Sería sustituible por `fetch` nativo + un parser CSV mínimo. **[INFERIDO]** *No se ha medido el bundle*: no puede afirmarse que «todo D3» se entregue al cliente sin analizar el empaquetado; el bundler puede aplicar *tree-shaking*. (Corregido en R00-respuesta, H7.)
- **[REPO]** Iconos cargados por red en runtime vía Iconify.

## 10. Caché de datos

- **[REPO]** Experiencia y programación: TTL de 1 h. Publicaciones/artículos/libros/capítulos: **sin expiración** → el visitante recurrente no ve cambios hasta limpiar `localStorage`.

## 11. Organización del código

- **[REPO]** CSS de `.container`/`.card` duplicado casi idéntico en ~6 archivos; `src/app.css` casi vacío; sin tokens ni sistema de estilos.
- **[REPO]** Código muerto y dependencias sin usar (ver §1).

---

## Qué conservar / qué envejeció / qué está roto

**Conservar** — paleta, intención tipográfica (cargándola de verdad), efecto gris→color, arquitectura de contenido, fuente de datos vía Sheets si se quiere seguir editando sin tocar código.

**Envejecido / poco resuelto** — `font-size: 2vw` en títulos, texto justificado, CSS duplicado, breakpoint único, home aleatorio en cada carga, animación de perfil de 2 s por párrafo.

**Roto / problemas reales** — fuentes no cargadas, contenido dinámico de Sheets no prerenderizado (SEO; la portada es el caso crítico, las interiores prerenderizan su andamiaje estático), metadatos ausentes y `lang` incorrecto, accesibilidad (aria/rel/iframe/contraste), CLS por imágenes, caché inconsistente.

## Prioridad de partida (referencia, no vinculante para las rondas)

- **P0** — cargar fuentes; metadatos base y `lang="es"`; prerenderizar contenido; unificar caché.
- **P1** — accesibilidad (aria-label, rel, iframe title, contraste, foco, `clamp()` en títulos, quitar justify); repensar home aleatorio y animación de perfil.
- **P2** — tokens + componentes reutilizables; borrar código muerto; sacar `docs/` viejo del control de versiones; quitar dependencias sin usar; `width/height` en imágenes; sitemap/robots.
- **P3** — JSON-LD `Person`/`ScholarlyArticle`; estados de carga/error/vacío.

> Estas prioridades son el diagnóstico de partida. La secuencia real de trabajo la fijan las **rondas** y `decisiones.md`, que reordenan según los acuerdos del `brief.md` (perfil primero, grilla estable en portada, etc.).

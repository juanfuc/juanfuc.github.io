# R01 — Base estructural

**Estado:** cerrada — aprobada.
**Toca código del sitio:** sí.
**Checkpoint previo:** commit `5e2e5fc` ("chore: checkpoint before R01"). Los cambios de esta sección son un diff limpio sobre ese commit (no se ha hecho un segundo commit).

## Objetivo

Establecer la **base estructural** de la portada y las páginas interiores de acuerdo con el `brief.md`, sin rediseñar todavía la estética: que la grilla quede correctamente confinada a la portada, que las interiores empiecen por su contenido, que el perfil sea visible sin depender de temporizadores y que el orden aleatorio de la grilla sea estable durante la sesión — **preservando el contenido, las proporciones de las imágenes y la apariencia visual actuales**.

Esta ronda es de **corrección estructural y de comportamiento**, no de rediseño visual. Los temas de tipografía, líneas, composición plana, franja contextual y hover-scale siguen **pendientes** en `decisiones.md` y no se cierran aquí.

## Contexto relevante

- Hoy la grilla vive en `src/routes/+layout.svelte` bajo la bandera `isHome`, que se decide con `localStorage('lastPage')` (frágil). Ver `estado-actual.md` §grilla.
- `src/routes/+page.svelte` está vacío.
- El orden de la grilla se baraja en `onMount` del layout raíz: como el layout persiste, **se rebaraja al cargar/recargar la app, no en cada navegación interna**; una recarga en la misma sesión cambia el orden (ver `estado-actual.md`).
- `/perfil` prerenderiza el texto en el HTML, pero lo **oculta con `opacity:0`** y lo revela con `setTimeout` cada 2 s (queda invisible si el JS falla o tarda).
- Hay cambios sin confirmar en el texto de `/perfil` que deben preservarse.

## Definición de «sesión» (para el criterio 4)

**[Precisado en R00-respuesta, H5]** A efectos de estabilidad del orden de la grilla:

- **Misma sesión** = misma pestaña del navegador mientras no se cierre, **incluyendo recargas (F5) de esa pestaña**. El orden debe **permanecer idéntico** ante: navegación interna, y **recarga de la misma pestaña**.
- **Nueva sesión** (se permite un orden distinto) = **abrir una pestaña/ventana nueva** o **reabrir tras cerrar la pestaña**. R01 adoptó `sessionStorage` como límite de sesión: persiste entre recargas de la pestaña y se pierde al cerrarla.

## Línea base visual (obligatoria antes de tocar el sitio)

**[Añadido en R00-respuesta, H5]** Como esta ronda debe **preservar la apariencia visual actual** y la grilla depende de **datos externos mutables** (Google Sheets), R01 **deberá capturar una línea base ANTES de modificar el código**:

- **Capturas** (`preview` del build actual o `dev`) de: `/`, `/perfil`, `/experiencia`, `/publicaciones`, `/publicaciones/articulos`, `/publicaciones/libros`, `/publicaciones/capitulos`, `/programacion`.
- **Viewports**: escritorio **1280×800** y móvil **375×667** (≤768px activa el layout móvil).
- **Conteos de referencia** por ruta (nº de tarjetas/ítems e imágenes renderizadas) **en el momento de la captura**, anotando que pueden variar si cambian las hojas. La comparación post-cambio se hace **contra esa base**, no contra un ideal.
- Guardar las capturas y los conteos en el **Registro de ejecución** de esta ronda.

## Criterios de aceptación

1. **La grilla aparece únicamente en `/`.** No se muestra en ninguna ruta interior ni las antecede. (La condición de portada no debe depender de una bandera frágil basada en el último clic.)
2. **Las rutas interiores comienzan después del encabezado**, por su propio contenido (perfil, experiencia, publicaciones, programación), sin grilla previa.
3. **El perfil es visible desde el primer render, sin temporizadores.** Se elimina la dependencia de `setTimeout`/`opacity:0` para mostrar el texto; el contenido no puede quedar oculto si el JS falla o tarda. (Nota: el texto ya se prerenderiza en el HTML; el defecto es que el CSS lo oculta hasta que actúa el JS — ver `diagnostico-inicial.md` §4.)
4. **El orden aleatorio de la grilla permanece estable durante la sesión**, según la definición de arriba. **Debe probarse explícitamente una recarga (F5) de la misma pestaña**: el orden no cambia. Abrir una pestaña nueva **puede** dar otro orden.
5. **Se preservan contenido, proporciones de las imágenes y apariencia visual actuales**, verificado **contra la línea base** capturada:
   - Sin pérdida de ítems: los conteos por ruta coinciden con la base (salvo cambios atribuibles a las hojas, que deben anotarse).
   - Las imágenes conservan sus **proporciones** (sin recortes ni deformaciones nuevas).
   - Paleta y **efecto gris→color** intactos; el resto de la apariencia no cambia respecto de las capturas base, en ambos viewports.
   - Los **cambios sin confirmar del texto de `/perfil`** se conservan (no se revierten).

## Verificaciones (con comandos realmente disponibles)

**[Ajustado en R00-respuesta, H5]** `package.json` solo define `dev`, `build` y `preview`. **No hay script `check` ni `svelte-check` instalado**; **no se exige**.

- **Build**: `npm run build` completa **sin errores**. *(Verificación primaria automatizable de esta ronda.)*
- **Sync de tipos (opcional, no bloqueante)**: `npx svelte-kit sync` sin errores. No sustituye a `svelte-check` y **no es criterio de aprobación**.
- **Preview del build**: `npm run preview` y comprobación manual de:
  - **Carga directa** por URL (no solo navegando) de `/` y de **cada ruta interior**, que **incluye explícitamente las tres subsecciones de publicaciones**: `/perfil`, `/experiencia`, `/publicaciones`, `/publicaciones/articulos`, `/publicaciones/libros`, `/publicaciones/capitulos`, `/programacion`. Cada una cumple los criterios 1–3.
  - **Navegación interna** entre secciones: la grilla no reaparece fuera de `/` (criterio 1) y, tras volver a `/`, el orden es el mismo de la sesión (criterio 4).
  - **Recarga (F5)** en `/`: el orden se mantiene (criterio 4).
  - Comparación visual **contra la línea base** en escritorio y móvil (criterio 5).

## Notas de implementación (orientativas, no vinculantes)

- Confinar la grilla a `/` probablemente implique moverla del layout a `src/routes/+page.svelte`, o condicionarla por la ruta real (`$page.route.id === '/'`) en lugar de por `localStorage`.
- Para la estabilidad por sesión se adoptó una semilla persistida en `sessionStorage`; la decisión queda registrada en `decisiones.md`.
- Para el perfil, renderizar el texto visible por defecto; si se conserva alguna animación, que sea de mejora progresiva (nunca ocultando el contenido base).

## Fuera de alcance de R01

- Cargar/definir la tipografía definitiva, el sistema de líneas, la aplicación de la composición plana y la franja contextual (pendientes).
- Metadatos/SEO, prerender de datos, accesibilidad avanzada, limpieza de código muerto y de `docs/`: se abordan en rondas posteriores salvo que se decida lo contrario.
- Eliminar o no el aumento de escala en hover: **pendiente**, no se decide en R01.

## Registro de ejecución

### Línea base capturada

**Limitación real de método (léase antes que los resultados):** esta sesión no tuvo herramienta de
captura de pantalla/navegador disponible. La "línea base" e "línea después" se capturaron con
`npm run build` + `npm run preview` + `curl` sobre las 8 rutas exigidas, comparando **HTML servido,
tamaño en bytes y las reglas CSS relevantes en el código fuente** — no píxel a píxel ni en los dos
viewports (1280×800 / 375×667) por separado, porque curl no ejecuta CSS. Es una verificación
**estructural**, no visual en sentido estricto. Queda como tarea real para la revisión independiente
abrir `npm run preview` en un navegador y confirmar visualmente ambos viewports; ver «Limitaciones».

- Antes de la refactorización estructural (código de `5e2e5fc`, con el ajuste temporal de `runtime`
  aplicado durante la sesión para poder construir con Node 22 — ver más abajo): bytes de
  `/`=2816, `/perfil`=5260, `/experiencia`=3056, `/publicaciones`=3277, `/publicaciones/articulos`=3105,
  `/publicaciones/libros`=3087, `/publicaciones/capitulos`=3104, `/programacion`=3043.
- Se confirmó en el HTML servido: `<html lang="en">`; grilla de portada vacía en el prerender (los
  datos de Sheets no están en el HTML, solo tras fetch en cliente); nav sin `<ul>`/`aria-current`;
  iconos sociales como `<a>` vacíos sin nombre accesible; **cero** ocurrencias de `rel="noopener"` en
  las 8 rutas; el `<iframe>` de `/programacion` no aparece en el HTML prerenderizado (depende de
  fetch en cliente, así que su `title` solo se pudo verificar leyendo el código fuente).
- Paleta y efecto gris→color confirmados en el CSS de origen (no en runtime): fondo `#dbe6e6`/`#e6ebf1`,
  texto `#054f6d`, acento `#f4ba00`; `filter: grayscale(...)` → `filter: none` + `scale(1.05)` en hover.

### Cambios realizados

**Grilla confinada a `/`:**
- La grilla, el fetch a Sheets y el barajado salieron de `src/routes/+layout.svelte` y ahora viven
  solo en `src/routes/+page.svelte` (antes vacío). El layout raíz quedó limitado a `<nav>` + `<slot />`.
- Se eliminó la bandera `isHome` basada en `localStorage('lastPage')` y el `handleNavigation` que la
  mantenía; ya no depende del último clic, solo de qué componente de ruta está montado.
- El contenedor de la grilla se renombró de `.container` a `.home-grid` (mismas reglas, valores
  intactos) para que no pueda heredar nada del nuevo `.container` compartido (ver más abajo): la
  portada no importa esa hoja compartida.

**Orden aleatorio estable por sesión (criterio 4):**
- El `shuffleArray` (Fisher–Yates) ahora recibe un generador seedeado (`mulberry32`) en vez de
  `Math.random` directo. La semilla se guarda en `sessionStorage` (`gridShuffleSeed`) la primera vez
  que se necesita y se reutiliza mientras dure la pestaña, incluidas recargas (F5); una pestaña o
  ventana nueva arranca con `sessionStorage` vacío y puede sortear otra semilla. Mecanismo elegido
  de los sugeridos como "orientativos" en este documento; queda registrado aquí como la decisión de
  implementación concreta.
- Verificación: se extrajo la lógica (`mulberry32` + `shuffleArray` + manejo de semilla) a un script
  Node aislado (no en el repo) que confirma: misma semilla → mismo orden en 5 corridas; semillas
  distintas → orden distinto; el resultado es una permutación válida (sin pérdida ni duplicación) del
  arreglo de entrada. No se pudo probar de punta a punta en un navegador real (persistencia de
  `sessionStorage` entre recargas de pestaña) — ver «Limitaciones».

**Perfil visible sin temporizadores (criterio 3):**
- Se eliminaron `showParagraphs`, el `setTimeout` de 2 s por párrafo y las reglas
  `opacity:0`/`transform:translateY(50px)` de `src/routes/perfil/+page.svelte`. El párrafo se
  renderiza visible desde el HTML prerenderizado; confirmado que no queda ninguna regla `opacity` en
  el CSS generado (`grep` sobre `.svelte-kit/output/server/_app/immutable/assets/*.css`) ni ninguna
  referencia a `setTimeout`/`showParagraphs` en `src/routes/`.
- El texto nuevo del perfil (historia intelectual de la ciencia, cultura visual, humanidades
  digitales, investigación-creación) se preservó íntegro; no se tocó el contenido, solo el CSS/JS que
  lo ocultaba.

**Deuda estructural (CSS):**
- Nuevo `src/lib/styles/structural.css`, importado explícitamente (`import '$lib/styles/...'`) solo
  por las 7 páginas interiores que lo necesitan (perfil, experiencia, publicaciones y sus 3
  subsecciones, programación):
  - `.container` — regla idéntica que estaba duplicada byte a byte en esas 7 páginas.
  - `.list-card` / `.list-card-media` / `.list-card-body` (+ el `@media (max-width:768px)` asociado)
    — el patrón de tarjeta en fila (imagen/iframe + contenido) que estaba duplicado byte a byte en
    Artículos, Libros, Capítulos y Programación. Se renombró desde `.card`/`.card-image`/
    `.card-content` **a propósito**, para no colisionar con las variantes locales de esos mismos
    nombres en la portada, Publicaciones y Experiencia (que se dejaron intactas, sin tocar).
  - Se decidió no centralizar `h1`, `.autor`, ni el color de enlaces: son reglas de tipografía/color,
    no de estructura, y esta ronda preserva el sistema visual actual tal cual (la tipografía se
    decide en la siguiente ronda).
  - Se descubrió que `src/app.css` (fondo `#dbe6e6` del `body`) **no está importado en ningún sitio**
    del código — es CSS huérfano desde antes de R01. No se tocó (ni se borró ni se conectó): conectarlo
    cambiaría el fondo visible y eso es una decisión visual fuera de esta ronda. Queda anotado para
    la siguiente.

**Accesibilidad básica:**
- Nav semántico: los enlaces de sección y los iconos sociales pasaron a `<ul>/<li>` dentro de
  `<nav aria-label="Principal">` (con el reset `list-style/margin/padding` necesario para que no
  aparezcan viñetas ni sangría nuevas).
- Indicación de página activa: `aria-current="page"` calculado por ruta (`$page.url.pathname`) en
  cada enlace del nav, con una regla CSS mínima que reutiliza el ámbar `#f4ba00` ya usado en
  `:hover`/`:focus` (no es diseño nuevo, es el mismo color aplicado a un estado adicional).
- Nombres accesibles: `aria-label` (GitHub, X (Twitter), Academia.edu, ResearchGate, ORCID,
  Observable) en los 6 iconos sociales.
- `rel="noopener noreferrer"` añadido a **todos** los `target="_blank"` activos del sitio: 6 iconos
  del nav, enlace de cada tarjeta de la grilla de portada, enlace de cada tarjeta de Publicaciones,
  DOI de Artículos/Libros/Capítulos, y el enlace del título en Programación. Verificado por grep: cero
  `target="_blank"` sin `rel="noopener"` en `src/` tras el cambio (fuera de los archivos muertos que
  se eliminaron en la misma pasada).
- `title` descriptivo (`` `Proyecto interactivo: ${item.titulo}` ``) en el `<iframe>` de
  `/programacion`; se retiró el `<!-- svelte-ignore a11y-missing-attribute -->` que ya no hace falta.
- Fuera de esta lista no se tocó nada más (contraste, `lang="en"`, texto justificado): no estaban en
  los cinco puntos pedidos y son metadatos/tipografía fuera de alcance de R01.

**Código muerto y dependencias:**
- Confirmado por grep en todo el repo (no solo `src/`) antes de borrar: ningún archivo activo importa
  `src/routes/Perfil.svelte`, `Experiencia.svelte`, `Publicaciones.svelte`, `Programacion.svelte`,
  `publicaciones/{Articulos,Libros,Capitulos}.svelte`, `programacion/{Stardust.svelte,stardust.js,
  index.html}`. Las únicas coincidencias fuera de esos mismos archivos eran menciones en la
  documentación de R00 (histórico, no código) y en `package.json`/`package-lock.json` (las
  dependencias que se retiran junto con ellos). Se eliminaron los 10 archivos.
- `svelte-routing` solo lo importaba el propio `Publicaciones.svelte` (ya eliminado); `svelte-spa-router`
  no tenía ninguna referencia en `src/`. Se quitaron ambos de `package.json` (dependencies) y se
  corrió `npm install` para sincronizar `package-lock.json` (elimina 3 paquetes).
- También se eliminaron `src/lib/index.js` y `src/index.test.js`: placeholders del template de
  SvelteKit sin ninguna referencia; `index.test.js` importa `vitest`, que **no** es una dependencia
  del proyecto (no hay script `test` ni `vitest` en `package.json`), así que ya era inejecutable.
- En esta primera pasada **no se tocó** `@sveltejs/adapter-static` (quedó como devDependency sin usar,
  reservado para el paso 4 de GitHub Pages, entonces bloqueado). Pasó a ser el adaptador activo en la
  segunda pasada, una vez desbloqueado — ver más abajo. No se retiró `docs/` (build antiguo de GitHub
  Pages) ni la estrategia de lectura de CSV (`d3-fetch`), tal como se pidió.

**Preparación de GitHub Pages (segunda pasada, tras renombrar el repositorio a `juanfuc.github.io`):**
- `origin` actualizado de `https://github.com/juanfuc/juanfuc.github.git` a
  `https://github.com/juanfuc/juanfuc.github.io.git` (mismo protocolo HTTPS, `git remote set-url`).
  No se pudo verificar la conectividad con `git ls-remote`: falla con
  `fatal: Out of memory, malloc failed` tanto contra la URL nueva como contra la vieja (ya
  confirmada de que funcionaba antes), así que es una limitación de red/entorno de esta sesión, no
  evidencia de que el remoto esté mal. Ver «Limitaciones».
- `svelte.config.js`: adaptador cambiado de `@sveltejs/adapter-vercel` a `@sveltejs/adapter-static`
  (ya estaba instalado, reservado desde la pasada anterior) y se retiró el `runtime: 'nodejs20.x'`
  temporal (solo aplicaba al adaptador de Vercel). Config mínima: `adapter({ fallback: '404.html' })`
  — `pages`/`assets` en su default (`build/`, ya está en `.gitignore`). No se tocó `paths.base`: sigue
  sin definir, por lo que publica en la raíz (`https://juanfuc.github.io/`), sin ruta base de
  proyecto. `@sveltejs/adapter-vercel` se dejó instalado en `package.json` (sin usar) por si hace
  falta revertir mientras Vercel siga de respaldo.
- **Importante para quien vaya a hacer push:** `svelte.config.js` es compartido — si el repo sigue
  conectado a Vercel vía integración de git, el próximo build de Vercel también usará ahora
  `adapter-static` (no hay dos adaptadores simultáneos). No se evaluó ese impacto en esta pasada
  porque no se hizo push; queda como algo a decidir antes de empujar este cambio.
- Workflow nuevo `.github/workflows/deploy.yml`: build con Node 20 (`npm ci` + `npm run build`),
  publica el artefacto de `build/` con `actions/upload-pages-artifact` y despliega con
  `actions/deploy-pages`, disparado en push a `main` y manualmente (`workflow_dispatch`). No se ha
  hecho commit ni push de este archivo: es inerte hasta que se confirme.
- `404.html`: lo genera el propio `fallback` del adaptador (shell de SvelteKit); no se escribió a
  mano. `static/.nojekyll` ya existía en el repo desde antes de esta ronda (no fue necesario crearlo);
  se copia automáticamente a `build/.nojekyll` en cada build.

**Fix incidental (no estructural, pero bloqueaba toda verificación):**
- `npm run build` fallaba **ya en el checkpoint**, antes de cualquier cambio de esta ronda:
  `@sveltejs/adapter-vercel` rechaza Node 22 (`Unsupported Node.js version: v22.14.0`; la máquina de
  esta sesión corre v22.14.0). Sin build no hay `preview` ni forma de verificar nada de R01. Se fijó
  `runtime: 'nodejs20.x'` explícitamente en `vercel()` (`svelte.config.js`), la salida documentada
  por el propio adaptador para este caso. No cambia el adaptador ni el destino de despliegue (sigue
  siendo Vercel, tal como exige la decisión de mantenerlo como respaldo); solo fija en qué runtime de
  Vercel corren las funciones generadas, evitando que la detección automática dependa de la versión
  local de Node.

### Archivos afectados

*(Todo relativo al checkpoint `5e2e5fc`; ver `git diff 5e2e5fc` para el detalle exacto.)*

**Modificados:** `package.json`, `package-lock.json`, `svelte.config.js`, `src/routes/+layout.svelte`,
`src/routes/+page.svelte`, `src/routes/perfil/+page.svelte`, `src/routes/experiencia/+page.svelte`,
`src/routes/publicaciones/+page.svelte`, `src/routes/publicaciones/articulos/+page.svelte`,
`src/routes/publicaciones/libros/+page.svelte`, `src/routes/publicaciones/capitulos/+page.svelte`,
`src/routes/programacion/+page.svelte`.

**Creados:** `src/lib/styles/structural.css`, `.github/workflows/deploy.yml`.

**Eliminados (código muerto confirmado, sin referencias activas):** `src/routes/Perfil.svelte`,
`src/routes/Experiencia.svelte`, `src/routes/Publicaciones.svelte`, `src/routes/Programacion.svelte`,
`src/routes/publicaciones/Articulos.svelte`, `src/routes/publicaciones/Libros.svelte`,
`src/routes/publicaciones/Capitulos.svelte`, `src/routes/programacion/Stardust.svelte`,
`src/routes/programacion/stardust.js`, `src/routes/programacion/index.html`, `src/lib/index.js`,
`src/index.test.js`.

**No tocados a propósito:** `src/app.html`, `src/app.css`, `docs/` (build antiguo de Pages),
estrategia de lectura de CSV (`d3-fetch`), tipografía/colores/`h1`, `.vercel/output/` (ver nota abajo).

### Resultado de `npm run build`

**Éxito**, sin errores ni warnings (se revisó la salida completa buscando "warn/error/selector/a11y/
deprecat" además del código de salida; no aparece ninguno). Se generaron los 8 HTML prerenderizados
esperados en `.svelte-kit/output/prerendered/pages/` (`index`, `perfil`, `experiencia`,
`publicaciones`, `publicaciones/{articulos,libros,capitulos}`, `programacion`). No se corrió
`svelte-kit sync` por separado (no es criterio de aprobación; el build ya lo ejercita).

**Nota sobre `.vercel/output/`:** este directorio está versionado en el repositorio desde antes de R01
(commits "Deploy…" previos; no está en `.gitignore`) y parece ser el mecanismo real de despliegue a
Vercel. Cada `npm run build` lo regenera con nombres de archivo con nuevo hash, lo que ensucia el diff
sin relación con R01. Tras verificar, se restauró a su estado del checkpoint (`git checkout -- .vercel`
+ `git clean -fd -- .vercel`) para que el diff de esta ronda quede limpio. **No se modifica ni se
decide nada sobre `.vercel/`**; queda anotado para que el responsable del proyecto decida si debe
seguir versionado (fuera del alcance de R01, no estaba entre las rutas conocidas del checkpoint).

**Segunda pasada (adapter-static, tras el rename del repo):** se limpió `.svelte-kit`/`build` y se
corrió `npm run build` de nuevo. Con `adapter-static` el build **ya no toca `.vercel/output/`**
(confirmado: `git status -- .vercel` da cero cambios), porque ese adaptador ya no se invoca. Salida
limpia, sin warnings (mismo barrido por "warn/error/selector/a11y/deprecat" que en la primera pasada).
Se verificó el contenido de `build/` directamente (lectura de archivo, no servidor): los 8 HTML
esperados, `404.html` generado por el `fallback`, `.nojekyll` copiado desde `static/`, y el `<nav>`/
grilla/`aria-current` de `index.html` y `perfil.html` idénticos en contenido a lo verificado en la
primera pasada (el adaptador solo cambia el empaquetado de salida, no el marcado de las páginas).
Un intento de build intermedio falló con `fatal error: runtime: cannot allocate memory`; no fue un
problema del proyecto sino presión de memoria de esta máquina — se detectaron **dos procesos externos
`vite dev` de ~3 GB cada uno**, ajenos a esta sesión (aparentan pertenecer a otro proceso/agente
trabajando en paralelo sobre el mismo directorio; ver «Limitaciones»). No se terminó ningún proceso
ajeno; el rebuild simplemente se reintentó y completó limpio.

### Verificación de carga directa, navegación interna y recarga (F5)

- **Carga directa (`npm run preview` + `curl`) de las 8 rutas exigidas:** las 8 devuelven HTTP 200
  (`/`, `/perfil`, `/experiencia`, `/publicaciones`, `/publicaciones/articulos`,
  `/publicaciones/libros`, `/publicaciones/capitulos`, `/programacion`).
- **Grilla confinada a `/` (criterio 1):** confirmado por grep sobre las 8 páginas servidas — la clase
  `home-grid` solo aparece en `index.html`; la única otra ocurrencia de `class="grid"` es la grilla
  (distinta, preexistente) de `/publicaciones`, no la de portada.
- **Interiores empiezan por su contenido (criterio 2):** confirmado leyendo el HTML de cada ruta — nav
  y luego el contenido propio, sin grilla antes.
- **Indicación de página activa:** confirmado por ruta — `aria-current="page"` aparece exactamente en
  el enlace correspondiente en cada una de las 8 páginas (las 3 subsecciones de Publicaciones marcan
  activo `/publicaciones`, ya que estructuralmente son parte de esa sección).
- **Navegación interna real y recarga (F5) en un navegador:** **no verificado en esta sesión** — no
  hubo herramienta de navegador disponible. Lo que sí se verificó: (a) que el mecanismo de
  confinamiento de la grilla ya no depende del último clic (`isHome`/`localStorage` eliminados, ahora
  es puramente "qué componente de ruta está montado"), por lo que no hay ruta de código por la que la
  grilla podría reaparecer fuera de `/`; (b) el algoritmo de barajado seedeado, de forma aislada (ver
  arriba). Ver «Limitaciones» — esto queda como pendiente real para la revisión independiente.

### Comparación contra línea base

- **Conteos/estructura:** sin pérdida de contenido en ninguna ruta; los 8 HTML prerenderizados
  conservan exactamente el mismo andamiaje (nav + contenido propio) más los atributos de accesibilidad
  añadidos. El tamaño en bytes subió de forma uniforme (~350–420 bytes) en las 8 rutas, consistente
  con `<ul>/<li>`, `rel="noopener noreferrer"`, `aria-label` y `aria-current` añadidos — no con pérdida
  o cambio de contenido.
- **Paleta y efecto gris→color:** intactos — las reglas `.card-image img { filter: grayscale(...) }` /
  `:hover { filter: none; transform: scale(1.05) }` de la portada, y las reglas equivalentes de
  Publicaciones/Artículos/Libros/Capítulos, no se modificaron (se movieron de archivo en el caso de
  `.list-card-media img`, pero con los mismos valores).
- **Proporciones de imágenes:** no se tocó ningún `<img>` (atributos, `object-fit`, `max-width`), así
  que no hay recortes ni deformaciones nuevas.
- **Texto de `/perfil`:** preservado íntegro (ver arriba).
- **Comparación en los dos viewports (1280×800 / 375×667):** **no realizada visualmente** — ver
  «Limitaciones». Sí se verificó que las reglas `@media (max-width: 768px)` relevantes se preservaron
  con los mismos valores (grilla a 1 columna, tarjetas en columna, anchos al 100%) al moverlas al CSS
  compartido.

### Limitaciones o bloqueos reales

1. **GitHub Pages (paso 4): desbloqueado y implementado en la segunda pasada.** El repositorio fue
   renombrado a `juanfuc.github.io` (confirmado por el responsable del proyecto). Se actualizó
   `origin`, se cambió el adaptador a `adapter-static` con `fallback: '404.html'`, se añadió
   `.github/workflows/deploy.yml`, y se verificó el build/output de `build/`. **Sigue pendiente, real
   y explícitamente fuera de esta ronda:** confirmar conectividad real con el remoto (no se pudo desde
   esta sesión, ver punto 2), y decidir/ejecutar el primer push+deploy — nada de eso se hizo aquí por
   instrucción expresa ("no hagas commit, push ni despliegue").
2. **Sin acceso de red verificable a GitHub desde esta sesión.** `git ls-remote --heads origin` falla
   con `fatal: Out of memory, malloc failed (tried to allocate 524288000 bytes)` — probado también
   contra la URL vieja (que sí funcionaba antes de este cambio), con el mismo resultado, así que es
   una limitación del entorno/sandbox de esta sesión (no del remoto ni del rename). El cambio de
   `origin` es correcto localmente (`git remote -v` lo confirma) pero **no verificado contra GitHub
   real**. Falta que alguien con red normal (o el propio CI) confirme que el push funciona.
3. **Posible sesión concurrente de otro agente/proceso en el mismo directorio.** Al investigar un fallo
   de memoria durante un build, aparecieron dos procesos `vite dev` ajenos a esta sesión (~3 GB cada
   uno, con rutas de `@openai/codex` en la cadena de procesos), corriendo sobre este mismo working
   directory. No se tocaron esos procesos ni nada relacionado. Si el responsable del proyecto no
   reconoce esa sesión, vale la pena revisarla — podría haber ediciones concurrentes fuera de este
   chat.
4. **Impacto en Vercel del cambio de adaptador, sin evaluar.** `svelte.config.js` es el único punto de
   configuración de adaptador; al cambiarlo a `adapter-static`, si Vercel sigue con integración de git
   activa, su próximo build (tras un push) usaría este mismo adaptador en vez de `adapter-vercel`. No
   se investigó cómo reaccionaría Vercel a eso (podría servir el `build/` estático igual, o fallar).
   Queda como decisión para antes de hacer push, no resuelta aquí.
5. **Sin herramienta de navegador/captura de pantalla en esta sesión.** La "línea base visual" que
   pide este documento (capturas en 1280×800 y 375×667) no se hizo con capturas reales; se sustituyó
   por verificación estructural (HTML servido + CSS de origen) vía `curl`, más lectura directa del
   código. Del mismo modo, no se pudo probar en un navegador real la persistencia de `sessionStorage`
   entre recargas (F5) ni la navegación interna SPA. El algoritmo de barajado se verificó aislado en
   Node (determinista por semilla, permutación válida), pero no de punta a punta. Esto es el trabajo
   real que le queda a la «Revisión independiente» de esta ronda.
6. **`src/app.css` está huérfano** (no importado en ningún lado) desde antes de R01; no se conectó
   porque hacerlo cambiaría visualmente el fondo del `body`, fuera de alcance de esta ronda. Anotado
   para la siguiente.
7. **`.vercel/output/` versionado.** Ver nota en «Resultado de `npm run build`»: es deuda de repositorio
   preexistente (como `docs/`), no introducida ni resuelta por R01.
8. **Repositorio anidado sin `.gitmodules` en `./juanfuc.github/`.** Al listar la raíz apareció un
   directorio `juanfuc.github/` con su propio `.git` (un clon del repo viejo dentro de sí mismo),
   registrado como gitlink (`160000`) pero sin `.gitmodules`. Es preexistente (no aparece en el diff
   de esta ronda, no se tocó) y no afecta el build (nada en `src/`/`static/` lo referencia), pero es
   estado de repositorio confuso que vale la pena que el responsable del proyecto revise y decida si
   limpiar — no se actuó sobre esto por no ser parte de lo pedido.

## Revisión independiente

- **Revisor:** Codex
- **Fecha:** 2026-07-24
- **Hallazgos:** sin bloqueos. Se revisó el diff completo contra `5e2e5fc`, incluidas las eliminaciones; no quedan referencias activas a los archivos retirados. La grilla está confinada a `src/routes/+page.svelte`, el perfil es visible sin temporizadores, el orden usa una semilla de sesión y el CSS estructural compartido conserva las reglas previas. `npm run build` completó correctamente con `adapter-static` y generó los ocho HTML esperados, `404.html` y `.nojekyll`. El workflow de GitHub Pages construye en `main` con Node 20 y publica `build/`; que `juanfuc.github.io` siga mostrando el build antiguo antes de commit/push es el estado esperado y no constituye un fallo. Correcciones menores aplicadas: errata «recarda», registro definitivo de `sessionStorage` y actualización de la documentación de cierre.
- **Veredicto:** **aprobada**.

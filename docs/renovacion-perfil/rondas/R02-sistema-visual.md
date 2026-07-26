# R02 — Sistema visual

**Estado:** Cerrada y aprobada (2026-07-25).
**Toca código del sitio:** sí.
**Checkpoint previo:** commit `45ae0fd` ("refactor: establish structural base and GitHub Pages deployment"). Los cambios de esta ronda son un diff limpio sobre ese commit.

## Estado definitivo al cierre

Esta sección es la referencia autoritativa del sistema visual y funcional tal como quedó aprobado. Todo lo que sigue en este documento (incluida la sección "Resumen del estado vigente" justo debajo) es **historial de cómo se llegó aquí** — léase para entender el porqué de una decisión, no como descripción del estado actual si contradice esta sección. Los valores se leyeron directamente del código en el momento del cierre, no de informes previos.

### Tipografía y paleta

- **IBM Plex Sans** en navegación, títulos, párrafos y descripciones. **IBM Plex Mono** únicamente en metadatos (fechas, tipo, ISBN, volumen, conteos). Cero tipografías serif en el sitio — los tres archivos `.woff2` autohospedados en `static/fonts/` son Sans (variable, pesos 400–600) y Mono (400 y 500); no existe ningún archivo ni declaración `@font-face` de una serif.
- **Fondo:** `--color-bg: #fbfcfc` (definido en `src/app.css`). Es el tercer y último valor probado en esta ronda (antes `#dbe6e6`, luego `#f5f8f9`); los dos anteriores se percibían, en revisión visual real, como una superficie azul en vez de un blanco con carácter.
- **Tinta:** `--color-ink: #054f6d` (azul petróleo, sin cambios desde el checkpoint original) para texto y bordes.
- **Acento:** `--color-accent: #f4ba00` (amarillo original, sin oscurecer) para hover, foco, `aria-current`/página actual, subrayado de `h1` y acentos generales. Es una decisión explícita del responsable del proyecto, tomada después de probar y descartar dos variantes oscurecidas por contraste (`#7a5000` y `#7a5e00` recalculado en OKLCH): el contraste real (~1.7:1 contra el fondo vigente) queda documentado en el propio `src/app.css`, no resuelto con una variante nueva.
- `<html lang="es">` en `src/app.html`.

### Estructura general

- **Encabezado horizontal en escritorio:** nombre del sitio a la izquierda, menú a la derecha, misma franja (`justify-content: space-between`), con `flex-wrap` como única adaptación a anchos angostos (no se apila por una regla fija de breakpoint). El nombre usa `clamp(1.75rem, 1.2rem + 1.6vw, 2.5rem)`, peso 700 — mayor presencia que el menú, pero dimensionado para convivir con él en la misma fila.
- **Sin línea divisoria bajo el encabezado.** La separación con el contenido es aire: `margin-bottom: var(--space-header-gap)` en la propia regla `nav` de `src/routes/+layout.svelte`. El token vigente es **`--space-header-gap: clamp(4rem, 7vw, 7rem)`** (definido una sola vez en `src/app.css`), aplicado igual en portada y en todas las páginas interiores — ninguna página fija su propio padding-top para este espacio.
- **Páginas internas sobre una única superficie clara:** `.container` (`src/lib/styles/structural.css`) no tiene `background-color` propio — se apoya en el `--color-bg` de `<body>` — y se delimita con `border: 1px solid var(--color-line)` (línea traslúcida derivada de la tinta) más su propio padding (`var(--space-7) var(--space-6)` en escritorio, `var(--space-6) var(--space-4)` en móvil). Sin sombras, sin `border-radius`, sin degradados, sin cápsulas, sin panel azulado independiente en ningún punto del sitio.

### Portada

- **Retícula regular y simétrica:** `display: grid`, columnas fijas (4 escritorio / 3 tablet ≤1024px / 2 móvil ≤768px), celdas `aspect-ratio: 4/3` + `object-fit: cover` — mismo criterio de proporciones iguales que ya usaba el checkpoint `45ae0fd` (que recortaba con `overflow: hidden` sobre filas de 250px fijos), adaptado a viewport con `aspect-ratio` en vez de una altura en píxeles.
- **Estructura completa visible de inmediato:** la copia en `localStorage` (`homeGridData`) se muestra apenas existe, sin esperar red; si no hay copia, se pide directamente. El armazón de la retícula no depende de que cada imagen termine de cargar (cada celda reserva su espacio con `aspect-ratio`).
- **Orden aleatorio estable dentro de la sesión:** semilla en `sessionStorage` (`gridShuffleSeed`, `mulberry32`), sin cambios desde R01.
- **Datos:** hoja de imágenes de portada (`indice`, `imagen`, `link` — verificado que no tiene columnas de título/autor). **17 imágenes utilizables** al cierre (una fila con imagen corrupta —comillas sin escapar, típico de una etiqueta `<img>` pegada por error— se descarta con `isUsableImage`, sin tocar la hoja).
- **Metadatos por cruce de enlaces**, no por autoría de la imagen ni por dominio: `normalizeLink()` compara el `link` de cada imagen contra Artículos y Libros/Capítulos (por DOI normalizado — se compara el identificador `10.xxxx/...`, no la URL completa del resolutor) y contra Programación (por URL normalizada: minúsculas, sin barra final, sin `#`, sin parámetros de idioma/tracking). Con coincidencia, la ficha muestra **título real + tipo + año** del contenido enlazado (tipo fijo "Artículo"/"Libro"/"Capítulo" para las tres tablas bibliográficas; para Programación, su propio campo `tipo`). Sin coincidencia, no hay ficha — nunca se inventa texto ni se deriva del dominio (esa idea se probó y se descartó, ver «Decisiones descartadas»).
- **Interacción:** gris al cargar en dispositivos con puntero fino (`(hover: hover) and (pointer: fine)`); en `:hover`/`:focus-visible` la imagen recupera color junto con la ficha (veladura en `--color-ink`, título en Sans, tipo/año en Mono). En táctil/puntero grueso, la imagen ya está en color y la ficha ya es visible desde el primer render — sin doble toque, sin JS simulando hover.
- **Carga progresiva:** las primeras 4 imágenes (`PRIORITY_COUNT`, aproximando la primera fila de escritorio) usan `loading="eager"` + `fetchpriority="high"`; el resto, `loading="lazy"` + `fetchpriority="low"`. Las 17 usan `decoding="async"`.
- **Espacio reservado:** `aspect-ratio: 4/3` más un fondo neutro (`rgba(5, 79, 109, 0.06)`, derivado de la tinta) visible mientras la imagen no ha pintado — nunca un hueco vacío ni blanco puro, y sin cambio de tamaño al llegar la imagen. Una animación CSS de aparición (no ligada a la descarga de la imagen, así que nunca deja nada invisible si falla JS) respeta `prefers-reduced-motion`.
- **Imagen fallida → ficha tipográfica:** `on:error` marca la imagen como fallida (una vez, sin reintentos) y la celda muestra título + tipo/año en texto, mismo tamaño de celda, sin icono de imagen rota; el enlace sigue funcionando. Una imagen defectuosa no afecta a las demás.
- **Tolerancia a índices duplicados:** la causa real observada durante el desarrollo fue un `indice` repetido en la hoja de Google Sheets. La clave del `{#each}` ya no depende solo de `indice`: combina `indice` + enlace normalizado + posición como último desempate (`entryKey`). En modo de desarrollo (`import.meta.env.DEV`, eliminado del build de producción) se emite una advertencia de consola si se detectan índices duplicados, sin bloquear ni recortar la retícula.

### Datos y caché

Confirmado leyendo el código de las siete páginas que consultan Google Sheets (portada, Publicaciones, Artículos, Libros, Capítulos, Experiencia, Programación) — todas siguen el mismo patrón:

1. Si existe una copia en `localStorage`, se muestra de inmediato (sin esperar red).
2. Se consulta la red en paralelo, siempre, haya o no copia local.
3. Si la red responde, esa respuesta reemplaza los datos en pantalla y actualiza `localStorage`.
4. Si la red falla, se conserva lo que ya había en pantalla (la copia local, o lo que trajo una carga previa) — nunca se vacía la página por un fallo de red.

Sustituye tanto el esquema anterior de caché sin expiración (Publicaciones/Artículos/Libros/Capítulos) como el de expiración por una hora (Experiencia/Programación): ambos podían dejar ver datos desactualizados indefinidamente sin volver a intentarlo. La portada, además, cruza cuatro fuentes de metadatos (Artículos/Libros/Capítulos/Programación) con este mismo patrón, en paralelo entre sí y sin bloquear el render de la retícula.

### Publicaciones

- `/publicaciones` muestra de entrada la **lista completa** (vista "Todo"), no una selección aleatoria: un índice bibliográfico unificado que combina Artículos + Libros + Capítulos, ordenado por año descendente (empate resuelto por orden de concatenación Artículos→Libros→Capítulos, estable y predecible).
- El acceso a Artículos/Libros/Capítulos **es navegación real entre rutas** (`<a href="/publicaciones/articulos">`, etc. — no un filtro en memoria dentro de la misma página): cada categoría sigue siendo su propia página, con sus propios datos. La fila de pestañas de `/publicaciones` (Todo/Artículos/Libros/Capítulos) muestra el conteo real de cada una, calculado desde los datos cargados, nunca escrito a mano.
- Las tres vistas por categoría usan los datos reales de sus propias hojas (sin cambios en esta ronda); los registros nuevos que se añadan a cualquiera de las tres hojas se incorporan solos, sin tocar código.
- Sistema visual: fondo compartido, filas separadas por línea fina, año en Mono, título en Sans como elemento principal, tipo/publicación contenedora como metadato — sin imágenes, sin grilla, sin apariencia de segunda portada.
- Copia local primero, red después, sin pantalla vacía durante la actualización (mismo patrón que el resto del sitio).
- **Cantidades actuales:** Artículos 7, Libros 2, Capítulos 4 — **13 registros** en la vista "Todo".

### Programación creativa y Proyectos de Humanidades Digitales

- **Fichas ligeras de entrada:** ningún `<iframe>` existe en el DOM durante la carga inicial (confirmado: 0 ocurrencias en el HTML prerenderizado). Cada proyecto se muestra como ficha con previsualización (columna real `imagen`, cuando existe y carga correctamente) o, si no hay imagen o la imagen falla al cargar (`on:error`), una variante tipográfica equivalente — mismo tamaño de celda, sin icono de imagen rota.
- **Iframe bajo petición:** se monta solo al pulsar "Cargar vista interactiva" (botón real), como máximo uno activo a la vez — abrir otro cierra y desmonta el anterior (bloque `{#if}` de Svelte, no `display: none`). "Volver a la ficha" lo desmonta y devuelve el foco al botón que lo abrió. "Abrir proyecto ↗" es un enlace real, siempre disponible, independiente del estado del iframe.
- **Identidad estable:** el iframe activo, las imágenes fallidas y los botones enfocables se identifican por `itemId(item)` (el `link` del proyecto), no por posición en la lista — necesario porque el filtrado cambia qué se muestra y en qué orden relativo.
- **Filtros de dos niveles**, exclusivamente sobre `tipo`/`subtipo`, filtrado en memoria (sin volver a consultar Sheets ni tocar `localStorage`):
  - Primer nivel — Todos / Programación creativa / Humanidades digitales (etiqueta de interfaz; el valor real es "Proyecto de Humanidades Digitales").
  - Segundo nivel, contextual al tipo elegido — oculto en "Todos"; Videojuego/Sonido generativo/Visualización bajo Programación creativa; Plataforma de investigación/Edición digital bajo Humanidades digitales.
  - Cambiar el tipo reinicia el subtipo a "Todos". Cambiar cualquier filtro conserva el orden original (se filtra con `.filter()`, nunca se reordena) y, si oculta la ficha cuyo iframe está abierto, lo cierra automáticamente.
  - Botones reales (`<button>`) con `aria-pressed`, agrupados en `<fieldset>`/`<legend>`; conteo visible en cada botón, calculado desde los datos. Región `aria-live="polite"` discreta anuncia el número de resultados. Controles en `flex-wrap` para envolver sin desbordamiento en móvil.
  - Normalización defensiva pequeña y explícita (`TYPE_ALIASES`/`SUBTYPE_ALIASES`) para variantes históricas (`Proyecto personal de Humanidades Digitales`, `Video Juego`, `Visualización/Mapa`) que pudieran quedar en registros antiguos o en una copia de `localStorage` guardada antes de normalizar la hoja — la hoja vigente ya usa los valores canónicos.
- **Cantidades actuales** (leídas de la hoja vigente): 10 proyectos en total — Programación creativa 4 (Videojuego 1, Sonido generativo 1, Visualización 2); Humanidades Digitales 6 (Plataforma de investigación 5, Edición digital 1).

### Decisiones descartadas durante R02

Se probaron y se abandonaron explícitamente, en este orden:

- IBM Plex Serif para cuerpo de texto y títulos (el usuario pidió sans serif en todo el sitio).
- Fondos estructurales azulados: `--color-bg-panel` (`#e6ebf1`) como relleno de `.container`/`.home-grid`, y los propios `#dbe6e6`/`#f5f8f9` como fondo general.
- Navegación en amarillo oscurecido/marrón (`--color-accent-ink: #7a5000`, luego `#7a5e00` recalculado en OKLCH como `--color-accent-text`) — el usuario prefirió el amarillo vivo original sin variante.
- Subrayado y color rojo/naranja en los estados de navegación (nunca hubo rojo real en el código; el "rojo" percibido era el matiz desviado del ámbar oscurecido).
- Retícula de altura variable en portada/Publicaciones (imágenes sin recortar, cada una con su proporción natural) — sustituida por celdas regulares.
- Dominio del enlace como metadato de la imagen de portada — sustituido por el cruce real contra Artículos/Libros/Capítulos/Programación.
- Autoría de la imagen (columnas `titulo`/`autor` propuestas para la hoja de portada) como contexto — la ficha describe el **contenido enlazado**, no quién tomó o subió la imagen.
- Carga inicial de todos los `<iframe>` de Programación simultáneamente.
- Menú apilado bajo el nombre del sitio en escritorio (encabezado en dos filas) — se volvió a una sola franja horizontal.
- Grilla aleatoria de imágenes en `/publicaciones` — sustituida por el índice bibliográfico unificado.

## Resumen del estado vigente (histórico — ver "Estado definitivo al cierre" arriba)

R02 tuvo varias correcciones parciales antes de esta pasada (documentadas más abajo, conservadas como historial). **Esta última pasada — "Revisión integral: datos, caché y recomposición" — reemplaza cualquier valor o decisión de las pasadas anteriores donde exista contradicción.** Estado vigente en el momento en que se escribió esta sección (ya superado en varios puntos por pasadas posteriores; ver la sección de arriba):

- **Datos:** las seis páginas que cachean en `localStorage` (Publicaciones, Artículos, Libros, Capítulos, Experiencia, Programación) usan ahora **red primero, `localStorage` solo como respaldo si la red falla** (antes: caché sin expiración en cuatro de ellas, expiración de 1 hora en las otras dos). La portada y la grilla de Publicaciones descartan filas cuya imagen venga corrupta desde la hoja (comillas sin escapar). Ver «Diagnóstico de datos» más abajo.
- **Fondo:** `--color-bg: #f5f8f9` (antes `#dbe6e6`) — un blanco derivado del azul petróleo, no el tono que en pantalla se leía como "azul". Un único fondo en todo el sitio, sin panel/superficie distinta (esto ya venía de la pasada anterior y no cambió).
- **Amarillo:** `--color-accent: #f4ba00`, sin oscurecer, usado también como color de texto en navegación/hover/foco/página actual (decisión de la pasada anterior, vigente).
- **Tipografía:** IBM Plex Sans en todo el sitio (navegación, títulos, párrafos); IBM Plex Mono solo en metadatos con función real. Cero serifas. `lang="es"`. (Sin cambios en esta pasada.)
- **Encabezado:** nombre del sitio y menú **apilados** (antes: una sola fila con nombre a la izquierda y menú a la derecha), nombre notablemente más grande (`clamp(1.85rem, …, 3.25rem)`, antes hasta `1.6rem`), más espacio propio del encabezado antes de la retícula.
- **Retícula:** celdas regulares con `aspect-ratio: 4/3` + `object-fit: cover` (sin cambios respecto a la pasada anterior; se mantiene).
- **Contextualización de imágenes:** en portada y Publicaciones **no existen columnas de título/autor** en la hoja real (verificado leyendo el CSV publicado) — la suposición de una pasada anterior (`item.titulo`/`item.autor`) era incorrecta y se retira. Se muestra en su lugar el dominio del enlace (dato que sí existe) como contexto mínimo. Artículos/Libros/Capítulos sí tienen `Título`/`Autor` reales y no cambiaron.
- **Páginas internas:** mismo fondo que el resto del sitio, borde fino en vez de relleno de color (de la pasada anterior), con más espaciado interior (`.container` y `.list-card-media`/`.list-card-body` con más padding que antes).

## Nota sobre el origen de esta ronda

Cuando esta sesión comenzó, `README.md` y `decisiones.md` todavía decían **"no hay una ronda de implementación activa; la siguiente está pendiente de definición"**, y no existía ningún archivo `rondas/R02-*.md` en el repositorio ni `contexto_revision_conceptual_perfil_web_R02.md` (ni en el repo ni como adjunto). Esta contradicción se señaló al responsable del proyecto antes de tocar código. El responsable del proyecto, en el mismo mensaje, dio directamente el alcance, los criterios y los límites de R02 como instrucción explícita de trabajo. Este documento formaliza esa instrucción como el registro de la ronda (creándolo, ya que no existía antes) y documenta su ejecución, para que el corpus documental quede consistente con lo que se autorizó y se hizo. Ver «Actualización de `README.md` y `decisiones.md`» más abajo para el resto de la sincronización.

## Objetivo

Convertir los principios conceptuales acordados (brief.md, decisiones.md) en un sistema visual coherente para todo el sitio: tipografía cargada de verdad, escala y jerarquía, anchos de lectura, ritmo vertical, un sistema de líneas/separadores, tratamiento de imágenes, y estados de interacción (hover/focus/active/aria-current) — preservando la arquitectura, los datos, el despliegue y las rutas resueltas en R01, y la paleta, la composición plana, el efecto gris→color, la ausencia de imagen dominante, la grilla confinada a `/`, el orden aleatorio estable y el contenido actuales.

Idea rectora: gabinete/colección + archivo plano, interpretada como principio editorial y de navegación (no literalmente: sin cajones, texturas, papeles, sepias, 3D ni museografía). Caja Negra Editora como referencia distante solo para tipografía, líneas, ritmo y densidad — no como plantilla.

## Decisiones visuales tomadas

### Tipografía (antes pendiente en `decisiones.md`)

- **Familia elegida: IBM Plex (Sans + Serif + Mono).** Se eligió precisamente por ser una familia diseñada como *sistema* (tres registros coordinados, mismo origen, mismas proporciones) — lo que traduce literalmente la idea de "sistemático" del brief sin depender de una sola tipografía de moda. No es Poppins/Montserrat (las declaradas-pero-nunca-cargadas de antes): esas dos nunca se resolvieron como decisión, solo como intención rota (diagnóstico R00).
- **Roles por función, no por contenedor:**
  - **Sans** (nav, `h1`, enlaces de índice/pestañas) → registro de navegación/estructura.
  - **Serif** (párrafo de `/perfil`, descripciones, `h2` de fichas bibliográficas) → registro de lectura/contenido. `h3` (título del libro contenedor en Capítulos) en **serif itálica**, replicando la convención académica de citar títulos de libro en cursiva.
  - **Mono** (fechas, volumen/número, ISBN, tipo, editorial, ciudad, autor) → registro de metadato/ficha de archivo.
- **Carga técnica:** autohospedada (no Google Fonts en tiempo de ejecución). Se descargaron 6 archivos `.woff2` (subconjunto `latin`, suficiente para español: cubre vocales acentuadas y `ñ`) a `static/fonts/`: `IBMPlexSans-Variable` (400–600, un solo archivo variable), `IBMPlexSerif-Regular`, `IBMPlexSerif-Medium`, `IBMPlexSerif-Italic`, `IBMPlexMono-Regular`, `IBMPlexMono-Medium`. `@font-face` en `src/app.css`, con `font-display: swap`. Un `<link rel="preload">` en `src/app.html` para la variable Sans (la de uso más inmediato/global). Se prefirió autohospedaje sobre CDN de Google Fonts por robustez (sin dependencia de red de terceros en producción) y porque no había ninguna razón pragmática para preferir la alternativa.
- **Escala y ritmo:** tokens en `:root` (`src/app.css`): `--fs-meta` (13px), `--fs-ui` (16px), `--fs-body` (18px), `--fs-h1/h2/h3` con `clamp()` fluido. Esto **corrige** el `font-size: 2vw` de los `h1` de todas las páginas interiores (bug real: en pantallas muy anchas o muy angostas producía tamaños absurdos); ahora escala entre un mínimo y máximo legibles.
- **Ancho de lectura:** `--measure: 68ch`, aplicado al párrafo de `/perfil` y a `.description` de Programación (los dos bloques de prosa larga del sitio). No se tocó el `max-width: 1200px` del `.container` (que sigue siendo el marco de página, no la medida de lectura): el texto queda centrado dentro de ese marco con su propia columna angosta.
- **Justificado → alineado a la izquierda:** se quitó `text-align: justify` (heredado de la regla `.container` y del `<p>` de perfil). Con el ancho de lectura ahora angosto y sin partición de palabras (`hyphens`), el justificado genera ríos de espacio en blanco; alineado a la izquierda es la opción robusta estándar para texto web de ancho variable.
- **Párrafos de `/perfil`:** el texto vive en un único `<p>` con líneas en blanco entre "párrafos" en el código fuente. Sin una regla de `white-space`, esas líneas en blanco se colapsan en HTML y el texto se ve como un bloque continuo. Se añadió `white-space: pre-line` (solo CSS, sin tocar el contenido ni el marcado) para que los saltos de párrafo se vean como tales — esto es una corrección de legibilidad, no un cambio de contenido.

### Líneas, bordes y separadores (antes pendiente)

- Tokens `--line-thin` (1px) y `--line-thick` (2px), color `--color-line` (tinta azul petróleo al 22% de opacidad, no un gris neutro — así la línea pertenece a la misma paleta).
- **Nav:** línea fina completa debajo de toda la barra, separando la navegación del contenido.
- **`h1` de cada página:** el propio título es `inline-block` con un subrayado grueso corto (`--line-thick`, color ámbar `--color-accent`) del ancho del texto — no una franja institucional de ancho completo.
- **Fichas de listado** (Artículos/Libros/Capítulos/Programación/Experiencia): línea fina entre entradas consecutivas (`.list-card + .list-card`, `.card + .card`), no antes de la primera ni después de la última — ritmo de "registro/ficha de archivo" en vez de tarjetas sueltas sin separación (antes no había ninguna separación visual entre entradas consecutivas: un vacío real, no solo estético).
- **Fila de autor:** línea fina encima de `.autor`, separándola de los datos bibliográficos como una firma de cierre de cada ficha.
- **Pestañas de Publicaciones** (Artículos/Libros/Capítulos): línea fina debajo de toda la fila de pestañas, separándola de la grilla.

### Composición plana (antes pendiente)

- `--radius` no se definió como token porque se resolvió como ausencia total: se quitó el único `border-radius: 8px` que quedaba (tarjetas de Experiencia). Sin bordes redondeados, sin sombras (ver abajo), en ningún componente del sistema.

### Estados de interacción — hover/focus/active/aria-current (antes pendiente, incluía la duda sobre el `scale()` en hover)

- **Se elimina el aumento de escala (`transform: scale()`) y las sombras `box-shadow` de "brillo"** en todas las imágenes de grilla/ficha y en el enlace "volver" (flecha) de Artículos/Libros/Capítulos. Estos eran exactamente los dos patrones que el brief pide evitar ("sombras decorativas", "efectos que dependan del ratón y no tengan alternativa táctil o de teclado"): el `scale()`+sombra no tenía equivalente de foco/teclado, y el crecimiento de fuente del enlace "volver" (3.5em→3.8em) producía además un salto de layout al pasar el cursor.
- **Se conserva el efecto gris→color** como el único mecanismo de retroalimentación de interacción en las imágenes (mandado explícitamente a conservar). Se aplica en `:hover` **y en `:focus-visible` del enlace contenedor**, para que quien navega por teclado obtenga el mismo efecto que quien usa ratón — esto resuelve, para el caso concreto de las imágenes, la advertencia de "sin alternativa táctil o de teclado" del brief.
- **`aria-current="page"`:** ya existía desde R01; ahora además de cambiar de color (tinta ámbar accesible, ver abajo) lleva subrayado, para no depender solo del color como única señal (principio general de accesibilidad, no exigido explícitamamente pero de costo cero).
- Enlaces del nav, pestañas de Publicaciones y enlace "volver": mismo patrón de color en `:hover`/`:focus-visible`.

### Paleta — contraste (ajuste dentro de "conserva la paleta acordada como punto de partida")

Al revisar contraste (pedido explícitamente en el punto 4 de esta ronda) se verificó con la fórmula de luminancia relativa de WCAG que el ámbar `#f4ba00` usado como **color de texto** sobre los fondos claros del sitio (`#e6ebf1` / `#dbe6e6`) da un contraste real de **~1.44:1** — muy por debajo del mínimo de 3:1 (texto grande) y 4.5:1 (texto normal). Lo mismo ocurría con el rojo `#EE4E4E` (autor, ~2.93:1) y el naranja `#E16526` (enlaces de DOI, ~2.81:1). Esto no era una sospecha ("contraste dudoso" en el diagnóstico original): es un fallo real y medible, presente en todos los `h1` de páginas interiores, en los iconos del nav y en enlaces/autor de las fichas bibliográficas.

Se resolvió **sin cambiar la identidad cromática** (mismos tonos, misma paleta como punto de partida): se añadieron variantes oscurecidas del mismo matiz para uso como texto/ícono —`--color-accent-ink` (`#7a5000`, ~5.5–5.75:1), `--color-red-ink` (`#b23838`, ~4.87:1), `--color-orange-ink` (`#a34a16`, ~4.83:1) — y se reservan los tonos vivos originales (`--color-accent`, `--color-red`, `--color-orange`) para uso gráfico/decorativo (líneas, subrayado de `h1`), donde el contraste de texto no aplica. Los `h1`, antes en ámbar puro sobre fondo claro, ahora usan la tinta azul petróleo (`--color-ink`, ~7:1, ya usada para el cuerpo del sitio) con el ámbar solo como línea de acento.

## Cambios realizados — archivos afectados

*(Todo relativo al checkpoint `45ae0fd`; ver `git diff 45ae0fd` para el detalle exacto. No hay commit nuevo: el árbol de trabajo es el diff.)*

**Creados:**
- `static/fonts/IBMPlexSans-Variable.woff2`, `IBMPlexSerif-Regular.woff2`, `IBMPlexSerif-Medium.woff2`, `IBMPlexSerif-Italic.woff2`, `IBMPlexMono-Regular.woff2`, `IBMPlexMono-Medium.woff2` (subconjunto `latin` de Google Fonts, descargados y autohospedados).
- `docs/renovacion-perfil/rondas/R02-sistema-visual.md` (este documento).

**Modificados:**
- `src/app.css` — de "huérfano, solo `body{background;color}`" pasa a ser el hoja de tokens/reset/`@font-face` global. **Se conecta por primera vez** (import en `src/routes/+layout.svelte`), algo que R01 dejó explícitamente pendiente ("`src/app.css` continúa huérfano y pendiente de una decisión visual"). Antes de este cambio no se aplicaba en ningún lado; ahora define `:root` (paleta + tipografía + escala + espaciado + líneas), `box-sizing: border-box` global, `body` (fondo/color/tipografía base), `:focus-visible` global.
- `src/app.html` — un `<link rel="preload">` para la fuente Sans. No se tocó `<html lang="en">` ni el resto (fuera de alcance, ver «Límites respetados»).
- `src/lib/styles/structural.css` — `.container` usa tokens de color/espaciado en vez de literales, quita `font-family`/`text-align:justify` de bloque (la tipografía ahora se decide por rol, no por contenedor) y añade padding responsive; `.list-card` gana ritmo vertical por tokens y línea entre entradas consecutivas.
- `src/routes/+layout.svelte` — conecta `app.css`; nav reescrito: tipografía del sistema, línea inferior, pestañas en mayúsculas con tracking (registro de "índice"), estados de color accesibles, `min-height: 44px` en los enlaces para tamaño táctil.
- `src/routes/+page.svelte` (grilla de portada) — grilla de altura variable (`grid-auto-rows` implícito + `align-items:start`, sin recorte por `overflow:hidden` ni fila fija de 250px); 4/3/2 columnas según ancho (antes 4/1); se quita `scale()`+sombra en hover, se conserva y se extiende a `:focus-visible` el efecto gris→color.
- `src/routes/perfil/+page.svelte` — tipografía serif de lectura, ancho de lectura, `white-space: pre-line`.
- `src/routes/experiencia/+page.svelte` — `h1` corregido (sans, tinta, `clamp()`, línea de acento), tarjetas sin `border-radius` ni fondo propio redundante, línea entre entradas, metadatos en mono, descripción en serif.
- `src/routes/publicaciones/+page.svelte` — grilla de altura variable 3/2 columnas (mismo tratamiento que portada), pestañas reescritas como mini-nav (mayúsculas, línea inferior), se quita `scale()`+sombra.
- `src/routes/publicaciones/articulos/+page.svelte`, `.../libros/+page.svelte`, `.../capitulos/+page.svelte` — `h1` corregido, enlace "volver" sin crecimiento de fuente ni sombra (con `aria-label` nuevo, ver «Ajustes mínimos de marcado»), imágenes sin `scale()`/sombra con reveal en `:focus-visible`, metadatos en mono, `autor` con línea superior y tinta roja accesible, enlaces de DOI en tinta naranja accesible. Capítulos añade además `h3` (título del libro) en serif itálica.
- `src/routes/programacion/+page.svelte` — mismo tratamiento de `h1`/metadatos/autor/descripción; el enlace del título de proyecto pasa de estilo inline a una clase (`title-link`) para poder aplicarle el color/subrayado accesible en hover y foco.

**No tocados a propósito:** `svelte.config.js`, `package.json`, `.github/workflows/deploy.yml`, `src/routes/+layout.js`, cualquier ruta, la estrategia de lectura de Sheets/CSV/`localStorage`, `src/app.html` más allá del `<link>` de preload (incluye `lang="en"`), `docs/` (build antiguo), `.vercel/output/`. Confirmado por `git diff --stat` contra `45ae0fd`: sin cambios en esos archivos.

## Ajustes mínimos de marcado (documentados, no estructurales)

Todos ellos CSS-compatibles o de accesibilidad, ninguno cambia la arquitectura ni el contenido:

1. **`aria-label="Volver a Publicaciones"`** en el enlace-flecha de Artículos/Libros/Capítulos (antes un enlace de solo-ícono sin nombre accesible — el mismo tipo de hueco que R01 ya había cerrado para los íconos sociales del nav, pero que no cubría este caso porque son componentes locales de cada página).
2. **`class="meta"`** añadida a los párrafos de metadato (revista, volumen, ISBN, institución, período, tipo, fecha, etc.) en Experiencia/Artículos/Libros/Capítulos/Programación, para poder darles el tratamiento tipográfico mono sin afectar `.autor` ni el resto de `<p>`.
3. **`class="title-link"`** en el enlace de título de Programación (antes tenía `style` inline).
4. Se quitaron los `<br><br>` usados como espaciado bajo cada `h1` (Experiencia/Artículos/Libros/Capítulos/Programación): el espaciado ahora es `margin`/`padding` del propio `h1`, no elementos de salto de línea usados como maquetación.
5. En la grilla de Publicaciones, se restauró la regla `.card:last-child { grid-column: 2/3 }` (centra el ítem sobrante cuando el total no es múltiplo de 3) pero **anulada explícitamente en el breakpoint móvil de 2 columnas**, donde esa regla no tiene sentido.

Ninguno de estos ajustes toca rutas, datos, SEO/metadatos generales, ni la estructura resuelta en R01.

## Lo que se dejó pendiente a propósito (no se resuelve en R02)

- **Franja contextual de la portada:** sigue sin resolverse (no estaba en el alcance dado para esta ronda; la portada sigue siendo nav + grilla, sin banda de texto añadida).
- **`lang="en"` y metadatos generales (`<title>`, description, Open Graph, JSON-LD):** no se tocan, por instrucción expresa de esta ronda ("no modifiques... SEO, JSON-LD o metadatos generales"). Nótese que `decisiones.md` agrupa `lang` junto con esos metadatos como un único punto pendiente; esta ronda no lo desagrega ni lo cierra.
- **"En curso" / hilos de investigación / jardín digital:** no se crean, por instrucción expresa.

## Verificaciones ejecutadas

- **`npm run build`:** éxito, sin errores ni advertencias (se revisó la salida completa buscando "warn/error/deprecat/a11y"; no aparece ninguno, más allá de la coincidencia trivial de la palabra "error" en el nombre de archivo `error.svelte.js`). Se generaron los 8 HTML esperados en `build/` (`index`, `perfil`, `experiencia`, `publicaciones`, `publicaciones/{articulos,libros,capitulos}`, `programacion`), más `404.html`, `.nojekyll` y los 6 `.woff2` copiados a `build/fonts/`.
- **`git diff --check`:** sin errores de espacio en blanco ni marcadores de conflicto; solo avisos informativos de normalización LF→CRLF (comportamiento estándar de Git en Windows, no un problema del diff).
- **`npm run preview` + `curl` sobre las 8 rutas exigidas** (carga directa, no solo navegación): las 8 devuelven HTTP 200. Se verificó además, leyendo el HTML servido:
  - `aria-current="page"` aparece exactamente una vez por página, en el enlace correcto.
  - El texto de `/perfil` sigue presente íntegro en el HTML prerenderizado.
  - El `<link rel="preload">` de la fuente Sans aparece en las páginas verificadas.
  - `home-grid`/`grid` siguen presentes solo en `/` (mismo patrón de confinamiento de R01; no se tocó esa lógica).
  - El `h1` de Experiencia renderiza con clase de Svelte (confirma que el estilo se aplica por hoja de estilos, no inline).
- **Grep dirigido sobre `src/`:** cero referencias a `Poppins`/`Montserrat` (las fuentes nunca cargadas quedaron reemplazadas); cero `box-shadow`; cero `transform: scale(...)` real (las coincidencias de "scale(" restantes son todas parte de `grayscale(...)`, la función de filtro que sí se conserva); cero colores hexadecimales sueltos fuera de `app.css` (todo pasó a tokens); conteo de `aria-label` = 10, consistente con los 7 preexistentes de R01 (nav + 6 íconos) más los 3 nuevos (`aria-label` en los enlaces "volver").
- **Verificación de contraste:** calculada analíticamente con la fórmula de luminancia relativa de WCAG 2.x (no con una herramienta automática de navegador, que no estaba disponible en esta sesión) para cada combinación texto/fondo relevante antes y después del cambio. Ver la tabla de resultados en «Decisiones visuales tomadas → Paleta».
- **Verificación de anchos táctiles:** por lectura de las reglas CSS resultantes (no medida en un dispositivo real): enlaces de nav y pestañas de Publicaciones con `min-height: 44px` explícito; enlace "volver" con icono de 1.75rem (28px) + padding de 0.5rem (8px) por lado = 44px total.
- **Verificación de desbordamiento horizontal:** por lectura de las reglas CSS (no medida en viewport real): no se detectaron anchos fijos en píxeles que excedan el contenedor ni combinaciones `width: 100%` + `padding` sin `box-sizing: border-box` (se añadió `box-sizing: border-box` global precisamente para descartar esta clase de problema).

### Limitación real de método (léase antes que "verificaciones")

**No hubo herramienta de navegador/captura de pantalla disponible en esta sesión** (se ofreció instalar la extensión de Claude en Chrome y se declinó continuar con herramientas de navegador). Por tanto:

- **No se generaron capturas de pantalla reales** de inicio/perfil/listado en escritorio ni en móvil. Todo lo anterior son verificaciones **estructurales** (HTML servido vía `curl`, CSS compilado, cálculo analítico de contraste), no una revisión visual en el sentido estricto que pide el punto 6 de esta ronda.
- **No se verificó de forma visual** el aspecto real de la tipografía cargada (parecido/densidad/ritmo tal como se ve en pantalla), el comportamiento exacto del `clamp()` en anchos intermedios, la alineación de la barra de navegación en el punto de quiebre a columna (768px), ni el aspecto de la grilla de altura variable (si el "escalón" entre columnas de distinta altura se ve como se pretende o resulta desordenado).
- **No se probó en un dispositivo táctil real** el comportamiento gris→color: por diseño, en un dispositivo sin puntero ni teclado (solo táctil), las imágenes que **no** llevan enlace nunca revelan color (no hay evento que lo dispare), y las que sí llevan enlace solo lo revelarían con un `:focus-visible` post-tap si el navegador lo dispara así (variable según plataforma). Esto es una limitación conocida y aceptada del mecanismo (ver más abajo), no verificada empíricamente aquí.
- Esto repite, para esta ronda, la misma limitación que R01 documentó para sí misma («Sin herramienta de navegador/captura de pantalla en esta sesión»); no se ha resuelto entre rondas porque depende del entorno de ejecución, no del código.

**Queda como trabajo real para la revisión independiente:** abrir `npm run preview` en un navegador real y confirmar visualmente los cinco puntos del punto 6 (inicio y perfil en escritorio/móvil, un listado), la legibilidad de la tipografía cargada, el punto de quiebre del nav, y el aspecto de la grilla de altura variable.

## Decisión documentada: gris→color y dispositivos sin puntero

El brief pide evitar "efectos que dependan del ratón y no tengan alternativa táctil o de teclado" pero también conservar explícitamente "el efecto gris → color de las imágenes". Estas dos instrucciones entran en tensión en un dispositivo puramente táctil (sin ratón ni teclado): no hay ningún evento estándar de CSS que "revele" el color de forma equivalente sin JavaScript nuevo, y añadir un manejador táctil a medida sería una función nueva no pedida ("no agregues nuevas funciones o requisitos").

Se resolvió así: el gris→color queda como una mejora progresiva de `:hover`/`:focus-visible` (con equivalente de teclado, que sí se pudo resolver sin JS nuevo); en un dispositivo solo-táctil la grilla se ve enteramente en escala de grises por defecto, sin bloquear ningún contenido ni funcionalidad (los enlaces siguen siendo completamente táctiles, el `alt` de cada imagen sigue presente). Se documenta como limitación aceptada, no como error, y se deja explícita para que el responsable del proyecto decida si amerita una función nueva en una ronda futura.

## Actualización de `README.md` y `decisiones.md`

Como parte del cierre de esta ronda (siguiendo la regla de trabajo del propio `README.md`: "cuando cierres una ronda, actualiza `decisiones.md`... y abre el documento de la siguiente ronda"):

- `README.md` — se actualiza la referencia de "ronda más reciente" de R01 a R02, y se anota que R02 queda implementada pendiente de revisión (no aprobada todavía).
- `decisiones.md` — se mueven a "confirmadas (R02)" las cinco propuestas que esta ronda resuelve (tipografía, sistema de líneas, composición plana, estados de interacción, eliminación del `scale()` en hover), con referencia a este documento. La franja contextual y el bloque de metadatos/`lang` permanecen en "pendientes" sin cambios.

## Corrección aplicada tras revisión del usuario (segunda pasada, antes de la auditoría final)

Tras la primera implementación (secciones anteriores de este documento), el responsable del proyecto revisó el resultado y pidió dos correcciones puntuales, más una auditoría de cierre. Esta sección documenta esa segunda pasada; las secciones anteriores quedan como registro histórico de lo que se probó primero y por qué se cambió.

### 1. Tipografía: eliminación completa de las serifas

El usuario indicó explícitamente que no quiere tipografías con serifa en ningún punto del sitio. La primera implementación usaba IBM Plex Serif para el cuerpo de `/perfil`, las descripciones de Programación/Experiencia y los `h2`/`h3` de las fichas bibliográficas (Artículos/Libros/Capítulos). Se corrigió así:

- **Se retiraron las tres declaraciones `@font-face` de IBM Plex Serif** (Regular, Medium, Italic) de `src/app.css`, y el token `--font-serif`.
- **Se eliminaron los tres archivos** `IBMPlexSerif-Regular.woff2`, `IBMPlexSerif-Medium.woff2` e `IBMPlexSerif-Italic.woff2` de `static/fonts/` (confirmado: solo quedan los tres `.woff2` de Sans/Mono).
- **Todo el texto de lectura (cuerpo de `/perfil`, descripciones, `h2` de fichas) pasa a IBM Plex Sans**, diferenciado de la navegación y de los `h1` mediante peso/tamaño, no mediante familia.
- **El `h3` de Capítulos** (título del libro contenedor, antes serif itálica) pasa a Sans regular con tracking ligero (`letter-spacing: 0.02em`) para leerse como referencia subordinada sin cursiva ni serifa.
- **IBM Plex Mono se conserva, limitado a metadatos** (fechas, volumen/número, ISBN, editorial, ciudad, tipo, institución/período) y a la fila de autor — nunca en párrafos de lectura.
- **No se introdujo ninguna otra familia** ni se descargó ninguna fuente nueva: el sistema final es Sans + Mono, ambas ya autohospedadas desde la primera pasada.
- **Auditoría de fuentes finales** (`static/fonts/*.woff2`):

  | Archivo | Familia | Peso | Estilo | Uso |
  |---|---|---|---|---|
  | `IBMPlexSans-Variable.woff2` | IBM Plex Sans | variable 400–600 | normal | nav (nombre, menú, iconos), pestañas de Publicaciones, todos los `h1`/`h2`/`h3`, cuerpo de `/perfil`, descripciones, enlace "volver", `title-link` |
  | `IBMPlexMono-Regular.woff2` | IBM Plex Mono | 400 | normal | `.meta` (metadatos bibliográficos/de proyecto en todas las páginas interiores con listado) |
  | `IBMPlexMono-Medium.woff2` | IBM Plex Mono | 500 | normal | `.autor` (fila de firma) — **se detectó en esta auditoría que el archivo estaba descargado pero ningún selector declaraba `font-weight: 500`, por lo que nunca se seleccionaba**; se corrigió añadiendo `font-weight: 500` a `.autor` en los cuatro archivos que la usan (Artículos, Libros, Capítulos, Programación), en vez de borrar el archivo, porque distinguir la fila de firma del resto de metadatos con un peso mayor es coherente con el propio sistema (línea de cierre de cada ficha) |

  Procedencia: los seis `.woff2` originales (y los tres que quedan) se descargaron del CDN de Google Fonts (`fonts.gstatic.com`), subconjunto `latin`, en la primera pasada de esta ronda. Licencia: IBM Plex se publica bajo **SIL Open Font License 1.1**, que permite explícitamente el autohospedaje, la redistribución y la modificación. No hay archivos duplicados, ninguno está codificado en base64 (todos son `.woff2` binarios referenciados por `url()`), y las tres declaraciones `@font-face` restantes usan `font-display: swap` y nomenclatura consistente (`'IBM Plex Sans'`, `'IBM Plex Mono'`).

### 2. Estados de navegación: solo color amarillo, sin rojo ni delineado persistente

El usuario reportó ver "un rojo extraño, un delineado o subrayado" en el nombre-enlace a portada y en el menú, en hover/foco/activo, y pidió que esos tres estados usaran únicamente el amarillo de identidad como color de texto, sin decoración.

**Causa real del "rojo":** no había ningún rojo en el código (`--color-red`/`--color-red-ink` nunca se usaron en el nav). El color aplicado era `--color-accent-ink: #7a5000`, un ámbar oscurecido para cumplir contraste como texto. Su matiz (~39°, calculado por HSL) se había desviado hacia el naranja/rojo respecto del ámbar vivo de identidad `--color-accent: #f4ba00` (~46°); un color oscuro y saturado que se desvía de matiz hacia el rojo se percibe como "rojizo" antes que como "ámbar oscuro" (el ojo exagera el corrimiento hacia el rojo en tonos oscuros y saturados — efecto Bezold–Brücke). A eso se sumaba: (a) `aria-current="page"` llevaba además `text-decoration: underline`, y (b) el `:focus-visible` global de `src/app.css` aplicaba un contorno de 2px en tinta azul petróleo sobre el mismo enlace — la combinación de un ámbar desviado de matiz + subrayado + contorno azul es, razonablemente, lo que se percibió como "rojo extraño con delineado".

**Causa de la disparidad de amarillo entre h1 y menú:** ninguna. Los `h1` de las páginas interiores nunca fueron amarillo como color de texto en esta ronda (se cambiaron a tinta azul petróleo con un subrayado ámbar corto, por el fallo de contraste que se explica en «Paleta — contraste» más arriba, ~1.44:1 sobre los fondos claros del sitio). Si el usuario los recuerda en amarillo, corresponde al sitio **anterior a R02** (antes de esta ronda, todos los `h1` interiores eran `color: #f4ba00` en texto). Esta ronda no revirtió esa decisión de contraste sin que se pida explícitamente: se documenta aquí para que quede claro que no es un descuido, y queda a la espera de que el responsable del proyecto confirme si prefiere mantener el subrayado (accesible) o quiere explorar otra solución.

**Corrección aplicada:**

- **Valor definitivo de `--color-accent-ink`: `#7a5e00`** (antes `#7a5000`). Mismo matiz que `--color-accent` (~46°, calculado por HSL), oscurecido solo lo necesario para pasar AA como texto: contraste ≈4.8:1 sobre `#dbe6e6` (el fondo más exigente de los dos que usa el sitio) y ≈5.0:1 sobre `#e6ebf1`.
- **Se centralizaron los estados de nav/pestañas en un solo lugar: `src/app.css`.** Antes, el color de `nav a` vivía en `src/routes/+layout.svelte` y el de `.link` (pestañas de Publicaciones) en `src/routes/publicaciones/+page.svelte`, cada uno con su propia regla — el riesgo señalado por el usuario de que divergieran entre archivos era real. Ahora una sola regla en `app.css` gobierna `nav a` y `.link` a la vez; los componentes locales solo definen layout (tamaño, espaciado, mayúsculas), nunca color ni decoración.
- **Reglas definitivas** (verificadas en el CSS compilado, no solo en el fuente):
  - Reposo: `color: var(--color-ink)` (tinta azul petróleo), sin decoración.
  - `:hover`, `[aria-current="page"]`: `color: var(--color-accent-ink)` — solo color, sin subrayado, sin borde, sin fondo, sin cambio de tamaño.
  - `:focus-visible`: mismo color, más un contorno de **1px en el mismo tono ámbar** (`outline: 1px solid var(--color-accent-ink)`, `outline-offset: 3px`) — no un color de sistema ajeno (ni azul, ni rojo). Al depender de `:focus-visible` (no de una clase persistente ni de `:focus` simple), el contorno desaparece en cuanto el foco se mueve: no queda residual tras un clic de ratón (que en general no dispara `:focus-visible`) ni se confunde con la marca permanente de página actual, que es solo color.
- **Se retiró el `text-decoration: underline` que tenía `aria-current="page"`.**
- El `:focus-visible` genérico de `app.css` (contorno azul de 2px) se conserva **para el resto del sitio** (enlaces de imagen de las grillas, enlace "volver", enlaces DOI) — no está en el alcance de esta corrección, que el usuario limitó a "el nombre, el menú, las pestañas y el indicador de página actual". La regla específica de `nav a`/`.link` tiene mayor especificidad y gana sobre la genérica solo donde corresponde.
- **No se tocó** `--color-red-ink` ni `--color-orange-ink` (autor y enlaces DOI de las fichas bibliográficas): no son elementos de navegación y no estaban en la queja del usuario.

### 3. Auditoría de retícula (orden DOM/visual/tab)

Se confirmó por inspección del código (`grep` de `order:`, `columns:`/`column-count` en todo `src/`) que la grilla de altura variable de portada y Publicaciones usa **CSS Grid estándar** (`display: grid` + `grid-template-columns` + `align-items: start`), con el `grid-auto-flow` por defecto (fila por fila, en el mismo orden en que aparecen en el DOM). No se usa `column-count`/multi-columna (que sí reordenaría visualmente respecto de la lectura) ni la propiedad `order` en ningún punto del sitio. Conclusión: **el orden visual, el orden del DOM y el orden de tabulación coinciden siempre**; no hizo falta ningún cambio de técnica. Los datos y su orden estable por sesión (semilla en `sessionStorage`, de R01) no se tocaron.

### 4. Imágenes en dispositivos táctiles/puntero grueso

Se envolvió el mecanismo de gris→color (declaración `filter: grayscale(...)` inicial, más las reglas de `:hover`/`:focus-visible` que lo revierten) dentro de `@media (hover: hover) and (pointer: fine)`, en los cinco lugares donde existía: portada (`src/routes/+page.svelte`), grilla de Publicaciones, y las fichas de Artículos/Libros/Capítulos. Fuera de esa condición (dispositivos táctiles o de puntero grueso) la imagen nunca recibe el filtro gris: se muestra en color desde el primer render, sin JavaScript y sin necesidad de doble toque para seguir el enlace. Es funcionalmente equivalente a la alternativa `@media (hover: none), (pointer: coarse) { /* quitar filtro */ }` sugerida, expresada como condición positiva para no duplicar la declaración del filtro en dos bloques.

### 5. Alcance del diff (archivo por archivo)

Se revisó el diff completo contra `45ae0fd` clasificando cada cambio:

- **Sistema visual compartido:** `src/app.css` (tokens, `@font-face`, estados de navegación centralizados), `src/lib/styles/structural.css`, `src/app.html` (preload de fuente), `src/routes/+layout.svelte` (nav).
- **Correcciones de accesibilidad:** `min-height: 44px` en enlaces de nav/pestañas, `aria-label` en los tres enlaces "volver", `box-sizing: border-box` global, `:focus-visible` coherente, gris→color condicionado a puntero fino.
- **Cambios específicos de portada:** `src/routes/+page.svelte` (grilla de altura variable, columnas responsive, filtro condicionado).
- **Cambios específicos de páginas interiores:** `perfil`, `experiencia`, `programacion`, `publicaciones` y sus tres subsecciones — en todos los casos, tipografía (Sans/Mono por rol), corrección del `h1` (`font-size: 2vw` → `clamp()`), líneas de separación entre entradas, y el filtro gris condicionado donde aplica.
- **Documentación:** `docs/renovacion-perfil/README.md`, `decisiones.md`, este archivo.
- **Fuentes:** `static/fonts/*.woff2` (ver tabla de la sección 1 de esta corrección).

No se encontró ningún cambio decorativo o estructural que no se pudiera justificar como consecuencia directa de tipografía, espaciado, líneas o accesibilidad — no se revirtió nada por exceder el alcance porque no se encontró exceso. No se tocaron rutas, contenido, contratos de datos (Sheets/CSV/`localStorage`) ni comportamiento funcional.

### 6. Aclaración del atributo `lang`

En el resumen de cierre de la primera pasada de R02, la frase que llegó truncada al usuario ("...el bloque `lang=\"en...`") se refería a **`<html lang="en">` en `src/app.html:2`** — el único atributo `lang` de todo el código fuente (confirmado por `grep -rn "lang=" src/`). Es un código BCP 47 válido en sí mismo ("en" = inglés), pero semánticamente no corresponde: el contenido del sitio es en español. No se corrigió, ni en la primera pasada ni en esta: `decisiones.md` lo registra como parte de un bloque de metadatos pendiente (junto con `<title>`, description, Open Graph), y las instrucciones de esta ronda excluyen explícitamente tocar "SEO, JSON-LD o metadatos generales". Sigue sin resolverse, a la espera de una ronda futura o de autorización explícita.

### Pendiente de aprobación visual

Ninguno de los puntos anteriores se ha visto en un navegador real dentro de esta sesión (ver «Limitación real de método» más arriba, que sigue aplicando sin cambios). Todo lo descrito en esta sección se verificó leyendo el código fuente, el CSS compilado en `build/` y el HTML servido por `npm run preview` vía `curl` — nunca visualmente. **No se afirma que la corrección quede visualmente aprobada hasta que el responsable del proyecto la revise en un navegador.**

## Segunda corrección aplicada tras revisión del usuario (tercera pasada, antes de aprobar R02)

El responsable del proyecto observó que el ámbar usado en la navegación (`--color-accent-ink`, oscurecido para contraste) **no coincidía** con el ámbar que ya se ve como subrayado bajo los `h1` de las páginas interiores (`--color-accent`, sin oscurecer). Pidió reutilizar exactamente ese segundo valor, sin crear ni oscurecer una variante nueva.

### 1. Amarillo reutilizado exactamente, sin variante

- **Regla localizada:** el subrayado de cada `h1` interior usa `border-bottom: var(--line-thick) solid var(--color-accent);` (confirmado en `experiencia`, `programacion`, `publicaciones/articulos`, `publicaciones/libros`, `publicaciones/capitulos` — cinco archivos, misma variable). El valor de `--color-accent` es **`#f4ba00`**, sin oscurecer.
- **Corrección:** en `src/app.css`, la regla centralizada de "Estados de navegación" (`nav a`, `.link`) pasó de `color: var(--color-accent-ink)` a `color: var(--color-accent)` en `:hover`, `[aria-current="page"]` y `:focus-visible`; el contorno de `:focus-visible` pasó de `outline: 1px solid var(--color-accent-ink)` a `outline: 1px solid var(--color-accent)`. Es la misma variable, el mismo valor exacto, sin derivar nada nuevo.
- **Sin cambios de comportamiento más allá del color:** sigue sin haber subrayado, borde, fondo ni cambio de tamaño en ningún estado de nav; el contorno de foco sigue atado a `:focus-visible` (desaparece al perder el foco, no queda residual tras clic de ratón) y sigue sin confundirse con el color permanente de página actual (ambos usan el mismo color ahora, por diseño: es el mismo ámbar en los dos casos).
- **Contraste — dato registrado, no bloqueante:** `#f4ba00` sobre los fondos claros del sitio (`#e6ebf1`/`#dbe6e6`) mide ~1.44:1, por debajo del mínimo AA (4.5:1 texto normal). Es la misma cifra que ya aplicaba al subrayado de los `h1` (ahí como línea decorativa, no sujeta al mismo criterio de contraste de texto, pero con la misma intensidad visual). Se documenta en `src/app.css` para que quede registrado; no se bloqueó el cambio porque es una decisión de identidad visual explícita del responsable del proyecto, tomada con esa cifra ya conocida (se le presentó en la corrección anterior).

### 2. `--color-accent-ink` — se revisó su uso, no se eliminó

Se comprobó con `grep -rn "color-accent-ink" src/` si el token quedaba huérfano tras retirarlo de la navegación. **Sigue teniendo uso legítimo fuera del alcance de esta corrección:** `.back-link:hover`/`:focus-visible` (flecha "volver" en Artículos/Libros/Capítulos) y `.title-link:hover h2`/`:focus-visible h2` (título de proyecto en Programación) — ninguno de los dos es "el nombre del sitio, el menú principal, las pestañas o el indicador de página actual" que pidió esta corrección, y la instrucción fue explícita en no tocar nada más. Por tanto **se conserva el token y sus cuatro usos actuales, sin modificarlos**; solo se actualizó el comentario en `src/app.css` para que documente con precisión dónde sigue en uso ahora que ya no cubre la navegación.

### 3. Corrección de `lang`

`src/app.html:2` cambió de `<html lang="en">` a `<html lang="es">`. Es el único atributo `lang` de todo el código (`grep -rn "lang=" src/` sigue devolviendo una sola coincidencia). Con esto se cierra el punto que la sección 6 de la corrección anterior dejaba explícitamente pendiente por estar fuera del alcance original de R02; esta corrección lo autorizó de forma explícita y puntual, sin extenderlo a `<title>`, `<meta description>` ni Open Graph, que siguen fuera de alcance.

### Archivos modificados en esta pasada

`src/app.css` (color de navegación + comentarios), `src/app.html` (`lang`), este documento. Ningún otro archivo.

### Sigue pendiente de aprobación visual

Como en la pasada anterior, este cambio se verificó por código, CSS compilado y HTML servido (`curl` sobre `npm run preview`), no en un navegador real. **No se afirma que quede visualmente aprobado hasta que el responsable del proyecto lo revise.**

## Tercera corrección aplicada tras revisión del usuario (cuarta pasada, antes de aprobar R02)

La pasada anterior hizo que la navegación reutilizara `--color-accent` (#f4ba00) directamente como color de texto — exactamente el mismo valor que el subrayado de los `h1`, sin oscurecer, por pedido explícito del responsable del proyecto en ese momento. En esta pasada, el responsable del proyecto reconsideró: acepta conservar `#f4ba00` como origen conceptual, pero pide derivar de él una variante con contraste real de texto, calculada rigurosamente (OKLCH, no HSL) contra el fondo efectivo real del sitio — en vez de reutilizar la variante `#7a5e00` de la segunda pasada (esa sí era HSL) sin recalcularla, ni el amarillo vivo sin oscurecer de la pasada inmediatamente anterior.

### Fondo efectivo identificado

`<nav>` (`src/routes/+layout.svelte`) no declara `background-color` propia. Se rastreó toda la cadena de ancestros: `<body>` en `src/app.html` no tiene estilo inline; el `<div style="display: contents">` que envuelve `%sveltekit.body%` no genera caja propia (`display: contents`); `body { background-color: var(--color-bg) }` en `src/app.css` es la única declaración de fondo en toda la cadena (`grep -n "background" src/app.css src/app.html src/routes/+layout.svelte` devuelve una sola coincidencia). **Fondo efectivo real: `--color-bg`, `#dbe6e6`.**

### Método de derivación

Se escribió y ejecutó un script Node.js (conversión sRGB → OKLab/OKLCH siguiendo las matrices de Björn Ottosson, las mismas que usa CSS Color 4) para evitar el error de precisión de una derivación manual en HSL:

1. `#f4ba00` → OKLCH: **L = 0.8192, C = 0.1677, H = 85.83°**.
2. Con H fijo en 85.83° (0.00° de desviación — el hue no se toca en ningún paso), se redujo L en pasos de 0.0005, y en cada paso se recalculó el **chroma máximo dentro del gamut sRGB** para ese L/H exactos (búsqueda binaria: el amarillo saturado no es representable en sRGB en todas las combinaciones de L/H — el propio gamut fuerza a bajar el chroma según se oscurece, no es una elección de diseño).
3. Se detuvo en el primer L (el más luminoso posible) cuyo resultado alcanza contraste ≥ 4.5:1 contra `#dbe6e6`.

### Resultado

| | Valor |
|---|---|
| Color de origen | `#f4ba00` |
| Fondo calculado | `#dbe6e6` (luminancia relativa 0.7737) |
| OKLCH de origen | L 0.8192, C 0.1677, H 85.83° |
| OKLCH derivado | L 0.5122, C 0.1049 (62.5% del croma original, recortado por el gamut), H 85.83° |
| Color derivado | **`#816100`** |
| Luminancia relativa derivada | 0.1328 |
| Contraste final vs `#dbe6e6` | **4.52:1** (4.50:1 antes de redondear a 8 bits — se verificó que el valor de 8 bits final sigue cumpliendo el mínimo) |
| Diferencia de hue vs origen | **0.00°** en OKLCH; en HSL clásico equivale a ~45° derivado vs ~46° de origen (≈1°, dentro del margen de redondeo a 8 bits) |

El resultado se sigue leyendo como una versión oscurecida del mismo `#f4ba00` (mismo hue, mismo croma hasta donde el gamut lo permite), no como un marrón, naranja u oliva arbitrario. Como referencia de control: el valor `#957100` que el usuario mencionó como aproximación solo alcanza **3.55:1 contra el fondo real** `#dbe6e6` (sí llega a ~4.53:1 contra blanco puro, que no es el fondo real) — confirma por qué no podía reutilizarse sin recalcular contra el fondo efectivo del sitio.

### Tokens finales

- `--color-accent: #f4ba00` — se conserva sin cambios, uso exclusivamente decorativo/gráfico (subrayado de `h1`, líneas). **No se tocó ningún subrayado de título interior.**
- `--color-accent-text: #816100` — nuevo token, reemplaza a `--color-accent-ink`. Se usa en: nombre del sitio (`:hover`), enlaces del menú principal (`:hover`, `[aria-current="page"]`), pestañas de Publicaciones (`:hover`), y `:focus-visible` de todos los anteriores (color + contorno de 1px del mismo tono — el amarillo vivo no llega ni a 3:1 contra el fondo real, así que el contorno tampoco lo usa).
- **`--color-accent-ink` se elimina.** Se migraron sus cuatro usos restantes (`.back-link:hover/:focus-visible` en Artículos/Libros/Capítulos, `.title-link:hover h2/:focus-visible h2` en Programación) a `--color-accent-text`: no tenían una necesidad de contraste distinta de la de la navegación (mismo tipo de fondo, mismo tipo de estado hover/foco sobre texto), así que mantener los dos tokens habría dejado dos ámbares oscuros casi idénticos (`#7a5e00` vs `#816100`) sin ninguna función que los distinguiera — exactamente lo que se pidió evitar. Confirmado por `grep -rn "color-accent-ink" src/`: cero declaraciones activas, solo queda la mención en el comentario de `app.css` que explica la migración.

### Archivos modificados en esta pasada

`src/app.css` (token `--color-accent-text` sustituye a `--color-accent-ink`; reglas de "Estados de navegación" actualizadas; comentarios reescritos con la derivación completa), `src/routes/programacion/+page.svelte`, `src/routes/publicaciones/articulos/+page.svelte`, `src/routes/publicaciones/capitulos/+page.svelte`, `src/routes/publicaciones/libros/+page.svelte` (migración de `.back-link`/`.title-link` al nuevo token), este documento. Ningún otro archivo.

### Sigue pendiente de aprobación visual

Igual que en las pasadas anteriores: esta corrección se verificó por cálculo (script Node.js con las matrices OKLab estándar), CSS compilado y HTML servido, no en un navegador real. **No se afirma que el resultado quede visualmente aprobado hasta que el responsable del proyecto lo revise.**

## Tercera corrección conceptual tras revisión del usuario

Tras ver el sitio en un navegador real por primera vez, el responsable del proyecto identificó que el problema de fondo no era (solo) el color de la navegación, sino algo más estructural: **R02 había cambiado la superficie general del sitio por un azul más oscuro que el original y había creado contrastes visibles entre distintas superficies azuladas** (fondo de página, panel de contenedor, líneas de separación), alejando el resultado de la dirección deseada — un sitio blanco, plano, aireado y editorial, con azul petróleo solo en texto/líneas y amarillo solo como acento. Los referentes conceptuales que se reafirman en esta corrección son Caja Negra Editora (tipografía, aire, líneas, interacción) y Media Archaeology Lab (carácter de archivo, colección, artefacto, ficha contextual) — ninguno copiado literalmente.

### 1. Colores recuperados directamente del checkpoint `45ae0fd`

Por instrucción expresa, no se recalculó ni se aproximó ningún valor: se leyeron directamente del código anterior a R02 (`git show 45ae0fd:src/app.css`, `git show 45ae0fd:src/lib/styles/structural.css`).

| Token | Valor exacto en `45ae0fd` | Uso en esa versión |
|---|---|---|
| Fondo general | `#dbe6e6` | `body { background-color }` |
| "Panel" de contenedor | `#e6ebf1` | `.container { background-color }` (páginas interiores) y `.home-grid { background-color }` (portada) |
| Texto/azul petróleo oscuro | `#054f6d` | `body { color }`, y color de texto en general |
| Amarillo original | `#f4ba00` | Color de `h1`, iconos del nav (`!important`), `.link` (pestañas de Publicaciones) y `.back-link`, todos en **reposo**, no solo en hover |

Hallazgo relevante: **`--color-bg` (`#dbe6e6`) y `--color-ink` (`#054f6d`) nunca se habían tocado durante todo R02** — son idénticos, byte a byte, a los de `45ae0fd`. El problema no era que estos dos valores hubieran cambiado, sino que R02 **generalizó el uso del segundo tono** (`#e6ebf1`, el "panel") como color de relleno estructural de `.container` y `.home-grid`, y además añadió líneas de separación (borde inferior del nav) justo en el punto donde ese panel se encuentra con el fondo general — convirtiendo una diferencia de tono casi imperceptible en el código original en una costura visible entre "azul de página" y "azul de caja" una vez vista con líneas y espaciados reales en un navegador.

### 2. Retorno a una base casi blanca sin panel estructural

- **Se elimina `--color-bg-panel` (`#e6ebf1`) como color de relleno.** Ya no se usa como `background-color` en ningún lugar: ni `.container` (páginas interiores) ni `.home-grid` (portada) tienen fondo propio; ambos se apoyan directamente en el `--color-bg` de `<body>`. Un solo fondo en todo el sitio, no una sucesión de azules.
- **Azul petróleo oscuro (`--color-ink`, `#054f6d`) para texto y para líneas/bordes.** `.container` pasó de `background-color` a `border: 1px solid var(--color-line)` (`--color-line` ya era, desde antes de esta corrección, una variante translúcida directa de `--color-ink` — no un tono nuevo). Las cajas ahora se leen como **fichas/marcos/documentos** delimitados por una línea fina, no como tarjetas rellenas de una interfaz comercial.
- **`--color-accent` (`#f4ba00`) vuelve a ser el único amarillo del sitio**, sin variante oscurecida para ningún uso. Se retira `--color-accent-text` (la variante derivada en OKLCH de la corrección anterior) y se migran sus cuatro usos (nav, pestañas, `.back-link`, `.title-link`) al amarillo vivo original. El contraste real (~1.39:1 contra `#dbe6e6`) se documenta en el propio `src/app.css`, sin iniciar una nueva búsqueda matemática de variantes, por instrucción expresa.
- **Tipografía sin cambios de fondo:** IBM Plex Sans en navegación, títulos y lectura; IBM Plex Mono solo en metadatos; sin IBM Plex Serif en ningún punto (esto ya estaba resuelto desde la corrección anterior y no se tocó).

### 3. Portada: espacio en vez de líneas

- **Se retira el `border-bottom` del `<nav>`** (en `src/routes/+layout.svelte`): ya no hay ninguna línea separando la navegación del resto del sitio, en ninguna página. La separación nav→contenido se resuelve con espacio, no con un trazo.
- **`.home-grid` gana `padding-top: var(--space-8)`** (4rem) antes de que empiece la retícula — un salto de aire claramente mayor que el que había, en vez de una costura de color o una línea.
- La retícula "flota" directamente sobre el fondo general, sin caja exterior ni separador propio.

### 4. Retícula: de altura variable a celdas regulares

La versión anterior de esta ronda (antes de esta corrección) usaba una grilla de altura variable (`align-items: start`, sin `grid-auto-rows` fijo) para que cada imagen conservara su proporción natural sin recortes. El usuario revisó esa solución visualmente y pidió recuperar la **regularidad simétrica** del `45ae0fd` original, que usaba `grid-auto-rows: 250px` fijos con las imágenes centradas y recortadas por `overflow: hidden` dentro de cada celda — es decir, **el original ya recortaba imágenes para mantener celdas regulares**; no es una técnica nueva de esta corrección, es la que ya existía antes de R02.

Técnica definitiva: `.card-image` con `aspect-ratio: 4 / 3` (en vez de una altura fija en píxeles, que se distorsiona en anchos de viewport muy distintos) + `img { object-fit: cover }`. Se aplicó igual en portada (4/3/2 columnas según ancho) y en la grilla de Publicaciones (3/2 columnas), incluida la regla `.card:last-child` que centra el elemento sobrante cuando el total no es múltiplo de 3 (se conserva, con su anulación en el breakpoint móvil de 2 columnas). No se usó `column-count`/multi-columna ni ninguna librería de masonry ni JavaScript de posicionamiento: sigue siendo `display: grid` con auto-flow por defecto, así que el orden visual, el orden del DOM y el orden de tabulación coinciden (mismo hallazgo que en la corrección anterior, que sigue aplicando). Los datos y su orden estable por sesión (semilla en `sessionStorage`, de R01) no se tocaron.

Nota honesta: esta técnica **sí recorta** las imágenes para forzar la misma proporción en toda celda — igual que hacía el original. Esto revierte, para la retícula de portada y Publicaciones, la decisión R00 nº 3 ("igualdad conceptual de los elementos de la grilla, respetando que las imágenes conserven proporciones distintas"): por instrucción explícita de esta corrección, la regularidad de la retícula pesa más que la conservación de la proporción individual de cada imagen. Se registra aquí como una decisión revertida, no como un descuido; **queda pendiente reflejarlo en `decisiones.md`**, que esta pasada no tocó (no estaba entre los archivos que esta corrección pidió actualizar).

### 5. Contextualización de imágenes: título y autor

Cada celda de la retícula (portada y Publicaciones) puede ahora mostrar, superpuesto sobre la propia imagen dentro de la misma celda (nunca desplazando la retícula), el título y el autor/atribución del ítem, cuando el dato existe:

- **Escritorio (con hover real):** estado inicial en gris, sin ficha visible; en `:hover` y `:focus-visible` la imagen recupera el color y aparece una veladura (degradado de `rgba(5,49,68,0.92)` a transparente) con el título y el autor superpuestos — una sola transición de opacidad/filtro, sin escala, sombra ni movimiento, y sin desplazar el grid.
- **Teclado:** la misma información aparece con `:focus-visible` sobre el enlace contenedor — no depende solo del hover.
- **Táctil/puntero grueso:** la ficha es **persistente** (no depende de ninguna media query de hover): se ve siempre, igual que la imagen ya se muestra en color desde el inicio en estos dispositivos (regla heredada de la corrección anterior). No hace falta tocar dos veces para seguir el enlace.
- **Nombres accesibles:** el `alt` de cada imagen pasa a usar el título real cuando existe (`item.titulo || `Publicación ${item.indice}``), mejorando el nombre accesible del enlace que la envuelve (antes era siempre el genérico "Publicación N").
- **Si falta el autor, se muestra solo el título; si falta también un título útil, no se muestra ninguna ficha** (no se inventa texto): la ficha completa está condicionada a `{#if item.titulo || item.autor}`.

**Limitación real, declarada explícitamente:** el contrato de datos actual (lo que el código ya leía antes de esta corrección) solo usaba `item.indice`, `item.link` e `item.imagen` para estas dos grillas — no había ninguna referencia previa a un campo de título o autor en `src/routes/+page.svelte` ni en `src/routes/publicaciones/+page.svelte`. Esta corrección **asume** que las hojas de cálculo correspondientes tienen columnas `titulo`/`autor` (minúsculas, sin tilde, según la misma convención que ya usan estas dos hojas para `indice`/`link`/`imagen` — distinta de la convención capitalizada `Título`/`Autor` que sí usan las hojas de Artículos/Libros/Capítulos). **No se verificó contra las hojas de Sheets reales** (no se hizo ninguna llamada de red a Google Sheets desde esta sesión): si esas columnas no existen con ese nombre exacto, `item.titulo`/`item.autor` serán `undefined` y la condición `{#if item.titulo || item.autor}` hará que ninguna ficha se muestre — no se inventa contenido, pero tampoco se ve el efecto hasta que el responsable del proyecto confirme o ajuste el nombre real de esas columnas en la hoja.

### 6. Páginas internas: fichas delimitadas por línea, no por color

`.container` (perfil, experiencia, publicaciones y sus tres subsecciones, programación) pasa de tener un `background-color` propio a un `border: 1px solid` en la misma variante translúcida de azul petróleo que ya usaban los separadores entre fichas de listado. El resultado: contenido sobre el mismo fondo general del sitio, enmarcado por una línea, no por una superficie de color distinta. Las fichas de listado (`.list-card`, `.card` de Experiencia) ya usaban este mismo criterio desde antes de esta corrección (no tenían `background-color` propio, solo la línea entre entradas consecutivas), así que no requirieron cambios adicionales.

### 7. Elementos de R02 conservados (no revertidos)

IBM Plex Sans como tipografía general; IBM Plex Mono limitado a metadatos; `lang="es"`; `aria-current` con color como señal de página actual; `aria-label` en enlaces de solo-ícono; `min-height: 44px` en enlaces de navegación/pestañas; orden DOM correcto en las retículas; comportamiento táctil sin doble toque; ausencia de `transform: scale()` y `box-shadow` decorativos; ancho de lectura (`--measure: 68ch`) en perfil y descripciones; configuración de GitHub Pages, adaptador estático y workflow (no tocados en ninguna pasada de R02); estrategia de carga desde Google Sheets vía `d3-fetch`/CSV y las claves de `localStorage`/`sessionStorage` existentes; todas las rutas actuales.

### Archivos modificados en esta pasada

`src/app.css` (retiro de `--color-bg-panel` y `--color-accent-text`, navegación vuelve a `--color-accent`), `src/lib/styles/structural.css` (`.container`: fondo → borde), `src/routes/+layout.svelte` (retiro del `border-bottom` del nav), `src/routes/+page.svelte` (retícula regular con `aspect-ratio`/`object-fit`, ficha de título/autor, espacio antes de la retícula), `src/routes/publicaciones/+page.svelte` (mismo tratamiento de retícula y ficha), `src/routes/programacion/+page.svelte`, `src/routes/publicaciones/{articulos,libros,capitulos}/+page.svelte` (migración de `.back-link`/`.title-link` a `--color-accent`), este documento.

### Pendiente señalado, no resuelto en esta pasada

`docs/renovacion-perfil/decisiones.md` sigue reflejando el estado de la corrección anterior (menciona `--color-accent-ink`/`--color-accent-text` y no registra la reversión de la decisión R00 nº 3 sobre proporciones distintas de imagen). Esta corrección no lo tocó porque la instrucción de esta pasada solo pidió actualizar este documento de ronda; queda pendiente sincronizarlo.

### Sigue pendiente de aprobación visual

**R02 continúa sin cerrarse.** Esta corrección se verificó por código, CSS compilado y HTML servido (`curl` sobre `npm run preview`), no en un navegador real dentro de esta sesión. No se afirma que la composición, el espaciado ni el efecto de la ficha de imagen queden visualmente aprobados hasta que el responsable del proyecto los revise directamente.

## Cuarta corrección: revisión integral de datos, caché y recomposición visual

Esta pasada respondió a un reporte de revisión visual real del usuario (portada con una sola imagen, `/publicaciones` sin sus contenidos, registros nuevos ausentes en Artículos, discrepancias entre GitHub Pages/Vercel/local, sitio aún "demasiado azul", encabezado poco presente, retícula sin simetría, espaciado interior insuficiente). Se pidió explícitamente inspeccionar el recorrido completo de los datos antes de seguir ajustando el diseño, y no seguir aplicando microcorrecciones aisladas.

### Diagnóstico de datos (verificado contra las hojas reales, no inferido)

Se descargaron las seis hojas publicadas (`curl -L` sobre cada URL `pub?output=csv`) y se parsearon con el mismo parser que usa el código (`d3-dsv`, la base de `d3-fetch`) para separar hechos de hipótesis:

| Hoja | Columnas reales | Filas actuales |
|---|---|---|
| Portada (grilla) | `indice, imagen, link` | 16 |
| Publicaciones (grilla) | `indice, imagen, link` | 9 |
| Artículos | `Autor, Título, Revista, Volumen, Número, Año, DOI, Páginas, Descripcion, Imagen` | 6 |
| Libros | `Autor, Título, Ciudad, Editorial, Año, ISBN, DOI, Descripcion, Imagen` | 2 |
| Capítulos | `Autor, Título, Título_libro, Editores, Ciudad, Editorial, Año, Páginas, Imagen, Descripcion, DOI` | 4 |
| Experiencia | `Cargo, Institución, Periodo, Descripción` | 11 |
| Programación | `titulo, tipo, fecha, link, descripcion, imagen, autor` | 7 |

**Hallazgo 1 — las columnas coinciden exactamente con lo que el código ya esperaba**, en las siete hojas (nombre, mayúsculas y tildes incluidos). No hay ningún caso de columna mal interpretada ni de nombre distinto entre lo que el código pide y lo que la hoja entrega. Esto descarta "columnas mal interpretadas" como causa de cualquiera de los síntomas reportados.

**Hallazgo 2 — la portada y la grilla de Publicaciones no tienen columnas de título ni autor.** Solo `indice`, `imagen` y `link`. Una pasada anterior de R02 había asumido campos `item.titulo`/`item.autor` para estas dos grillas **sin verificarlo contra la hoja real** — la condición `{#if item.titulo || item.autor}` simplemente nunca se cumplía y la ficha contextual nunca llegó a mostrarse, pero el supuesto quedaba en el código como si fuera válido. Se retira por completo (ver «Contextualización de imágenes» más abajo).

**Hallazgo 3 — un registro roto en la hoja de imágenes (fila `indice=4`, compartida por portada y Publicaciones):** el valor de la celda "imagen" contiene un fragmento de etiqueta HTML pegado por error (`..._k.jpg" width="2048" height="1270" alt="Italy V55 18 1586 (2 of 2)`), con comillas sin escapar. El parser `d3-dsv` **no falla** con esto (se probó explícitamente: sigue devolviendo las 16/9 filas), pero el campo `imagen` de esa fila queda con comillas literales dentro de la URL, que el navegador no puede resolver como imagen válida — esa fila en concreto mostraría una imagen rota. Es un dato mal cargado en la hoja, no un error de parseo ni de código; no se edita la hoja (fuera de alcance), se filtra defensivamente en el cliente (ver más abajo).

**Hallazgo 4 — no se encontró causa de datos para "solo aparece una imagen" en la portada.** Con los 16 registros reales, claves (`indice`) únicas y sin vacíos, parseo correcto, CORS correcto (`access-control-allow-origin: *` confirmado en la respuesta final tras redirección) y el `<script>` de `src/routes/+page.svelte` **byte a byte idéntico** al de `45ae0fd` (confirmado con `git diff 45ae0fd -- src/routes/+page.svelte`: ningún cambio toca las líneas 1–54, solo el `<style>` y el marcado de la ficha) — no hay una causa reproducible en datos ni en la lógica de carga. Como ninguna versión de R02 ha tenido `push` ni despliegue, esto no puede corresponder al *preview* local revisado en esta sesión con código nuevo: si se observó en `juanfuc.github.io`/Vercel, corresponde al sitio publicado tal como estaba antes de esta ronda, no a un cambio introducido aquí. Queda como algo que **debe confirmarse visualmente por el responsable del proyecto** después de este pase, no como un hallazgo cerrado.

**Hallazgo 5 — causa confirmada de "los registros nuevos no aparecen" y de la discrepancia entre dominios:** `localStorage` sin expiración. Antes de esta pasada, Publicaciones/Artículos/Libros/Capítulos guardaban los datos en `localStorage` la primera vez y **nunca volvían a pedirlos** mientras existiera esa clave — ni Experiencia/Programación, que expiraban cada hora pero mientras tanto también servían la copia vieja. Esto ya estaba señalado como problema desde el diagnóstico inicial del proyecto (R00), antes de que existiera R02. Como `localStorage` es por origen (dominio), cada despliegue (GitHub Pages, Vercel, `localhost`) acumula su propia copia cacheada de forma independiente y en momentos distintos — exactamente el patrón descrito ("GitHub Pages mostró datos más recientes mientras Vercel y local seguían con información antigua").

### Estrategia de caché implementada

Red primero, `localStorage` únicamente como respaldo si la red falla — la misma estrategia en las seis páginas, reemplazando tanto el "cachear para siempre" como la expiración por tiempo:

```js
try {
  const fetchedData = await csv(url);
  data = fetchedData;
  localStorage.setItem(CACHE_KEY, JSON.stringify(data));
} catch (error) {
  console.error("Error fetching data:", error);
  const cachedData = localStorage.getItem(CACHE_KEY);
  if (cachedData) data = JSON.parse(cachedData);
} finally {
  isLoading = false;
}
```

Efecto: cada carga intenta la hoja real; si la red falla (sin conexión, Sheets caído), se usa la última copia buena conocida en vez de dejar la página vacía; si la red funciona, la copia local se actualiza en el acto. No hace falta que nadie borre `localStorage` a mano, no depende de temporizadores ni de versión de caché, y se comporta igual en desarrollo, Vercel y GitHub Pages porque es lógica de cliente pura, sin *service worker* ni dependencia nueva. Se aplicó tal cual en Publicaciones, Artículos, Libros y Capítulos; en Experiencia y Programación además se retiró la lógica de expiración por hora (`CACHE_DURATION`/`*Timestamp`) y los `console.log` de depuración que traía, por quedar redundantes con el nuevo modelo.

### Filtro defensivo de imágenes corruptas

En portada y Publicaciones (las dos grillas que comparten el problema de la fila `indice=4`), se añadió `isUsableImage(url)` que descarta cualquier fila cuyo campo `imagen` esté vacío o contenga un carácter `"` literal (síntoma de una celda con HTML pegado por error) antes de renderizarla. No se modifica la hoja ni se inventa una imagen de reemplazo: la fila simplemente no se muestra hasta que se corrija en origen.

### Contextualización de imágenes — corrección del supuesto anterior

Sin columnas de título/autor reales en portada/Publicaciones, se retiró toda referencia a `item.titulo`/`item.autor` en esas dos grillas. En su lugar, `sourceLabel(url)` extrae el dominio del `link` (dato que sí existe siempre) — p. ej. `doi.org`, `observablehq.com` — y se muestra como ficha mínima en la esquina inferior de la imagen, con el mismo comportamiento de aparición que antes (gris→color + veladura en `:hover`/`:focus-visible`, persistente en táctil, sin mover la retícula). El `aria-label` del enlace pasa a `"Ver referencia en {dominio}"` para dar un nombre accesible real. **Esto no es título ni autor**: es el contexto mínimo disponible con los datos existentes. Se documenta como carencia real de contenido, no se resuelve inventando texto. Artículos, Libros y Capítulos sí tienen `Título`/`Autor` reales en su hoja y ya los mostraban correctamente desde antes; no se tocó esa parte.

### Recomposición del encabezado

Se pasó de una fila única (nombre a la izquierda, menú a la derecha) a una composición apilada: identidad (nombre + iconos) y luego menú, cada uno en su propia fila, con el nombre notablemente más grande (`clamp(1.85rem, 1.3rem + 2.6vw, 3.25rem)`, peso 700, antes tope de `1.6rem`/600) y el menú con más separación entre enlaces y tamaño de texto más legible (`--fs-ui` en vez de `--fs-meta`). El espacio antes de la retícula ya no depende solo del `padding-top` de `.home-grid` (se redujo a `--space-6`): el propio `<nav>` gana padding vertical generoso (`--space-7`/`--space-6`) como parte de su construcción, tal como se pidió.

### Estado de las validaciones

`npm run build` sin errores ni advertencias; `git diff --check` sin errores reales (solo avisos de fin de línea de Windows); las 8 rutas devuelven HTTP 200 en `npm run preview`; no se tocaron rutas, `svelte.config.js`, `package.json`, `.github/workflows/`, ni columnas de ninguna hoja (confirmado con `git diff --stat` contra esos archivos: sin cambios). No hubo navegador disponible en esta sesión para confirmar visualmente la composición, la retícula ni el efecto de las fichas — ver «Limitaciones pendientes» a continuación.

### Limitaciones pendientes (reales, no resueltas aquí)

1. **"Solo una imagen en portada" no se pudo reproducir** por datos, parseo, CORS ni lógica de carga (ver Hallazgo 4). Requiere que el responsable del proyecto confirme si el síntoma persiste en el *preview* de esta pasada; si persiste, es un problema distinto al que este diagnóstico pudo cubrir sin navegador.
2. **La fila `indice=4` de la hoja de imágenes sigue rota en origen** (Google Sheets): se filtra en el cliente, pero la corrección real es editar esa celda en la hoja, fuera del alcance de esta ronda.
3. **Portada y Publicaciones no tienen título/autor**: si se quiere contextualización real (no solo el dominio del enlace), hace falta añadir esas columnas a las hojas correspondientes — una decisión de contenido, no de código.
4. **`docs/renovacion-perfil/decisiones.md`** se actualiza en esta pasada solo en lo estrictamente necesario para reflejar el fondo/tipografía/caché vigentes; no se reabre ni se re-decide nada de lo ya confirmado en rondas anteriores.
5. Como en todas las pasadas anteriores, nada de esto se ha visto en un navegador real dentro de esta sesión. No se afirma aprobación visual.

## Revisión final propia (autor de la implementación, no revisor externo)

Se releyeron los 12 archivos modificados completos tras el build final, buscando específicamente: selectores CSS huérfanos o mal targeteados, tokens sin definir, `!important` residual, y coherencia entre lo que cada archivo dice hacer y lo que hace. Un ajuste se corrigió en esta misma pasada: `align-items: baseline` en `nav .left` se cambió a `center` antes de dar por cerrada la ronda, por ser una alineación que no se pudo verificar visualmente y para la que `center` es la opción más predecible sin navegador disponible.

**Esta revisión no sustituye a la revisión independiente.** No se aprueba ni se cierra la ronda en este documento: queda "implementada — pendiente de revisión", como pide la instrucción de esta sesión.

# Decisiones

Registro vivo. Separa lo que ya está **cerrado** de lo que está **abierto**. Al cerrar una ronda, mueve lo validado de «pendientes» a «confirmadas» con fecha/ronda.

---

## Decisiones confirmadas

Acordadas y vinculantes para el trabajo. (Se corresponden con los acuerdos del `brief.md`.)

1. **El sitio es, prioritariamente, un perfil académico personal.** (R00)
2. **La grilla es un índice visual de descubrimiento**, no una galería ni un portafolio. (R00)
3. **Igualdad conceptual de los elementos de la grilla**, respetando que las imágenes conserven **proporciones distintas**. (R00)
4. **La combinación de la grilla es aleatoria pero estable durante una sesión.** (R00)
5. **La grilla pertenece a la portada (`/`) y no antecede a las páginas interiores.** (R00)
6. **Se conservan la paleta actual y el efecto de gris a color.** (R00)
7. **La renovación es pragmática:** aprovecha contenidos e infraestructura existentes. (R00)
8. **«En curso» es una capa secundaria para proyectos e investigaciones en desarrollo.** Sus proyectos pueden agrupar y conectar publicaciones, clases, convocatorias, residencias, materiales u otros trabajos; pueden superponerse y relacionarse con elementos de la grilla. (R00)
9. **Caja Negra es el referente concreto, pero parcial:** solo para tipografía, líneas, composición plana y estados de interacción; no como plantilla general. (R00)
10. **El destino de producción será `https://juanfuc.github.io/`.** (R01)
11. **Vercel se conserva temporalmente como respaldo** hasta verificar que GitHub Pages sirve correctamente el sitio. (R01)
12. **R01 incluye la estabilización estructural necesaria antes del rediseño visual** (grilla confinada a portada, revelado del perfil, estabilidad de sesión, deuda de CSS/accesibilidad/código muerto), ampliando lo que su documento de ronda tenía como «fuera de alcance» para código muerto y accesibilidad avanzada. (R01)
13. **La estabilidad del orden de la grilla se implementa con una semilla en `sessionStorage`**, persistente durante recargas de la misma pestaña y renovable en una sesión nueva. (R01)
14. **GitHub Pages se publica mediante `adapter-static` y GitHub Actions**, con salida en `build/` y fallback `404.html`. (R01)
15. **Combinación tipográfica: IBM Plex Sans en todo el sitio (sin serifas), autohospedada; IBM Plex Mono reservado a metadatos/registro.** [Corregido tras revisión del usuario: la primera implementación de R02 incluía IBM Plex Serif para cuerpo de texto y títulos; se retiró por completo — el usuario no quiere tipografías con serifa en ningún punto del sitio. La jerarquía entre navegación, títulos y cuerpo se resuelve con peso/tamaño/espaciado, no con cambio de familia.] Carga real vía `@font-face` con archivos `.woff2` propios (no CDN de Google Fonts en producción). Ver [`rondas/R02-sistema-visual.md`](./rondas/R02-sistema-visual.md). (R02)
16. **Sistema de líneas:** líneas finas (1px, tinta azul petróleo al 22%) como separador de nav, entre entradas de listado consecutivas y sobre la fila de autor; línea gruesa (2px, ámbar) como subrayado corto bajo cada `h1`. (R02)
17. **Composición plana aplicada de forma literal:** sin `border-radius` ni `box-shadow` en ningún componente. (R02)
18. **Estados de interacción resueltos:** se elimina `transform: scale()` y `box-shadow` decorativos (sin alternativa de teclado); se conserva el efecto gris→color como único mecanismo de hover, extendido a `:focus-visible` para paridad de teclado. En dispositivos sin puntero preciso (`(hover: hover) and (pointer: fine)` como condición) las imágenes se muestran directamente en color, sin filtro gris inicial. (R02)
19. **Se elimina el aumento de escala en hover** de las imágenes de grilla/ficha (antes registrado como «posible eliminación», pendiente de validación). (R02)
20. **Estados de navegación (nombre, menú principal, pestañas de sección, página actual): solo color, sin subrayado ni borde.** [Superado por revisión posterior del usuario: se probaron primero un tono oscurecido (`#7a5000`, luego `#7a5e00` recalculado en OKLCH) para cumplir contraste de texto; el usuario prefirió finalmente el amarillo vivo original sin oscurecer.] **Valor definitivo: `--color-accent` (`#f4ba00`), sin variante oscurecida — el mismo tono en reposo decorativo (subrayado de `h1`) y en los estados de navegación.** Reposo en tinta azul petróleo; hover, foco de teclado y página actual en `--color-accent`. Contraste real ~1.66:1 contra el fondo del sitio (`#f5f8f9`, ver punto 22), por debajo del mínimo AA — documentado y aceptado explícitamente, no resuelto con una nueva variante. El foco de teclado añade además un contorno de 1px del mismo tono (no un color ajeno), visible solo mientras dura el foco. Reglas centralizadas en `src/app.css` para que no diverjan entre archivos. Los tokens intermedios (`--color-accent-ink`, `--color-accent-text`) se eliminaron del código: no tienen uso vigente. (R02)
21. **Retícula de portada y de Publicaciones: celdas regulares (`aspect-ratio` + `object-fit: cover`), no altura variable.** [Revierte la decisión R00 nº 3 ("igualdad conceptual... proporciones distintas") para estas dos grillas, por pedido explícito del usuario tras revisión visual: prioriza la simetría de la retícula sobre la conservación de la proporción individual de cada imagen — como ya hacía el propio `45ae0fd` con `grid-auto-rows` fijos y recorte por `overflow: hidden`.] (R02)
22. **Fondo general: `--color-bg: #fbfcfc`, no `#dbe6e6` ni `#f5f8f9`.** [Dos correcciones sucesivas tras revisión visual del usuario: `#dbe6e6` se percibía como una superficie azul; la primera corrección, `#f5f8f9`, seguía leyéndose con demasiado tinte azul. `#fbfcfc` es el valor definitivo.] Un solo fondo en todo el sitio, sin panel/superficie distinta para contenedores (ver punto 17). (R02)
23. **Caché de datos: copia local primero (si existe, sin esperar red), red siempre en paralelo para refrescar, y si la red falla se conserva lo que ya había en pantalla** ("stale-while-revalidate"), en las siete páginas que leen Sheets y cachean (portada, Publicaciones, Artículos, Libros, Capítulos, Experiencia, Programación). [Superado respecto a la redacción anterior de este punto ("red primero, localStorage solo como respaldo"): esa variante podía dejar la portada sin datos visibles durante el tiempo de red; el esquema definitivo muestra la copia local de inmediato y siempre refresca en segundo plano.] Sustituye tanto el cacheo sin expiración como la expiración por una hora que tenían distintas páginas antes de R02. Ver diagnóstico completo en [`rondas/R02-sistema-visual.md`](./rondas/R02-sistema-visual.md). (R02)
24. **Encabezado horizontal en escritorio** (nombre a la izquierda, menú a la derecha, misma franja, con `flex-wrap` como única adaptación a anchos angostos) y **separación encabezado→contenido unificada en un solo token** (`--space-header-gap`, aplicado una vez como `margin-bottom` de `nav`, sin línea divisoria). [Se probó y se descartó un encabezado apilado en dos filas.] (R02)
25. **Imágenes de la grilla de portada: contexto por cruce de enlaces, no por autoría ni por dominio.** El `link` de cada imagen se normaliza (DOI cuando existe, o URL sin protocolo/mayúsculas/parámetros de tracking) y se cruza contra Artículos, Libros, Capítulos y Programación; con coincidencia, la ficha muestra el título, tipo y año reales del contenido enlazado. [Se probaron y descartaron: usar el dominio del enlace como metadato, y añadir columnas de autoría/título a la propia hoja de imágenes — la ficha describe el contenido enlazado, no la imagen.] Incluye carga progresiva (primeras imágenes con prioridad alta, resto en `lazy`), reserva de espacio con `aspect-ratio`, y una ficha tipográfica de respaldo si una imagen falla al cargar. (R02)
26. **Publicaciones (`/publicaciones`) es un índice bibliográfico editorial** (Artículos + Libros + Capítulos combinados, ordenado por año descendente), no una grilla de imágenes. El acceso a cada categoría (Artículos/Libros/Capítulos) sigue siendo navegación real entre rutas propias, no un filtro en memoria. [Se probó y descartó una grilla aleatoria de imágenes en esta sección, igual a la de portada.] Queda **resuelta**, sin pendientes de diseño. (R02)
27. **Programación creativa y Proyectos de Humanidades Digitales: iframes bajo petición** (nunca cargados de entrada) **y filtro de dos niveles** (Tipo → Subtipo, contextual, en memoria sobre los datos ya cargados). [Se probó y descartó cargar todos los iframes simultáneamente en la carga inicial.] (R02)

---

## Posibles asuntos futuros (no son una ronda activa)

Observaciones registradas al cerrar R02, para una eventual ronda futura — **no constituyen R03** ni ninguna ronda abierta; no se actúa sobre ellas hasta que el responsable del proyecto decida abrir una nueva ronda:

- Revisar la selección editorial de imágenes de portada (qué imágenes representan mejor cada trabajo), como criterio de contenido separado del sistema técnico ya cerrado.
- Ampliar subtipos/etiquetas de Programación a medida que la hoja crezca con nuevos proyectos que no encajen en las categorías actuales.
- Evaluar CDN o miniaturas propias para las imágenes de portada **solo si** la carga nativa (`loading`/`fetchpriority` actuales) resulta insuficiente en el uso real.
- Revisión conceptual futura de otras secciones del sitio (Perfil, Experiencia) no cubiertas por el alcance visual de R02.

## Propuestas pendientes de validación

Ideas y direcciones **no cerradas**. No se implementan hasta ser validadas; cuando lo sean, pasan a «confirmadas».

### Grilla
- **Franja contextual de la grilla.** Pendiente decidir si la portada incorpora una **franja/banda contextual** (texto breve, encabezado o marco que sitúe la grilla como índice de descubrimiento) y, en tal caso, su contenido, posición y relación con la grilla. **No se resuelve en R02** (fuera del alcance dado para esa ronda).

### Otros temas abiertos (registro, aún sin acordar)
- Estrategia para hacer **indexable** el contenido dinámico de Sheets (mover carga de datos a build/servidor vs. seguir en cliente): pendiente; con impacto en SEO.
- **Metadatos generales (`<title>`, description, Open Graph).** El diagnóstico original observaba también `<html lang="en">` en un sitio en español; **eso ya se corrigió** (`<html lang="es">` desde una corrección de R02, autorizada puntualmente por el responsable del proyecto). Sigue pendiente, sin autorización todavía, el resto: `<title>`/description/OG.
- **Destino del build antiguo en `docs/`, de `.vercel/output/` y del gitlink anidado `juanfuc.github/`:** pendiente. El código muerto identificado en R00 fue eliminado en R01.

### Preguntas pendientes (requieren respuesta del responsable del proyecto)
- **[Corregido R00-respuesta, H6] Prioridad de audiencias.** El `README.md` anunciaba que `brief.md` explica «para quién» es el sitio, pero **no existe una prioridad de audiencias confirmada** en el corpus. **No se inventa.** Pregunta abierta: ¿cuál es el público prioritario (pares académicos, comités de evaluación/becas, estudiantes, medios/divulgación, colaboradores de proyectos)? La respuesta condiciona jerarquía de contenidos y metadatos.
- **[Corregido R00-respuesta] Colisión de `docs/`.** La nueva documentación (`docs/renovacion-perfil/`) **convive** en el mismo directorio que un **build antiguo de GitHub Pages** versionado bajo `docs/` (46 archivos: `docs/index.html`, `docs/_app/…`, etc.). Riesgo de confusión y de que un futuro proceso de publicación mezcle ambos. Se registra como **pendiente**; **no se mueve la carpeta sin una decisión confirmada**.

---

## Cómo se registran los cambios

- Al validar una propuesta, muévela a **Confirmadas** anotando la ronda (p. ej. «(R01)») y, si aplica, un enlace al documento de esa ronda.
- Si una decisión confirmada se revierte, muévela de nuevo a **Pendientes** explicando por qué.

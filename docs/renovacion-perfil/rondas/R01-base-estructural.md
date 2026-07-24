# R01 — Base estructural

**Estado:** activa.
**Toca código del sitio:** sí.

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
- **Nueva sesión** (se permite un orden distinto) = **abrir una pestaña/ventana nueva** o **reabrir tras cerrar la pestaña**. El límite de sesión propuesto es `sessionStorage` (persiste entre recargas de la pestaña, se pierde al cerrarla); el mecanismo exacto sigue **pendiente** en `decisiones.md`.

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
4. **El orden aleatorio de la grilla permanece estable durante la sesión**, según la definición de arriba. **Debe probarse explícitamente una recarda (F5) de la misma pestaña**: el orden no cambia. Abrir una pestaña nueva **puede** dar otro orden.
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
- Para la estabilidad por sesión, considerar fijar el orden en el primer render (p. ej. semilla en `sessionStorage`); el mecanismo exacto está **pendiente** en `decisiones.md`.
- Para el perfil, renderizar el texto visible por defecto; si se conserva alguna animación, que sea de mejora progresiva (nunca ocultando el contenido base).

## Fuera de alcance de R01

- Cargar/definir la tipografía definitiva, el sistema de líneas, la aplicación de la composición plana y la franja contextual (pendientes).
- Metadatos/SEO, prerender de datos, accesibilidad avanzada, limpieza de código muerto y de `docs/`: se abordan en rondas posteriores salvo que se decida lo contrario.
- Eliminar o no el aumento de escala en hover: **pendiente**, no se decide en R01.

## Registro de ejecución

> Se completa al implementar la ronda.

- **Línea base capturada (rutas, viewports, conteos):** _pendiente_
- **Cambios realizados:** _pendiente_
- **Archivos afectados:** _pendiente_
- **Resultado de `npm run build` (y `npx svelte-kit sync` opcional):** _pendiente_
- **Verificación de carga directa, navegación interna y recarga (F5):** _pendiente_
- **Comparación contra línea base (escritorio y móvil):** _pendiente_

## Revisión independiente

- **Revisor:** _pendiente_
- **Fecha:** _pendiente_
- **Hallazgos:** _pendiente_
- **Veredicto:** _pendiente_

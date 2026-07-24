# R00 — Documentación inicial

**Estado:** cerrada — aprobada.
**Toca código del sitio:** no. Solo documentación y referencias mínimas en los archivos de instrucciones de agentes.

> Nota de estado: la revisión independiente emitió veredicto **«requiere correcciones»**. La respuesta resolvió ocho hallazgos y la confirmación posterior del responsable del proyecto resolvió H6. R00 queda **aprobada y cerrada**; **R01 pasa a ser la ronda activa**.

## Objetivo

Externalizar el contexto de la conversación de origen para que **cualquier agente o persona pueda continuar el proyecto sin depender de esa sesión**. Dejar establecida la estructura de documentación, el orden de lectura y los acuerdos de dirección.

## Alcance

- Crear la carpeta `docs/renovacion-perfil/` con su puerta de entrada y documentos base.
- Convertir la auditoría técnica/visual previa en `diagnostico-inicial.md`, contrastándola con el repositorio y con el sitio publicado.
- Registrar acuerdos (`brief.md`) y separar decisiones confirmadas de pendientes (`decisiones.md`).
- Describir la implementación actual (`estado-actual.md`).
- Referenciar esta documentación desde `AGENTS.md` y `CLAUDE.md`.

Fuera de alcance: cualquier cambio en el código del sitio.

## Documentos creados

- `docs/renovacion-perfil/README.md` — puerta de entrada y orden de lectura.
- `docs/renovacion-perfil/brief.md` — acuerdos de dirección.
- `docs/renovacion-perfil/diagnostico-inicial.md` — auditoría contrastada (repo vs. live).
- `docs/renovacion-perfil/estado-actual.md` — implementación actual.
- `docs/renovacion-perfil/decisiones.md` — confirmadas vs. pendientes.
- `docs/renovacion-perfil/rondas/R00-documentacion-inicial.md` — este documento.
- `docs/renovacion-perfil/rondas/R01-base-estructural.md` — apertura de la ronda activa.

## Referencias añadidas

- `AGENTS.md` — instrucción de leer `docs/renovacion-perfil/README.md` y la ronda activa antes de trabajar.
- `CLAUDE.md` — misma instrucción.

(Ambos archivos no existían previamente; se crearon solo con esta referencia.)

## Preservación de cambios existentes

Antes de documentar se ejecutó `git status`. Había **cambios sin confirmar** en `src/routes/perfil/+page.svelte` y `src/routes/Perfil.svelte` (reescritura del texto de perfil). **Se preservaron intactos**; R00 no tocó código del sitio. Quedan descritos en `estado-actual.md`.

## Validación

- [x] Existe `docs/renovacion-perfil/` con los siete documentos listados.
- [x] `README.md` indica el orden de lectura: brief → estado-actual → decisiones → ronda activa.
- [x] `diagnostico-inicial.md` distingue **[REPO]**, **[LIVE]** e **[INFERIDO]**.
- [x] Las afirmaciones del diagnóstico se contrastaron con el HTML servido por https://juanfuc.vercel.app/ (shell ~2,8 KB: `lang="en"`, sin `<title>`/description/OG, sin fuentes, grilla vacía).
- [x] `brief.md` registra los nueve acuerdos.
- [x] `decisiones.md` separa confirmadas de pendientes e incluye los pendientes señalados (tipografía exacta, franja contextual, sistema de líneas, posible eliminación del hover-scale).
- [x] `AGENTS.md` y `CLAUDE.md` referencian el README y la ronda activa.
- [x] No se modificó código del sitio; los cambios previos en `/perfil` siguen presentes.

## Limitaciones conocidas

- La verificación del sitio en vivo se hizo sobre el **HTML crudo** (curl), no con inspección visual de navegador. Los juicios puramente estéticos del render quedan como **[INFERIDO]**.

## Revisión independiente

- **Revisor:** Codex (revisión independiente)
- **Fecha:** 2026-07-24
- **Hallazgos:**
  1. **[Corrección necesaria] Alcance del prerender y del contenido en cliente.** `diagnostico-inicial.md` §4 afirma que «cada página» carga su contenido en `onMount` desde CSV y que el HTML de build no incluye el contenido. Esto no se cumple en `/perfil`: `src/routes/perfil/+page.svelte` contiene el perfil de forma estática y el HTML publicado incluye el texto completo, aunque el CSS lo deje visualmente oculto hasta que actúe el JavaScript. Los títulos y enlaces estáticos de las demás rutas también aparecen en el HTML inicial. Debe acotarse la afirmación al contenido dinámico proveniente de Sheets y distinguir la portada prácticamente vacía de las rutas que sí prerenderizan contenido estático.
  2. **[Corrección necesaria] Diferencia entre repositorio y despliegue.** `estado-actual.md` §«Cambios sin confirmar» describe correctamente la reescritura local de `/perfil`, pero no explicita que el despliegue revisado todavía sirve la versión anterior del texto. La comparación independiente del HTML de `https://juanfuc.vercel.app/perfil` confirmó esa divergencia. Dado que el documento dice describir repositorio y despliegue, debe registrarla expresamente.
  3. **[Corrección necesaria] Caché y aleatoriedad de la portada.** `estado-actual.md` §«Fuentes de datos» generaliza que el contenido dinámico se cachea en `localStorage`, pero la grilla de `src/routes/+layout.svelte` no guarda sus datos allí. Además, `estado-actual.md` §«Comportamiento de la grilla» y `diagnostico-inicial.md` §7 dicen que el orden cambia «entre navegaciones/recargas»: el layout raíz persiste durante la navegación interna de SvelteKit y no vuelve a ejecutar `onMount`, por lo que el rebarajado ocurre al cargar o recargar la aplicación, no en cada navegación interna. Sí cambia tras una recarga dentro de la misma sesión, que es el incumplimiento relevante.
  4. **[Corrección necesaria] Decisiones implícitas fuera del registro de confirmadas.** `decisiones.md` §«Tipografía» declara acordado que la tipografía intencional debe cargarse, pero ese acuerdo no figura entre las nueve decisiones confirmadas ni está formulado en `brief.md`. Asimismo, §«Otros temas abiertos» llama «acordado como necesario» al tratamiento de metadatos y `lang="es"`, mientras `diagnostico-inicial.md` presenta sus prioridades como referencia no vinculante. En el corpus documental disponible ambas afirmaciones carecen de sustento como decisiones confirmadas: deben promoverse explícitamente con su procedencia o redactarse como propuestas pendientes.
  5. **[Corrección necesaria] Verificabilidad de R01.** `rondas/R01-base-estructural.md` criterio 4 no exige probar una recarga de la misma pestaña, por lo que su verificación puede pasar aunque el orden no sea estable durante toda la sesión; también falta definir el límite de «sesión» (recarga, pestaña nueva y cierre de pestaña). El criterio 5 («apariencia visual actual», proporciones y ausencia de pérdida) no fija rutas, viewports, capturas o conteos de referencia y depende de datos externos mutables. El criterio 6 no enumera si «cada ruta interior» incluye las tres subsecciones de publicaciones. Finalmente, exige `svelte-check`, pero el paquete no está instalado y `package.json` no ofrece un script `check`; `svelte-kit sync` no lo sustituye. Deben indicarse comandos reproducibles, cobertura de rutas y una línea base objetiva.
  6. **[Corrección necesaria] Suficiencia del brief para un agente nuevo.** `README.md` anuncia que `brief.md` explica «para quién» es el sitio, pero el brief no identifica audiencias prioritarias. Además, `brief.md` §«Acuerdos de dirección» menciona *Caja Negra*, «En curso» y los hilos de investigación sin definirlos ni enlazar una referencia o indicar dónde existe su contenido. Un agente sin la conversación original no puede verificar qué referente visual debe consultar ni qué significan esas capas secundarias.
  7. **[Corrección menor] Afirmaciones técnicas no sustentadas o imprecisas.** `estado-actual.md` §«Stack» presenta «Node 22» como propiedad del proyecto, pero no hay campo `engines`, `.nvmrc` ni otro archivo que lo fije; `.npmrc` solo activa `engine-strict`. `diagnostico-inicial.md` §9 y `estado-actual.md` §«Problemas vigentes» dicen que se importa «d3 completo», mientras el código importa `d3-fetch`; lo comprobable es que `package.json` declara el paquete agregador `d3` para disponer de ese subpaquete. No puede afirmarse que todo D3 se entregue al cliente sin analizar el bundle.
  8. **[Conforme] Referencias y enlaces.** `AGENTS.md` y `CLAUDE.md` remiten al README correcto y nombran la ronda activa correcta. Todos los enlaces Markdown internos existentes resuelven a archivos presentes; también se verificaron los siete documentos de R00 y los 46 archivos previamente versionados bajo `docs/`.
  9. **[Conforme con la limitación ya declarada] Diagnóstico visual.** La paleta, tipografías declaradas, grillas y estados hover descritos corresponden al CSS. La verificación independiente del HTML público reprodujo el `lang="en"`, la ausencia de título/description/OG/fuentes y los 2816 bytes de `/`. No fue posible realizar una nueva inspección renderizada con capturas; por tanto, los juicios de percepción visual continúan correctamente limitados a `[INFERIDO]`.
- **Veredicto:** **requiere correcciones**.

## Respuesta a la revisión

> Responsable documental de R00. Fecha: 2026-07-24. **No se modificó el texto del revisor**; se resuelve cada hallazgo numerado. Convención de evidencia heredada de `diagnostico-inicial.md`: **[REPO]** (código), **[LIVE]** (HTML servido por `curl`), **[INFERIDO]** (no verificado con navegador renderizado).

### H1 — Alcance del prerender y del contenido en cliente
- **Corrección realizada:** se acotó la afirmación al **contenido dinámico de Sheets** y se distinguió del **contenido estático prerenderizado**. `/perfil` es estático y su texto **sí** está en el HTML publicado (aunque oculto por CSS); las rutas interiores prerenderizan su andamiaje (nav, títulos, enlaces); la **portada** es el caso realmente vacío.
- **Archivo y sección:** `diagnostico-inicial.md` §4 (reescrita) y §1 (bullet de datos); `estado-actual.md` §Rutas (`/perfil`), §Despliegue, §Problemas vigentes (SEO).
- **Evidencia:** **[LIVE]** `curl https://juanfuc.vercel.app/perfil` → 3979 bytes con el párrafo dentro del `<p>`; `curl …/` → 2816 bytes con grilla vacía. **[REPO]** `perfil/+page.svelte` no hace fetch.
- **Pendiente:** la estrategia para indexar el contenido dinámico sigue pendiente en `decisiones.md` (no es objeto de R00).

### H2 — Diferencia entre repositorio y despliegue
- **Corrección realizada:** se registró expresamente que el **despliegue sirve el texto anterior** de `/perfil`, mientras el **árbol local** tiene la reescritura **sin confirmar**.
- **Archivo y sección:** `estado-actual.md` §«Cambios sin confirmar…» (nuevo párrafo «Divergencia repositorio ↔ despliegue»).
- **Evidencia:** **[LIVE]** el HTML de `/perfil` contiene «Hola, soy Juan Felipe… Profesor universitario…» y **no** «mediaciones visuales»/«tesis laureada»; **[REPO]** `git status` marca ` M` en `perfil/+page.svelte` y `Perfil.svelte`.
- **Pendiente:** decidir si/ cuándo se confirma y despliega ese contenido (fuera de R00).

### H3 — Caché y aleatoriedad de la portada
- **Corrección realizada:** (a) se aclaró que **la grilla NO se cachea en `localStorage`** (el layout solo guarda `lastPage`); (b) se corrigió el momento del rebarajado: ocurre **al cargar/recargar la app, no en cada navegación interna** de SvelteKit; el incumplimiento real es que **una recarga en la misma sesión** cambia el orden.
- **Archivo y sección:** `estado-actual.md` §«Fuentes de datos» (nueva tabla de `localStorage`) y §«Comportamiento de la grilla»; `diagnostico-inicial.md` §1 y §7.
- **Evidencia:** **[REPO]** grep de `localStorage.*Item` (layout: solo `lastPage`; datos en experiencia/publicaciones/artículos/libros/capítulos/programación); `shuffleArray` en `onMount` del layout raíz, que persiste en navegación interna.
- **Pendiente:** el mecanismo de estabilidad por sesión (semilla/`sessionStorage`) sigue pendiente (R01/decisiones).

### H4 — Decisiones implícitas fuera del registro de confirmadas
- **Corrección realizada:** se **degradaron a propuestas pendientes** dos afirmaciones que se presentaban como acuerdos sin procedencia: (a) «la tipografía debe cargarse de verdad»; (b) «metadatos y `lang="es"` acordados como necesarios». Ahora se distinguen el **hecho verificado** (fuentes no cargadas; `lang` en inglés) de la **respuesta no validada**.
- **Archivo y sección:** `decisiones.md` §Tipografía y §«Otros temas abiertos».
- **Evidencia:** **[REPO]** las nueve decisiones confirmadas y `brief.md` no contienen esos acuerdos; `diagnostico-inicial.md` marca sus prioridades como «referencia no vinculante».
- **Pendiente:** ambas requieren validación explícita del responsable para promoverse a confirmadas.

### H5 — Verificabilidad de R01
- **Corrección realizada:** (a) se **definió «sesión»** (recarga de la misma pestaña vs. pestaña nueva/cierre) y se exige **probar recarga F5**; (b) se añadió una **línea base visual obligatoria** (rutas, viewports 1280×800 y 375×667, conteos de referencia) por depender de datos externos mutables; (c) se enumeraron las **tres subsecciones de publicaciones** en la cobertura de rutas; (d) se **eliminó la exigencia de `svelte-check`** (no instalado; sin script `check`) y se fijaron **comandos reales** (`npm run build`; `npx svelte-kit sync` opcional y no bloqueante; `npm run preview` para comprobación manual).
- **Archivo y sección:** `rondas/R01-base-estructural.md` §«Definición de sesión», §«Línea base visual», §«Criterios de aceptación» (4 y 5), §«Verificaciones», §«Contexto» y §«Registro de ejecución».
- **Evidencia:** **[REPO]** `package.json` `scripts` = `dev`/`build`/`preview`; no hay `svelte-check` en dependencias.
- **Pendiente:** la ejecución real de la línea base y las verificaciones corresponde a R01 (aún no activa).

### H6 — Suficiencia del brief para un agente nuevo
- **Corrección realizada:** (a) se **definieron** «En curso», «hilos de investigación» y el papel de **Caja Negra**, con enlace **https://cajanegraeditora.com.ar/**; (b) al **no existir prioridad de audiencias confirmada**, se registró como **pregunta pendiente** (no se inventó) y se ajustó el README para no dar por sentado el «para quién».
- **Archivo y sección:** `brief.md` §«Qué es el sitio» (audiencia pendiente) y nuevo §«Glosario»; `decisiones.md` §«Preguntas pendientes»; `README.md` (punto 1 del orden de lectura).
- **Evidencia:** **[REPO]** el `brief.md` original no listaba audiencias ni definía esos términos.
- **Pendiente:** el responsable debe fijar la prioridad de audiencias y el alcance concreto de «En curso», «hilos» y de lo que se adopta de Caja Negra.

### H7 — Afirmaciones técnicas no sustentadas o imprecisas
- **Corrección realizada:** (a) **Node 22** pasa a describirse como **entorno observado**, no versión fijada por el proyecto (no hay `engines`/`.nvmrc`/`.node-version`); (b) se distingue el **paquete agregador `d3` declarado** de la **importación efectiva `d3-fetch`**, y se retira toda afirmación sobre el bundle (marcada **[INFERIDO]**, sin medición).
- **Archivo y sección:** `estado-actual.md` §Stack y §Problemas vigentes; `diagnostico-inicial.md` §9.
- **Evidencia:** **[REPO]** `grep engines` sin resultado; no existen `.nvmrc`/`.node-version`; `import { csv } from 'd3-fetch'`; `d3` en `dependencies`.
- **Pendiente:** medir el bundle si se quiere afirmar el peso real de D3 (fuera de R00).

### H8 — Referencias y enlaces (Conforme)
- **Acción:** sin corrección de contenido. Tras las ediciones se **revalidaron todos los enlaces internos** (ver «Revalidación de enlaces» abajo).
- **Pendiente:** ninguno.

### H9 — Diagnóstico visual (Conforme con la limitación)
- **Acción:** se **mantiene** la limitación: los juicios de percepción visual siguen marcados **[INFERIDO]**; no hubo inspección renderizada con capturas en R00. La reverificación de R00 y la línea base de R01 podrán aportar capturas.
- **Pendiente:** verificación visual con navegador (R01 / reverificación).

### Revalidación de enlaces internos (tras las correcciones)
- `README.md` → `brief.md`, `estado-actual.md`, `decisiones.md`, `diagnostico-inicial.md`, `rondas/R00-…`, `rondas/R01-…`: **OK** (archivos presentes).
- Referencias cruzadas nuevas: `estado-actual.md` ↔ `diagnostico-inicial.md` (§4); `decisiones.md` ← `brief.md` y `README.md`; enlace externo `https://cajanegraeditora.com.ar/` en `brief.md`: **OK** (comprobados los destinos internos por ruta).
- `AGENTS.md` y `CLAUDE.md` → `docs/renovacion-perfil/README.md` y ronda: **OK**.

### Estado resultante
- R00: **correcciones aplicadas — pendiente de reverificación**. No cerrada.
- R01: **no activa todavía**.

## Reverificación final

- **Revisor:** Codex (reverificación independiente)
- **Fecha:** 2026-07-24
- **Resolución de H6:** el responsable del proyecto confirmó que Caja Negra es el referente concreto únicamente para tipografía, líneas, composición plana y estados de interacción; que «En curso» será una capa secundaria para proyectos e investigaciones en desarrollo; y que esos proyectos pueden agrupar, conectar y superponer publicaciones, clases, convocatorias, residencias, materiales u otros trabajos, además de relacionarse con elementos de la grilla. Las decisiones quedaron registradas en `brief.md` y `decisiones.md`.
- **Veredicto final:** **aprobada**.

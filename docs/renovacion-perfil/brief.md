# Brief — dirección del proyecto

Documento breve y estable. Registra **qué es el sitio y los acuerdos de dirección ya cerrados**. Si un acuerdo cambia, se actualiza aquí y se refleja en `decisiones.md`.

## Qué es el sitio

Sitio personal de **Juan Felipe Urueña Calderón** — doctor en Historia, magíster en Filosofía. Construido en SvelteKit y desplegado en Vercel (https://juanfuc.vercel.app/).

**Audiencia prioritaria: pendiente.** No hay una prioridad de audiencias confirmada en la documentación; **no se infiere aquí**. Queda registrada como pregunta abierta en `decisiones.md` → «Preguntas pendientes → Prioridad de audiencias». (Corrige el hallazgo H6: el `README.md` no debe dar por sentado el «para quién».)

## Intención

Renovar el sitio conservando su identidad visual y su intención original, llevándolo a los estándares actuales de una web de perfil académico y profesional.

## Acuerdos de dirección

Estos puntos están **acordados** y orientan cualquier decisión posterior:

1. **Perfil académico primero.** El sitio debe reconocerse prioritariamente como un **perfil académico personal**. Todo lo demás se subordina a esa lectura.

2. **La grilla es un índice visual de descubrimiento.** La grilla de imágenes de la portada funciona como una entrada exploratoria al trabajo, no como una galería decorativa ni como un portafolio cerrado.

3. **Igualdad conceptual de los elementos.** Los elementos de la grilla tienen el mismo peso conceptual entre sí, **aunque las imágenes conserven proporciones distintas**. Las diferencias de proporción son propias de cada imagen y se respetan; no implican jerarquía.

4. **Aleatoriedad estable por sesión.** La combinación/orden de la grilla es **aleatoria**, pero debe **permanecer estable durante una sesión** (no rebarajar en cada navegación o re-render dentro de la misma visita).

5. **La grilla pertenece a la portada.** La grilla es de la portada (`/`) y **no debe anteceder a las páginas interiores**. Las rutas interiores empiezan por su propio contenido, no por la grilla.

6. **Se conserva la identidad cromática y el gesto.** Se mantienen la **paleta actual** (azul petróleo + ámbar sobre fondo azul-gris claro) y el **efecto de gris a color** en las imágenes.

7. **Renovación pragmática.** La renovación debe ser **pragmática y aprovechar los contenidos y la infraestructura existentes** (Google Sheets como fuente, imágenes ya publicadas, rutas actuales) en lugar de rehacer desde cero.

8. **«En curso» como capa secundaria de proyectos.** Hará explícitos proyectos e investigaciones en desarrollo. Cada proyecto podrá agrupar y conectar publicaciones, clases, convocatorias, residencias, materiales u otros trabajos; los proyectos podrán superponerse y relacionarse con elementos de la grilla. Esta capa no lidera la renovación inicial.

9. **Caja Negra como referente concreto y parcial.** Se toma como referencia únicamente para **tipografía, líneas, composición plana y estados de interacción**; **no** como plantilla general del sitio.

## Glosario (definiciones para un agente sin la conversación original)

**[Confirmado por el responsable del proyecto al cerrar R00]** Estos términos deben entenderse así:

- **«En curso».** Capa secundaria para hacer explícitos proyectos e investigaciones en desarrollo.
- **Proyectos e hilos de investigación.** Pueden agrupar y conectar publicaciones, clases, convocatorias, residencias, materiales u otros trabajos; pueden superponerse y relacionarse con elementos de la grilla. Su implementación concreta sigue pendiente.
- **Caja Negra.** Referente visual concreto para tipografía, líneas, composición plana y estados de interacción, no una plantilla general. Enlace: **https://cajanegraeditora.com.ar/**.

## Fuera del alcance inicial

- Rehacer la fuente de datos (migrar Sheets a Markdown/CMS): posible a futuro, no en las primeras rondas.
- Convertir el sitio en portafolio o en revista.
- Que «En curso» o los hilos de investigación sean la portada.

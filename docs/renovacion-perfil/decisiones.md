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

---

## Propuestas pendientes de validación

Ideas y direcciones **no cerradas**. No se implementan hasta ser validadas; cuando lo sean, pasan a «confirmadas».

### Tipografía
- **Combinación tipográfica exacta.** **[Corregido R00-respuesta, H4]** En el corpus documental disponible **no consta como decisión confirmada** (no está entre las nueve decisiones ni en `brief.md`) el que la tipografía intencional «deba cargarse de verdad». Se registra, por tanto, como **propuesta pendiente**: el diagnóstico observa que Poppins/Montserrat están declaradas pero no se cargan (hecho verificado), pero **la respuesta —cargarlas, y con qué pareja definitiva, pesos, escala y fallback— no ha sido validada**. Cualquier promoción a «confirmada» debe hacerse con su procedencia.

### Grilla
- **Franja contextual de la grilla.** Pendiente decidir si la portada incorpora una **franja/banda contextual** (texto breve, encabezado o marco que sitúe la grilla como índice de descubrimiento) y, en tal caso, su contenido, posición y relación con la grilla.
- **Mecanismo exacto de estabilidad por sesión.** Está acordado *que* debe ser estable por sesión; el **cómo** (semilla en `sessionStorage`, orden fijado en el primer render, etc.) está pendiente.

### Sistema visual
- **Sistema preciso de líneas.** Uso de líneas/reglas como recurso estructural (inspiración parcial en *Caja Negra*): grosor, color, dónde aplican (separadores, marcos, encabezados). Pendiente de definición.
- **Aplicación precisa de la composición plana.** Caja Negra está confirmada como referencia para este aspecto, pero su traducción concreta al sitio sigue pendiente.
- **Estados de interacción.** Definición fina de hover/focus/active para enlaces, tarjetas e imágenes (parcialmente inspirados en *Caja Negra*). Pendiente.

### Interacción de la grilla
- **Posible eliminación del aumento de escala en hover.** Se evalúa **quitar el `scale()` en hover** de las imágenes (conservando el paso de gris a color). Pendiente de validación; el efecto gris→color sí se conserva (confirmado).

### Otros temas abiertos (registro, aún sin acordar)
- Estrategia para hacer **indexable** el contenido dinámico de Sheets (mover carga de datos a build/servidor vs. seguir en cliente): pendiente; con impacto en SEO.
- **Metadatos y `lang="es"`.** **[Corregido R00-respuesta, H4]** El diagnóstico observa (hecho verificado) que faltan `<title>`/description/OG y que `<html lang="en">` en un sitio en español. Pero **corregirlo no es una decisión confirmada**: las prioridades del diagnóstico son referencia **no vinculante**. Se registra como **propuesta pendiente** de validar y asignar a una ronda, no como acuerdo.
- **Destino de `docs/` y del código muerto:** pendiente.

### Preguntas pendientes (requieren respuesta del responsable del proyecto)
- **[Corregido R00-respuesta, H6] Prioridad de audiencias.** El `README.md` anunciaba que `brief.md` explica «para quién» es el sitio, pero **no existe una prioridad de audiencias confirmada** en el corpus. **No se inventa.** Pregunta abierta: ¿cuál es el público prioritario (pares académicos, comités de evaluación/becas, estudiantes, medios/divulgación, colaboradores de proyectos)? La respuesta condiciona jerarquía de contenidos y metadatos.
- **[Corregido R00-respuesta] Colisión de `docs/`.** La nueva documentación (`docs/renovacion-perfil/`) **convive** en el mismo directorio que un **build antiguo de GitHub Pages** versionado bajo `docs/` (46 archivos: `docs/index.html`, `docs/_app/…`, etc.). Riesgo de confusión y de que un futuro proceso de publicación mezcle ambos. Se registra como **pendiente**; **no se mueve la carpeta sin una decisión confirmada**.

---

## Cómo se registran los cambios

- Al validar una propuesta, muévela a **Confirmadas** anotando la ronda (p. ej. «(R01)») y, si aplica, un enlace al documento de esa ronda.
- Si una decisión confirmada se revierte, muévela de nuevo a **Pendientes** explicando por qué.

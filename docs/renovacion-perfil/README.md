# Renovación del perfil académico — documentación del proyecto

Esta carpeta externaliza el contexto de la renovación del sitio para que **cualquier agente o persona pueda continuar el trabajo sin depender de la sesión donde se originó**. Es la fuente de verdad del proyecto.

## Orden de lectura para un agente nuevo

Léelos en este orden antes de proponer o escribir nada:

1. **[`brief.md`](./brief.md)** — qué es el sitio, los acuerdos de dirección ya cerrados y un glosario de términos («En curso», «hilos de investigación», Caja Negra). La **prioridad de audiencias no está confirmada** y figura como pregunta pendiente en `decisiones.md`.
2. **[`estado-actual.md`](./estado-actual.md)** — cómo está implementado el sitio hoy (stack, rutas, datos, despliegue, problemas vigentes).
3. **[`decisiones.md`](./decisiones.md)** — decisiones confirmadas vs. propuestas pendientes de validación.
4. **Documento de la ronda más reciente** — **[`rondas/R01-base-estructural.md`](./rondas/R01-base-estructural.md)**, cerrada y aprobada. **No hay una ronda de implementación activa; la siguiente está pendiente de definición.**

## Documentos de contexto (referencia)

- **[`diagnostico-inicial.md`](./diagnostico-inicial.md)** — auditoría técnica y visual de partida, contrastada con el código y con el sitio publicado. Distingue lo *verificado en el repositorio* de lo *observado en https://juanfuc.vercel.app/*.

## Rondas

Cada ronda es una unidad acotada de trabajo con objetivo y criterios de aceptación propios.

- **[`rondas/R00-documentacion-inicial.md`](./rondas/R00-documentacion-inicial.md)** — creación de esta documentación (no toca código del sitio). **Cerrada y aprobada.**
- **[`rondas/R01-base-estructural.md`](./rondas/R01-base-estructural.md)** — primera ronda de implementación (base estructural). **Cerrada y aprobada.**

## Reglas de trabajo

- Antes de editar, ejecuta `git status` y **preserva los cambios existentes** (a la fecha de R00 hay ediciones sin confirmar en el texto de `/perfil`; ver `estado-actual.md`).
- No se modifica el código del sitio hasta que la ronda activa lo autorice explícitamente en sus criterios de aceptación.
- Cuando cierres una ronda, actualiza `decisiones.md` (mueve lo validado de «pendiente» a «confirmada») y abre el documento de la siguiente ronda.
- Distingue siempre **lo verificado en el repositorio** de **lo observado en el sitio publicado**; no mezcles ambas cosas como si tuvieran el mismo grado de certeza.

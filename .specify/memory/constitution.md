# Constitución de todo-sdd

## Principios Fundamentales

### I. Simplicidad

La aplicación DEBE ser la solución más simple que cumpla los requisitos vigentes.

- Cada dependencia externa (biblioteca, framework, servicio) DEBE justificarse por escrito en el
  plan de la funcionalidad, explicando por qué no basta con las capacidades nativas de la
  plataforma.
- NO se DEBE añadir código, configuración ni abstracciones para necesidades futuras hipotéticas
  (YAGNI).
- Ante dos soluciones equivalentes, se DEBE elegir la que tenga menos piezas móviles.

**Justificación**: una app de tareas tiene un dominio pequeño; cada dependencia añade
mantenimiento, peso y superficie de fallo sin aportar valor proporcional.

### II. Usabilidad

Cualquier usuario DEBE poder crear una tarea en menos de 10 segundos desde que abre la aplicación.

- El campo para crear una tarea DEBE estar visible y accesible sin navegación previa.
- Crear una tarea NO DEBE requerir más que escribir el texto y confirmar (p. ej., tecla Enter
  o un único botón).
- Ningún campo distinto del texto de la tarea DEBE ser obligatorio al crearla.

**Justificación**: la captura rápida es el propósito central de una lista de tareas; la fricción
en ese paso hace que el usuario abandone la herramienta.

### III. Calidad

Cada funcionalidad DEBE tener al menos una prueba automatizada que verifique sus criterios de
aceptación.

- Cada criterio de aceptación de una especificación DEBE estar cubierto por al menos una prueba
  identificable.
- Una funcionalidad NO se considera terminada hasta que sus pruebas existan y pasen.
- Las pruebas DEBEN ejecutarse con un único comando documentado.

**Justificación**: las pruebas son la evidencia objetiva de que lo especificado es lo que se
construyó, y protegen contra regresiones.

### IV. Persistencia

Las tareas NO DEBEN perderse al cerrar o recargar la aplicación.

- Toda creación, edición, completado o eliminación de una tarea DEBE guardarse de forma
  persistente en el momento en que ocurre, sin requerir una acción explícita de "guardar".
- Al reabrir o recargar la aplicación, el usuario DEBE ver exactamente el estado de tareas que
  dejó.
- Este comportamiento DEBE estar cubierto por al menos una prueba (ver Principio III).

**Justificación**: una lista de tareas que pierde datos destruye la confianza del usuario y
anula su utilidad.

### V. Idioma

Toda la interfaz de usuario y toda la documentación del proyecto DEBEN estar en español.

- Textos visibles, mensajes de error, etiquetas, marcadores de posición y ayudas DEBEN
  redactarse en español.
- Especificaciones, planes, tareas, README y demás documentación DEBEN redactarse en español.
- Los identificadores de código pueden seguir las convenciones del lenguaje, pero los
  comentarios dirigidos a personas DEBERÍAN estar en español.

**Justificación**: el público objetivo y el equipo trabajan en español; la coherencia
lingüística evita confusiones y mejora la accesibilidad.

## Restricciones Adicionales

- El stack tecnológico DEBE elegirse priorizando el Principio I: se prefieren capacidades nativas
  de la plataforma frente a dependencias de terceros.
- El mecanismo de persistencia DEBE funcionar sin configuración adicional por parte del usuario.
- El tiempo para crear una tarea (Principio II) DEBE poder verificarse mediante una prueba o un
  recorrido documentado.

## Flujo de Desarrollo y Puertas de Calidad

- Cada funcionalidad sigue el flujo Spec Kit: especificación → plan → tareas → implementación.
- El plan de cada funcionalidad DEBE incluir una verificación explícita de cumplimiento contra
  los cinco principios de esta constitución.
- Antes de dar una funcionalidad por terminada se DEBE comprobar que:
  1. Todas sus pruebas de criterios de aceptación pasan (Principio III).
  2. Las tareas sobreviven a cerrar y recargar la aplicación (Principio IV).
  3. Toda dependencia nueva está justificada (Principio I).
  4. Todo texto nuevo de interfaz y documentación está en español (Principio V).

## Gobernanza

- Esta constitución prevalece sobre cualquier otra práctica o preferencia del proyecto.
- Las enmiendas DEBEN documentarse en este archivo, indicando el cambio, su motivo y el impacto
  en especificaciones, planes o código existentes.
- Política de versionado (semántico):
  - MAYOR: eliminación o redefinición incompatible de un principio o regla de gobernanza.
  - MENOR: nuevo principio o sección, o ampliación material de la guía existente.
  - PARCHE: aclaraciones, redacción o correcciones sin cambio semántico.
- Toda revisión de especificaciones, planes e implementación DEBE verificar el cumplimiento de
  estos principios; cualquier excepción DEBE justificarse por escrito en el plan correspondiente.

**Versión**: 1.0.0 | **Ratificada**: 2026-11-05 | **Última enmienda**: 2026-11-05

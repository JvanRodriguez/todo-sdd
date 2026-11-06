# Plan de Implementación: Gestión de tareas para estudiantes

**Rama**: `001-gestion-tareas` | **Fecha**: 2026-11-06 | **Especificación**: [spec.md](./spec.md)

**Entrada**: Especificación de funcionalidad desde `specs/001-gestion-tareas/spec.md`

## Resumen

Aplicación web personal de una sola pantalla para registrar y organizar entregas universitarias.
Se implementará con HTML, CSS y JavaScript nativo (módulos ES), sin framework, sin proceso de
compilación y sin dependencias en tiempo de ejecución. Las tareas se guardan en el
`localStorage` del dispositivo mediante escritura inmediata tras cada creación, edición, cambio
de estado o eliminación. La única dependencia será de desarrollo: Playwright Test, para verificar
los criterios de aceptación en un navegador real, incluida la persistencia tras recargar.

El enfoque prioriza la captura rápida: el campo de título está visible al abrir, el título es el
único campo obligatorio y se confirma con Enter o con un único botón. El resto de campos
(descripción, fecha límite y prioridad) son opcionales y tienen valores por defecto.

## Contexto Técnico

**Lenguaje/Versión**: JavaScript ES2022, HTML5 y CSS3. Módulos ES nativos del navegador.

**Dependencias principales**: Ninguna en tiempo de ejecución. Solo desarrollo:
`@playwright/test` (automatización de navegador para las pruebas de aceptación).

**Almacenamiento**: `localStorage` del navegador, con una única clave versionada
`todo-sdd:tareas:v1` que contiene un sobre JSON `{ version, tareas }`. No hay backend ni base de
datos.

**Pruebas**: Playwright Test sobre Chromium, con un único comando documentado: `npm test`.

**Plataforma objetivo**: Navegadores de escritorio modernos (Chrome, Edge, Firefox y Safari
actuales), en un único dispositivo personal. La aplicación se sirve como archivos estáticos
durante desarrollo y pruebas; una vez cargada no necesita red ni servicios externos.

**Tipo de proyecto**: Aplicación web estática de una sola página (single project). No hay
frontend/backend separados ni aplicación móvil nativa.

**Objetivos de rendimiento**: Con al menos 200 tareas, filtrar, ordenar y volver a renderizar la
lista en menos de 1 segundo percibido (SC-004). Crear una tarea con solo título en menos de 10
segundos desde que se abre la aplicación (SC-001).

**Restricciones**: Sin cuentas ni inicio de sesión; datos solo en el dispositivo; interfaz y
documentación en español; sin proceso de build ni dependencias de runtime; fechas límite sin hora;
la aplicación debe seguir funcionando sin conexión después de cargada.

**Escala/Alcance**: Un usuario, un dispositivo, una pantalla, 200 tareas o más. Sin etiquetas,
subtareas, recordatorios, búsqueda, importación/exportación, adjuntos ni sincronización.

## Constitution Check

*PUERTA: debe pasar antes de la investigación de Fase 0. Se vuelve a evaluar tras el diseño de
Fase 1.*

| Principio | Puerta verificable | Resultado | Evidencia en el plan |
|-----------|--------------------|-----------|----------------------|
| I. Simplicidad | ¿La solución usa las capacidades nativas y justifica cada dependencia? | **PASA** | HTML/CSS/JS nativos y `localStorage`; sin framework, build ni runtime externo. Playwright es solo de desarrollo y se justifica abajo. |
| II. Usabilidad | ¿El campo de creación está visible y crear solo exige título + confirmar? | **PASA** | Formulario de creación siempre visible en la pantalla principal; título obligatorio; Enter o botón único; prioridad media por defecto. |
| III. Calidad | ¿Cada criterio de aceptación tiene al menos una prueba automatizada y un comando único? | **PASA** | Suites de Playwright mapeadas a las historias y requisitos; `npm test` ejecuta todo. |
| IV. Persistencia | ¿Cada cambio se guarda al ocurrir y sobrevive a recargar? | **PASA** | Escritura inmediata en `localStorage` tras cada mutación; pruebas de recarga para creación, edición, completado y eliminación. |
| V. Idioma | ¿Interfaz y documentación están en español? | **PASA** | Textos de UI, mensajes, especificación, plan, contratos y pruebas documentados en español. |

**Resultado de la puerta inicial**: PASA. No hay violaciones que justificar.

**Reevaluación tras el diseño de Fase 1**: PASA. Los artefactos `research.md`, `data-model.md`,
`contracts/` y `quickstart.md` mantienen las cinco puertas sin excepciones ni complejidad
adicional.

## Estructura del Proyecto

### Documentación (esta funcionalidad)

```text
specs/001-gestion-tareas/
├── plan.md              # Este archivo
├── research.md          # Fase 0: decisiones técnicas
├── data-model.md        # Fase 1: entidades, reglas y orden
├── quickstart.md        # Fase 1: guía de validación ejecutable
├── contracts/
│   ├── persistencia.schema.json  # Contrato del sobre guardado en localStorage
│   └── interfaz-ui.md            # Contrato de elementos, textos y estados de la UI
├── checklists/
│   └── requirements.md
└── tasks.md             # Fase 2 (/speckit-tasks; no lo crea este comando)
```

### Código fuente (raíz del repositorio)

```text
index.html                 # Estructura de la pantalla única
styles.css                 # Presentación, estados completada/vencida y accesibilidad visual
src/
├── app.js                 # Arranque, eventos de UI, render y coordinación
├── modelo.js              # Reglas puras: validación, filtros, orden y vencimiento
└── almacen.js             # Lectura/escritura de localStorage y traducción de errores
scripts/
└── servidor.mjs           # Servidor estático mínimo con módulos nativos de Node
tests/
├── e2e/
│   ├── crear-y-persistir.spec.js
│   ├── gestionar-tareas.spec.js
│   ├── vencimiento.spec.js
│   ├── filtrar-y-ordenar.spec.js
│   └── interfaz-y-errores.spec.js
└── apoyo/
    └── datos.js           # Fábricas de tareas y utilidades de fechas para pruebas
playwright.config.js
package.json
```

**Decisión de estructura**: Proyecto único y plano. Los archivos estáticos se sirven tal cual
desde la raíz; no hay carpeta de compilación ni empaquetado. `modelo.js` concentra la lógica pura
y comprobable, `almacen.js` aísla la persistencia y `app.js` se limita a la interfaz. Esta
separación evita dependencias y mantiene una única pantalla, en línea con el Principio I.

## Diseño Técnico

### Arquitectura y responsabilidades

- **`index.html`**: una sola página con encabezado, formulario de creación, controles de filtro y
  orden, región de mensajes (`aria-live`) y lista de tareas.
- **`modelo.js`**: funciones puras para crear tareas con valores por defecto, validar campos,
  determinar si una tarea está vencida, filtrar y ordenar. No toca el DOM ni el almacenamiento.
- **`almacen.js`**: carga y guarda el sobre JSON versionado; valida la forma de los datos al leer;
  convierte fallos de `localStorage` en un resultado controlado para la UI.
- **`app.js`**: mantiene el estado en memoria, escucha los eventos del formulario y de la lista,
  aplica mutaciones transaccionales, renderiza y gestiona el foco y los mensajes.
- **`styles.css`**: estados visuales de completada y vencida; el estado nunca se comunica solo con
  color.

### Flujo de persistencia transaccional

1. Al arrancar se lee una vez `localStorage["todo-sdd:tareas:v1"]`. Si no existe, se parte de una
   lista vacía. Si existe pero no cumple el contrato, se muestra un mensaje en español y se parte
   de una lista vacía sin sobrescribir hasta la siguiente acción del usuario.
2. Ante cualquier mutación, se trabaja sobre una copia del estado, se valida y se serializa el
   sobre completo.
3. Se intenta `localStorage.setItem`. Solo si la escritura termina sin error se confirma el nuevo
   estado en memoria y se vuelve a renderizar.
4. Si la escritura falla (por ejemplo, cuota llena o almacenamiento bloqueado), el estado en
   memoria no se sustituye, la interfaz no muestra el cambio como guardado y se informa en
   español: «No se pudo guardar el cambio. Inténtalo de nuevo.».
5. No hay botón de «guardar lista»: cada acción se persiste en el momento en que ocurre.

No se implementa sincronización entre pestañas ni escucha del evento `storage`; la especificación
lo deja fuera de alcance.

### Validación y normalización

- **Título**: se recortan espacios al inicio y al final; debe quedar entre 1 y 200 caracteres. Un
  título vacío o solo con espacios se rechaza con «El título es obligatorio.».
- **Descripción**: opcional, hasta 2000 caracteres; vacía se guarda como cadena vacía.
- **Fecha límite**: opcional, día del calendario en formato `AAAA-MM-DD`; debe ser una fecha real.
  Se permite una fecha pasada al crear o editar.
- **Prioridad**: `alta`, `media` o `baja`; si no se elige, `media`.
- **Estado**: toda tarea nueva nace pendiente; al editar no se permite cambiar el estado desde el
  formulario de edición (se usa la casilla de completada).
- Las restricciones HTML (`required`, `maxlength`, `type="date"`) son una ayuda de entrada, pero
  `modelo.js` vuelve a validar antes de guardar. La edición aplica exactamente las mismas reglas.
- Los mensajes de validación se muestran en español en una región `aria-live`, sin crear ni
  modificar la tarea.

### Fechas y condición de vencida

- La fecha límite se guarda como cadena `AAAA-MM-DD`, sin hora ni conversión de zona horaria.
- El «día actual» se obtiene del reloj local del dispositivo y se compara como cadena
  `AAAA-MM-DD`; una tarea con la fecha de hoy no está vencida.
- `vencida` es un valor derivado: `estado === "pendiente" && fechaLimite !== null &&
  fechaLimite < hoy`. Nunca se guarda en `localStorage`.
- La condición se recalcula en cada render. Al recargar la aplicación se vuelve a evaluar con la
  fecha actual, que es el mínimo exigido por la especificación.
- Las fechas se muestran en formato español `dd/mm/aaaa` a partir de sus componentes, sin
  conversiones que puedan desplazar el día.

### Filtros, orden y desempates

- El estado de filtro y orden vive solo en memoria; al abrir se usan «Todas» y «Fecha límite».
- Filtros: «Todas», «Pendientes» y «Completadas».
- Orden por fecha límite: fecha más próxima primero; las tareas sin fecha van al final. Desempate:
  prioridad alta, media, baja; después, creación más reciente primero.
- Orden por prioridad: alta, media, baja. Desempate: fecha límite más próxima primero, sin fecha al
  final; después, creación más reciente primero.
- Si se crea una tarea mientras el filtro «Completadas» está activo, la tarea se crea igualmente y
  el filtro cambia automáticamente a «Todas» para que la nueva tarea sea visible.
- El orden se aplica siempre después del filtro y antes de renderizar.

### Interfaz y accesibilidad

- Formulario de creación siempre visible en la pantalla principal. La fila principal contiene el
  título y el botón «Añadir tarea»; los campos opcionales (descripción, fecha y prioridad) están
  visibles debajo, sin navegación adicional.
- Confirmar con Enter desde el campo de título o con el botón crea la tarea.
- Cada tarea muestra casilla de completada, título, descripción si existe, fecha si existe,
  prioridad y sus acciones «Editar» y «Eliminar».
- Las tareas completadas muestran la etiqueta textual «Completada» además del estilo visual. Las
  vencidas muestran la etiqueta textual «Vencida» acompañada de un icono; el color no es el único
  indicador (FR-014).
- La edición es en línea: el elemento de la tarea se sustituye por un formulario con los valores
  actuales, «Guardar cambios» y «Cancelar». Cancelar no modifica nada.
- Eliminar usa la confirmación nativa del navegador con un texto en español que incluye el título.
  Si se cancela, la tarea permanece.
- Mensajes vacíos distintos: sin ninguna tarea se invita a crear la primera; con un filtro sin
  resultados se indica que no hay tareas en ese estado.
- El foco se mueve al campo de título tras crear y al elemento editado tras guardar o cancelar, para
  que la operación sea predecible con teclado.
- Todos los textos visibles, etiquetas, mensajes y fechas están en español.

### Manejo de errores

- Validación: mensaje específico en español y ningún cambio de estado.
- Persistencia: mensaje en español y estado anterior intacto; nunca se anuncia como guardado un
  cambio que no llegó a `localStorage`.
- Lectura corrupta: mensaje en español y lista vacía; no se intenta reparar ni migrar datos, ya que
  no hay versiones anteriores en alcance.
- No se añaden reintentos automáticos, colas offline ni mecanismos de recuperación: no están
  pedidos y aumentarían las piezas móviles (Principio I).

### Dependencias justificadas

| Dependencia | Ámbito | Justificación | Alternativa descartada |
|-------------|--------|---------------|------------------------|
| `@playwright/test` | Solo desarrollo | Permite ejecutar las pruebas de aceptación en un navegador real: recargar la página para verificar persistencia, usar `localStorage`, manejar la confirmación nativa y medir la interacción con 200 tareas. Las capacidades nativas no incluyen un controlador de navegador. | Pruebas solo con `node:test` o jsdom: no reproducen recarga real, `localStorage` del navegador, diálogos nativos ni el renderizado visual. |

No se añaden dependencias de runtime, frameworks, librerías de fechas, gestores de estado ni
herramientas de build.

## Estrategia de Pruebas

Cada prueba usa un contexto de navegador limpio y siembra datos solo cuando el escenario lo
necesita. La persistencia se valida con `page.reload()` dentro del mismo contexto. Los escenarios
de vencimiento usan el reloj controlado de Playwright para que «hoy» sea determinista. Los nombres
de las pruebas incluyen el identificador del criterio o requisito cubierto.

| Archivo | Cobertura principal |
|---------|---------------------|
| `tests/e2e/crear-y-persistir.spec.js` | Historia 1 (AC 1-5), FR-001 a FR-007, FR-020, FR-021; creación rápida y estado vacío inicial. |
| `tests/e2e/gestionar-tareas.spec.js` | Historia 2 (AC 1-7), FR-008 a FR-012, FR-020, FR-021; completar, desmarcar, editar, cancelar, eliminar y confirmar. |
| `tests/e2e/vencimiento.spec.js` | Historia 3 (AC 1-4), FR-013 a FR-015, SC-003; ayer/hoy/mañana, completada y sin fecha. |
| `tests/e2e/filtrar-y-ordenar.spec.js` | Historia 4 (AC 1-6), FR-016 a FR-019; filtros, ambos órdenes, desempates, sin fecha al final y cambio automático de filtro al crear. |
| `tests/e2e/interfaz-y-errores.spec.js` | FR-004, FR-005, FR-012, FR-014, FR-019, FR-022, FR-024; textos en español, límites de longitud, fallo de guardado simulado y estados vacíos. |

Además, una prueba de rendimiento dentro de `filtrar-y-ordenar.spec.js` carga 200 tareas y
comprueba que aplicar un filtro y un orden se percibe en menos de 1 segundo (SC-004). La creación
de una tarea con solo título se mide en la suite de la Historia 1 para verificar SC-001. SC-005 es
un resultado de usabilidad con personas y se comprueba mediante el recorrido manual de
`quickstart.md`; no se sustituye por una prueba automatizada. El comando único `npm test` ejecuta
toda la suite automatizada.

## Fases y Entregables

- **Fase 0 — Investigación**: completada en [research.md](./research.md). No quedan
  `NEEDS CLARIFICATION`.
- **Fase 1 — Diseño**: completada con [data-model.md](./data-model.md),
  [contracts/](./contracts/) y [quickstart.md](./quickstart.md).
- **Fase 2 — Tareas**: la generará `/speckit-tasks` en `tasks.md`; este plan no la crea.

Secuencia prevista para la implementación: esqueleto estático y servidor local; modelo y
almacenamiento; creación y persistencia; gestión de tareas; vencimiento; filtros y orden; manejo
de errores y accesibilidad; suite E2E completa.

## Riesgos y Mitigaciones

| Riesgo | Mitigación |
|--------|------------|
| `localStorage` lleno o bloqueado | Escritura transaccional con `try/catch`; no confirmar el estado y mostrar mensaje en español. |
| Desfase de zona horaria en fechas | Guardar y comparar cadenas `AAAA-MM-DD`; no usar conversiones a UTC para el día límite. |
| Pruebas de vencimiento frágiles | Reloj controlado por Playwright y fechas calculadas explícitamente. |
| Módulos ES bloqueados al abrir `file://` | Servir la aplicación con `npm start` y configurar Playwright con `webServer`. |
| Instalación del navegador de Playwright | Documentar `npx playwright install chromium` en `quickstart.md`. |
| Crecimiento de la lista | Orden y filtro en memoria sobre 200 tareas; prueba de rendimiento incluida. |

## Complexity Tracking

No hay violaciones de la constitución ni excepciones que justificar. La estructura elegida evita
capas, servicios, repositorios y dependencias innecesarias.

# Especificación de Funcionalidad: Gestión de tareas para estudiantes

**Rama de la funcionalidad**: `001-gestion-tareas`

**Creada**: 2026-11-05

**Estado**: Borrador

**Entrada**: Descripción del usuario: "Quiero una aplicación de lista de tareas para estudiantes
universitarios que necesitan organizar sus entregas. El usuario puede crear una tarea con un título
obligatorio, una descripción opcional, una fecha límite opcional y una prioridad (alta, media o
baja). Puede marcar una tarea como completada y desmarcarla, editarla y eliminarla. Puede filtrar
las tareas por estado (todas, pendientes, completadas) y ordenarlas por fecha límite o por
prioridad. Las tareas vencidas que no están completadas se deben distinguir visualmente. No hay
usuarios ni inicio de sesión: es una app personal en un solo dispositivo."

## Escenarios de Usuario y Pruebas *(obligatorio)*

### Historia de Usuario 1 - Crear y conservar tareas (Prioridad: P1)

Una estudiante abre la aplicación y anota rápidamente una entrega pendiente ("Informe de
laboratorio de Química"). Si lo desea, añade una descripción, una fecha límite y una prioridad.
La tarea aparece de inmediato en la lista y sigue ahí cuando vuelve a abrir la aplicación días
después.

**Por qué esta prioridad**: sin poder registrar tareas y conservarlas no existe la aplicación.
Por sí sola ya es un producto mínimo útil: una lista de entregas que no se pierde.

**Prueba independiente**: crear varias tareas (una solo con título, otra con todos los campos),
cerrar y volver a abrir la aplicación, y comprobar que todas aparecen con sus datos intactos.

**Escenarios de aceptación**:

1. **Dado** que la aplicación está abierta, **cuando** el usuario escribe un título y confirma,
   **entonces** la tarea aparece en la lista como pendiente, con prioridad media y sin fecha
   límite ni descripción.
2. **Dado** que la aplicación está abierta, **cuando** el usuario crea una tarea indicando título,
   descripción, fecha límite y prioridad alta, **entonces** la tarea aparece en la lista mostrando
   esos cuatro datos.
3. **Dado** que el usuario intenta crear una tarea con el título vacío o formado solo por
   espacios, **cuando** confirma, **entonces** la tarea no se crea y se muestra un mensaje en
   español indicando que el título es obligatorio.
4. **Dado** que existen tareas creadas, **cuando** el usuario cierra o recarga la aplicación y
   vuelve a abrirla, **entonces** ve exactamente las mismas tareas con los mismos datos.
5. **Dado** que no existe ninguna tarea, **cuando** el usuario abre la aplicación, **entonces** ve
   un mensaje que le invita a crear su primera tarea.

---

### Historia de Usuario 2 - Completar, editar y eliminar tareas (Prioridad: P2)

Al entregar el informe, la estudiante lo marca como completado. Si se equivocó, lo desmarca. Si
el profesor cambia la fecha de entrega, edita la tarea. Si una tarea ya no tiene sentido, la
elimina.

**Por qué esta prioridad**: mantener la lista al día es lo que la hace útil más allá del primer
día; sin esto la lista se llena de tareas obsoletas.

**Prueba independiente**: con varias tareas creadas, marcar una como completada, desmarcarla,
editar el título y la fecha de otra, eliminar una tercera, recargar la aplicación y comprobar que
todos los cambios se mantienen.

**Escenarios de aceptación**:

1. **Dado** una tarea pendiente, **cuando** el usuario la marca como completada, **entonces** la
   tarea se muestra como completada y se distingue visualmente de las pendientes.
2. **Dado** una tarea completada, **cuando** el usuario la desmarca, **entonces** vuelve a
   mostrarse como pendiente.
3. **Dado** una tarea existente, **cuando** el usuario edita su título, descripción, fecha límite
   o prioridad y guarda, **entonces** la tarea muestra los nuevos valores.
4. **Dado** que el usuario está editando una tarea, **cuando** deja el título vacío e intenta
   guardar, **entonces** los cambios no se guardan y se muestra el mensaje de título obligatorio.
5. **Dado** que el usuario está editando una tarea, **cuando** cancela la edición, **entonces** la
   tarea conserva sus valores anteriores.
6. **Dado** una tarea existente, **cuando** el usuario la elimina y confirma, **entonces** la
   tarea desaparece de la lista; si no confirma, la tarea se conserva.
7. **Dado** que se ha completado, editado o eliminado una tarea, **cuando** el usuario recarga la
   aplicación, **entonces** el cambio se mantiene.

---

### Historia de Usuario 3 - Detectar tareas vencidas (Prioridad: P3)

La estudiante abre la aplicación y ve de un vistazo qué entregas pendientes ya pasaron su fecha
límite, para atenderlas o renegociarlas cuanto antes.

**Por qué esta prioridad**: aporta mucho valor a estudiantes con entregas, pero depende de que ya
existan tareas con fecha límite (Historia 1).

**Prueba independiente**: crear una tarea pendiente con fecha límite de ayer, otra con fecha de
hoy, otra con fecha de mañana y una completada con fecha de ayer; comprobar que solo la primera
se muestra como vencida.

**Escenarios de aceptación**:

1. **Dado** una tarea pendiente cuya fecha límite es anterior al día actual, **cuando** se
   muestra la lista, **entonces** la tarea se distingue visualmente como vencida mediante un
   indicador que no depende solo del color (por ejemplo, un texto "Vencida").
2. **Dado** una tarea pendiente cuya fecha límite es hoy o posterior, **cuando** se muestra la
   lista, **entonces** la tarea no se muestra como vencida.
3. **Dado** una tarea vencida, **cuando** el usuario la marca como completada, **entonces** deja
   de mostrarse como vencida.
4. **Dado** una tarea pendiente sin fecha límite, **cuando** se muestra la lista, **entonces**
   nunca se muestra como vencida.

---

### Historia de Usuario 4 - Filtrar y ordenar la lista (Prioridad: P4)

Con muchas entregas, la estudiante filtra para ver solo las pendientes y las ordena por fecha
límite para saber qué viene primero, o por prioridad para atacar lo más importante.

**Por qué esta prioridad**: mejora la organización cuando la lista crece, pero la aplicación es
útil sin ella.

**Prueba independiente**: con tareas pendientes y completadas de distintas prioridades y fechas,
aplicar cada filtro y cada orden y comprobar qué tareas aparecen y en qué posición.

**Escenarios de aceptación**:

1. **Dado** tareas pendientes y completadas, **cuando** el usuario elige el filtro "Pendientes",
   **entonces** solo se muestran las tareas no completadas.
2. **Dado** tareas pendientes y completadas, **cuando** el usuario elige el filtro
   "Completadas", **entonces** solo se muestran las tareas completadas.
3. **Dado** cualquier filtro activo, **cuando** el usuario elige "Todas", **entonces** se
   muestran todas las tareas.
4. **Dado** tareas con distintas fechas límite y algunas sin fecha, **cuando** el usuario ordena
   por fecha límite, **entonces** se muestran de la fecha más próxima a la más lejana y las
   tareas sin fecha al final.
5. **Dado** tareas con distintas prioridades, **cuando** el usuario ordena por prioridad,
   **entonces** se muestran primero las de prioridad alta, luego media y luego baja.
6. **Dado** un filtro que no tiene tareas que mostrar, **cuando** se aplica, **entonces** se
   muestra un mensaje indicando que no hay tareas en ese estado.

---

### Casos Límite

- **Título solo con espacios**: se trata como vacío y no se permite crear ni guardar la tarea.
- **Título muy largo**: el título admite hasta 200 caracteres; no se permite escribir más.
- **Descripción muy larga**: la descripción admite hasta 2000 caracteres.
- **Fecha límite en el pasado al crear**: se permite (el estudiante puede anotar algo ya
  atrasado) y la tarea se muestra como vencida de inmediato si está pendiente.
- **Cambio de día con la aplicación abierta**: el indicador de vencida se basa en la fecha
  actual del dispositivo; como mínimo se actualiza al recargar la aplicación.
- **Empates al ordenar**: con la misma fecha límite (o la misma prioridad), se usa el criterio
  secundario (prioridad o fecha límite, respectivamente) y, si persiste el empate, la tarea
  creada más recientemente va primero.
- **Quitar la fecha límite al editar**: se permite; la tarea pasa a no tener fecha y deja de
  poder estar vencida.
- **Fallo al guardar los datos** (por ejemplo, almacenamiento del dispositivo lleno): se muestra
  un mensaje en español indicando que el cambio no se pudo guardar, y no se muestra como guardado
  algo que no lo está.
- **Crear una tarea con el filtro "Completadas" activo**: la tarea se crea igual y el filtro
  cambia automáticamente a "Todas" para que el usuario vea la tarea recién creada.

## Requisitos *(obligatorio)*

### Requisitos Funcionales

**Creación**

- **FR-001**: El sistema DEBE permitir crear una tarea indicando solo el título y confirmando con
  una única acción (tecla Enter o un botón).
- **FR-002**: El sistema DEBE permitir indicar opcionalmente al crear la tarea una descripción,
  una fecha límite (día) y una prioridad entre alta, media y baja.
- **FR-003**: El sistema DEBE asignar prioridad "media" cuando el usuario no elija una.
- **FR-004**: El sistema DEBE rechazar títulos vacíos o formados solo por espacios y mostrar un
  mensaje en español que explique el motivo.
- **FR-005**: El sistema DEBE limitar el título a 200 caracteres y la descripción a 2000.
- **FR-006**: El campo para crear una tarea DEBE estar visible al abrir la aplicación, sin
  necesidad de navegar a otra pantalla.
- **FR-007**: Toda tarea nueva DEBE crearse en estado pendiente.

**Gestión**

- **FR-008**: El sistema DEBE permitir marcar una tarea pendiente como completada y desmarcar una
  tarea completada para que vuelva a estar pendiente.
- **FR-009**: El sistema DEBE permitir editar el título, la descripción, la fecha límite y la
  prioridad de una tarea existente, aplicando las mismas validaciones que en la creación.
- **FR-010**: El sistema DEBE permitir cancelar una edición sin modificar la tarea.
- **FR-011**: El sistema DEBE permitir eliminar una tarea, pidiendo confirmación antes de
  eliminarla definitivamente.
- **FR-012**: Las tareas completadas DEBEN distinguirse visualmente de las pendientes.

**Vencimiento**

- **FR-013**: El sistema DEBE considerar vencida una tarea cuando está pendiente, tiene fecha
  límite y esa fecha es anterior al día actual del dispositivo.
- **FR-014**: El sistema DEBE distinguir visualmente las tareas vencidas mediante un indicador
  que no dependa solo del color.
- **FR-015**: Las tareas completadas o sin fecha límite NUNCA DEBEN mostrarse como vencidas.

**Filtrado y orden**

- **FR-016**: El sistema DEBE ofrecer los filtros "Todas", "Pendientes" y "Completadas"; el filtro
  inicial al abrir la aplicación es "Todas". Si se crea una tarea con el filtro "Completadas"
  activo, el filtro DEBE cambiar a "Todas".
- **FR-017**: El sistema DEBE permitir ordenar la lista por fecha límite (más próxima primero,
  sin fecha al final) o por prioridad (alta, media, baja); el orden inicial es por fecha límite.
- **FR-018**: Ante empates, el sistema DEBE aplicar los criterios secundarios descritos en Casos
  Límite.
- **FR-019**: El sistema DEBE mostrar un mensaje en español cuando la lista, o el filtro activo,
  no tenga tareas que mostrar.

**Persistencia e idioma**

- **FR-020**: El sistema DEBE guardar cada creación, edición, cambio de estado y eliminación en el
  momento en que ocurre, sin acción explícita de "guardar" la lista.
- **FR-021**: Al cerrar, recargar y volver a abrir la aplicación, el sistema DEBE mostrar
  exactamente las mismas tareas con los mismos datos y estados.
- **FR-022**: El sistema DEBE informar al usuario en español si un cambio no se pudo guardar.
- **FR-023**: El sistema DEBE funcionar sin cuentas de usuario ni inicio de sesión, guardando los
  datos solo en el dispositivo del usuario.
- **FR-024**: Todos los textos de la interfaz (etiquetas, botones, mensajes, marcadores de
  posición y fechas) DEBEN estar en español.

### Entidades Clave

- **Tarea**: una entrega o actividad que el estudiante debe realizar. Atributos: título
  (obligatorio, hasta 200 caracteres), descripción (opcional, hasta 2000 caracteres), fecha
  límite (opcional, un día del calendario sin hora), prioridad (alta, media o baja; por defecto
  media), estado (pendiente o completada) y momento de creación (usado para desempatar al
  ordenar). La condición de "vencida" no se guarda: se deduce del estado, la fecha límite y la
  fecha actual.

## Criterios de Éxito *(obligatorio)*

### Resultados Medibles

- **SC-001**: Un usuario que abre la aplicación por primera vez crea una tarea con solo título en
  menos de 10 segundos.
- **SC-002**: El 100 % de las tareas creadas, editadas, completadas o eliminadas conservan su
  último estado tras cerrar y volver a abrir la aplicación.
- **SC-003**: El 100 % de las tareas pendientes con fecha límite pasada se muestran como vencidas,
  y ninguna tarea completada o sin fecha se muestra como vencida.
- **SC-004**: Con al menos 200 tareas, filtrar u ordenar la lista muestra el resultado de forma
  percibida como inmediata (en menos de 1 segundo).
- **SC-005**: Al menos 9 de cada 10 usuarios de prueba completan sin ayuda las acciones de
  completar, editar, eliminar, filtrar y ordenar en su primer intento.
- **SC-006**: Cada criterio de aceptación de esta especificación está cubierto por al menos una
  prueba automatizada que pasa.

## Supuestos

- Los usuarios son estudiantes universitarios que usan la aplicación en un único dispositivo
  personal; no se necesita sincronización entre dispositivos ni compartir tareas.
- La fecha límite es un día, sin hora; una tarea con fecha de hoy no está vencida hasta que
  comienza el día siguiente.
- La prioridad por defecto es "media", para que crear una tarea solo requiera el título
  (Principio II de la constitución).
- El filtro y el orden elegidos no necesitan conservarse entre sesiones; al abrir la aplicación
  se usan los valores iniciales ("Todas" y orden por fecha límite).
- Las fechas se muestran en formato español (día/mes/año).
- Eliminar una tarea es definitivo tras confirmar; no hay papelera ni opción de deshacer.
- Si la misma aplicación se abre a la vez en varias ventanas, no se garantiza que se sincronicen
  entre sí sin recargar.
- Quedan fuera de alcance: recordatorios o notificaciones, etiquetas o asignaturas, subtareas,
  tareas recurrentes, búsqueda por texto, exportar o importar datos y adjuntar archivos.

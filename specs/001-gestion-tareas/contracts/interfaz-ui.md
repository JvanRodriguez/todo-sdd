# Contrato de Interfaz de Usuario

**Funcionalidad**: `001-gestion-tareas`
**Fecha**: 2026-11-06
**Ámbito**: Elementos, textos y comportamientos observables que la suite de Playwright usa como
contrato. Los textos visibles y los `data-testid` son acuerdos de prueba: si cambian, deben
actualizarse este documento y las pruebas en el mismo cambio.

## Principios del contrato

- La aplicación tiene una sola pantalla; el formulario de creación está visible al cargar.
- El título es el único campo obligatorio para crear.
- Todos los textos visibles están en español.
- Las pruebas deben preferir roles y etiquetas accesibles; `data-testid` se reserva para elementos
  cuya selección por rol sería ambigua o inestable.
- La interfaz nunca muestra un cambio como guardado si la escritura en `localStorage` falló.

## Estructura y elementos

### Encabezado

| Elemento | Contrato |
|----------|----------|
| Título de la página | Texto visible «Mis tareas». |
| Encabezado principal | Un único `<h1>` con «Mis tareas». |

### Formulario de creación (siempre visible)

| Elemento | `data-testid` | Contrato |
|----------|---------------|----------|
| Formulario | `form-crear` | Contiene todos los campos de creación. Confirmar con Enter o con el botón crea la tarea. |
| Campo de título | `entrada-titulo` | `<input>` obligatorio, `maxlength="200"`, asociado a la etiqueta «Título». Es el primer campo y recibe el foco inicial. |
| Campo de descripción | `entrada-descripcion` | `<textarea>` opcional, `maxlength="2000"`, etiqueta «Descripción (opcional)». |
| Campo de fecha límite | `entrada-fecha` | `<input type="date">` opcional, etiqueta «Fecha límite (opcional)». Formato de valor `AAAA-MM-DD`. |
| Selector de prioridad | `selector-prioridad` | `<select>` con opciones «Alta» (`alta`), «Media» (`media`, seleccionada por defecto) y «Baja» (`baja`), etiqueta «Prioridad». |
| Botón de creación | `boton-crear` | Texto «Añadir tarea». |
| Mensajes de error | `mensajes` | Región con `role="alert"` y `aria-live="assertive"` para errores de validación o guardado. |

### Controles de vista

| Elemento | `data-testid` | Contrato |
|----------|---------------|----------|
| Filtro «Todas» | `filtro-todas` | Seleccionado al abrir. Muestra todas las tareas. Texto «Todas». |
| Filtro «Pendientes» | `filtro-pendientes` | Muestra solo tareas no completadas. Texto «Pendientes». |
| Filtro «Completadas» | `filtro-completadas` | Muestra solo tareas completadas. Texto «Completadas». |
| Grupo de filtros | `grupo-filtros` | Agrupación con etiqueta accesible «Filtrar por estado». |
| Selector de orden | `selector-orden` | `<select>` con opciones «Fecha límite» (`fecha`, seleccionada por defecto) y «Prioridad» (`prioridad`), etiqueta «Ordenar por». |

### Lista de tareas

| Elemento | `data-testid` | Contrato |
|----------|---------------|----------|
| Lista | `lista-tareas` | Contenedor de todas las tareas visibles. |
| Tarea | `tarea` | Cada elemento tiene además `data-id="<id>"`. Es la unidad de una tarea. |
| Título de la tarea | `tarea-titulo` | Muestra el título guardado. |
| Descripción | `tarea-descripcion` | Presente solo si la descripción no está vacía. |
| Fecha | `tarea-fecha` | Presente solo si hay fecha límite; se muestra como `dd/mm/aaaa`. |
| Prioridad | `tarea-prioridad` | Muestra «Alta», «Media» o «Baja». |
| Casilla de completada | `checkbox-completada` | `<input type="checkbox">`; marcada si la tarea está completada. Su etiqueta accesible incluye el título. |
| Etiqueta de completada | `tarea-completada` | Visible solo si está completada; texto «Completada». |
| Etiqueta de vencida | `tarea-vencida` | Visible solo si está vencida; texto «Vencida» con un icono o símbolo adicional. No depende solo del color. |
| Botón editar | `boton-editar` | Texto «Editar». |
| Botón eliminar | `boton-eliminar` | Texto «Eliminar». |

### Formulario de edición en línea

Al pulsar «Editar», el contenido de la tarea se sustituye por un formulario dentro de la lista.

| Elemento | `data-testid` | Contrato |
|----------|---------------|----------|
| Formulario | `form-editar` | Contiene los campos editables de esa tarea. |
| Título | `editar-titulo` | Valor actual; mismas reglas y límite que en la creación. |
| Descripción | `editar-descripcion` | Valor actual. |
| Fecha | `editar-fecha` | Valor actual o vacío. |
| Prioridad | `editar-prioridad` | Valor actual. |
| Guardar | `boton-guardar` | Texto «Guardar cambios». |
| Cancelar | `boton-cancelar` | Texto «Cancelar». No modifica la tarea. |

### Estado vacío

| Situación | `data-testid` | Texto exacto |
|-----------|---------------|--------------|
| No hay ninguna tarea | `estado-vacio` | «Aún no tienes tareas. Crea tu primera tarea para empezar.» |
| Filtro «Pendientes» sin resultados | `estado-vacio` | «No hay tareas pendientes.» |
| Filtro «Completadas» sin resultados | `estado-vacio` | «No hay tareas completadas.» |

## Textos contractuales

| Uso | Texto exacto |
|-----|--------------|
| Título de la aplicación | «Mis tareas» |
| Etiqueta de título | «Título» |
| Marcador de posición del título | «¿Qué tienes que entregar?» |
| Etiqueta de descripción | «Descripción (opcional)» |
| Etiqueta de fecha | «Fecha límite (opcional)» |
| Etiqueta de prioridad | «Prioridad» |
| Etiqueta de filtros | «Filtrar por estado» |
| Etiqueta de orden | «Ordenar por» |
| Botón de creación | «Añadir tarea» |
| Botón de edición | «Editar» |
| Botón de eliminación | «Eliminar» |
| Botón de guardado | «Guardar cambios» |
| Botón de cancelación | «Cancelar» |
| Etiqueta de completada | «Completada» |
| Etiqueta de vencida | «Vencida» |
| Título obligatorio | «El título es obligatorio.» |
| Fallo al guardar | «No se pudo guardar el cambio. Inténtalo de nuevo.» |
| Fallo al cargar | «No se pudieron cargar las tareas guardadas.» |
| Confirmación de eliminación | «¿Eliminar la tarea “{título}”? Esta acción no se puede deshacer.» |

La confirmación de eliminación se muestra con el diálogo nativo del navegador. El marcador
`{título}` se sustituye por el título de la tarea. Aceptar elimina; cancelar conserva la tarea.

## Comportamientos observables

1. **Crear**: al enviar el formulario con un título válido, la tarea aparece como pendiente, con
   prioridad media si no se eligió otra, sin fecha si no se indicó y sin descripción visible si
   está vacía. El formulario se limpia y el foco vuelve al título.
2. **Título inválido**: un título vacío o solo con espacios no crea la tarea; se muestra el texto
   «El título es obligatorio.» en `mensajes`.
3. **Persistencia**: tras crear, editar, completar, desmarcar o eliminar, recargar la página
   conserva exactamente el nuevo estado.
4. **Completar/desmarcar**: la casilla cambia el estado; la etiqueta «Completada» aparece o
   desaparece y la tarea se distingue visualmente.
5. **Editar**: guardar aplica los nuevos valores; cancelar conserva los anteriores; un título
   inválido impide guardar y muestra el mensaje.
6. **Eliminar**: el diálogo nativo se muestra antes de eliminar; aceptar quita la tarea de la
   lista y cancelar la conserva.
7. **Vencida**: una tarea pendiente con fecha anterior a hoy muestra «Vencida»; hoy o una fecha
   futura no la muestran; una tarea completada o sin fecha nunca la muestra.
8. **Filtros**: «Todas» muestra pendientes y completadas; «Pendientes» solo no completadas;
   «Completadas» solo completadas. Si el filtro no tiene resultados, se muestra el estado vacío
   correspondiente.
9. **Cambio automático de filtro**: si se crea una tarea con «Completadas» activo, el filtro pasa a
   «Todas» y la nueva tarea es visible.
10. **Orden por fecha**: fecha más próxima primero, sin fecha al final; desempates según
    `data-model.md`.
11. **Orden por prioridad**: alta, media, baja; desempates según `data-model.md`.
12. **Fallo de guardado**: si `localStorage.setItem` lanza una excepción, la tarea no se añade ni
    se modifica en la lista visible y aparece «No se pudo guardar el cambio. Inténtalo de
    nuevo.».
13. **Límites**: el título no admite más de 200 caracteres y la descripción más de 2000; los
    campos aplican `maxlength` y el dominio vuelve a validar.
14. **Fechas visibles**: se muestran en formato `dd/mm/aaaa`.

## Accesibilidad mínima

- Cada campo tiene una etiqueta asociada.
- El formulario de creación se puede usar solo con teclado; Enter en el título envía.
- Los mensajes de error se anuncian mediante `role="alert"` y `aria-live="assertive"`.
- Los estados «Completada» y «Vencida» incluyen texto, no solo color.
- La lista usa elementos semánticos (`<ul>`/`<li>`) o una estructura equivalente con roles
  accesibles.
- El foco se gestiona de forma predecible al crear, guardar y cancelar una edición.

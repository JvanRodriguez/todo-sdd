# Guía de Validación Rápida: Gestión de tareas para estudiantes

**Funcionalidad**: `001-gestion-tareas`
**Fecha**: 2026-11-06
**Referencias**: [spec.md](./spec.md) · [plan.md](./plan.md) · [data-model.md](./data-model.md) ·
[contracts/](./contracts/)

Esta guía permite comprobar de extremo a extremo que la funcionalidad cumple la especificación.
No contiene la implementación ni el código de las pruebas.

## Requisitos previos

- Node.js 20 LTS o superior y npm 10 o superior.
- Un navegador de escritorio moderno (la suite automatizada usa Chromium).
- Terminal en la raíz del repositorio.

## Instalación

```bash
npm install
npx playwright install chromium
```

`npm install` instala solo la dependencia de desarrollo `@playwright/test`.
`npx playwright install chromium` descarga el navegador que usa la suite.

## Ejecutar las pruebas

Comando único documentado:

```bash
npm test
```

Resultado esperado:

- La suite termina con código de salida 0.
- Todas las pruebas de las historias 1 a 4 pasan.
- Se cubren creación y persistencia, gestión, vencimiento, filtros, orden, validaciones, estados
  vacíos, textos en español y fallo de guardado.
- La prueba de rendimiento con 200 tareas completa el filtrado y el orden en menos de 1 segundo.

La suite usa un contexto de navegador limpio por prueba, recarga la página para verificar la
persistencia y controla el reloj para los escenarios de vencimiento. No depende de la fecha real
del equipo.

## Ejecutar la aplicación

```bash
npm start
```

Abrir `http://localhost:4173` en el navegador. El servidor es estático y no requiere backend ni
configuración adicional.

Para detenerlo, usar `Ctrl+C` en la terminal.

## Recorrido manual de validación

Realizar las comprobaciones en este orden. Cada una referencia el criterio o requisito que valida.

### 1. Creación y persistencia (Historia 1, FR-001 a FR-007 y FR-020/021)

1. Abrir la aplicación con el almacenamiento del sitio vacío.
   - **Esperado**: se ve el formulario de creación sin navegar y el mensaje «Aún no tienes tareas.
     Crea tu primera tarea para empezar.».
2. Escribir «Informe de laboratorio de Química» en el título y pulsar Enter.
   - **Esperado**: la tarea aparece como pendiente, con prioridad «Media», sin fecha y sin
     descripción visible.
3. Crear otra tarea indicando título, descripción, fecha límite y prioridad «Alta».
   - **Esperado**: la tarea muestra los cuatro datos; la fecha se ve como `dd/mm/aaaa`.
4. Intentar crear una tarea con el título vacío y con solo espacios.
   - **Esperado**: no se crea ninguna tarea y aparece «El título es obligatorio.».
5. Recargar la página.
   - **Esperado**: las dos tareas anteriores siguen ahí con los mismos datos y estados.

### 2. Gestión (Historia 2, FR-008 a FR-012 y FR-020/021)

1. Marcar una tarea pendiente como completada.
   - **Esperado**: la casilla queda marcada, aparece la etiqueta «Completada» y la tarea se
     distingue visualmente.
2. Desmarcarla.
   - **Esperado**: vuelve a mostrarse como pendiente.
3. Editar el título, la descripción, la fecha y la prioridad de una tarea y guardar.
   - **Esperado**: la tarea muestra los nuevos valores.
4. Entrar en edición, cambiar el título a espacios y guardar.
   - **Esperado**: no se guarda y aparece «El título es obligatorio.».
5. Entrar en edición, modificar un valor y pulsar «Cancelar».
   - **Esperado**: la tarea conserva los valores anteriores.
6. Pulsar «Eliminar» y aceptar la confirmación.
   - **Esperado**: la tarea desaparece de la lista.
7. Repetir «Eliminar» y cancelar la confirmación.
   - **Esperado**: la tarea permanece.
8. Recargar la página.
   - **Esperado**: se mantienen el completado, las ediciones y las eliminaciones.

### 3. Vencimiento (Historia 3, FR-013 a FR-015 y SC-003)

Para el recorrido manual, usar fechas relativas al día actual del equipo.

1. Crear una tarea pendiente con fecha de ayer.
   - **Esperado**: muestra «Vencida» con texto e icono, no solo color.
2. Crear una tarea pendiente con fecha de hoy y otra con fecha de mañana.
   - **Esperado**: ninguna se muestra como vencida.
3. Crear una tarea completada con fecha de ayer.
   - **Esperado**: no se muestra como vencida.
4. Crear una tarea pendiente sin fecha.
   - **Esperado**: no se muestra como vencida.
5. Marcar como completada la tarea vencida del paso 1.
   - **Esperado**: deja de mostrarse como vencida.

### 4. Filtros y orden (Historia 4, FR-016 a FR-019)

Preparar tareas pendientes y completadas con distintas prioridades y fechas, incluyendo algunas
sin fecha.

1. Elegir «Pendientes».
   - **Esperado**: solo se muestran tareas no completadas.
2. Elegir «Completadas».
   - **Esperado**: solo se muestran tareas completadas.
3. Elegir «Todas».
   - **Esperado**: se muestran todas.
4. Ordenar por «Fecha límite».
   - **Esperado**: primero la fecha más próxima; las tareas sin fecha al final. Ante empates se
     aplican prioridad y luego creación más reciente, según [data-model.md](./data-model.md).
5. Ordenar por «Prioridad».
   - **Esperado**: primero alta, luego media y luego baja. Ante empates se aplican fecha límite y
     luego creación más reciente.
6. Elegir un filtro sin resultados.
   - **Esperado**: aparece «No hay tareas pendientes.» o «No hay tareas completadas.» según
     corresponda.
7. Con el filtro «Completadas» activo, crear una tarea nueva.
   - **Esperado**: la tarea se crea, el filtro cambia a «Todas» y la nueva tarea es visible.

### 5. Validaciones, límites e idioma (FR-004, FR-005, FR-014, FR-022 y FR-024)

1. Intentar escribir más de 200 caracteres en el título y más de 2000 en la descripción.
   - **Esperado**: los campos no admiten más allá del límite.
2. Revisar etiquetas, botones, mensajes, prioridades, filtros, orden y fechas.
   - **Esperado**: todo está en español y las fechas usan `dd/mm/aaaa`.
3. Simular un fallo de guardado (la prueba automatizada lo hace sustituyendo
   `localStorage.setItem` por una función que lanza un error).
   - **Esperado**: la lista no muestra el cambio como guardado y aparece «No se pudo guardar el
     cambio. Inténtalo de nuevo.».

## Cobertura de criterios de aceptación

| Historia | Criterios | Prueba automatizada |
|----------|-----------|---------------------|
| Historia 1 | AC 1-5 | `tests/e2e/crear-y-persistir.spec.js` |
| Historia 2 | AC 1-7 | `tests/e2e/gestionar-tareas.spec.js` |
| Historia 3 | AC 1-4 | `tests/e2e/vencimiento.spec.js` |
| Historia 4 | AC 1-6 | `tests/e2e/filtrar-y-ordenar.spec.js` |
| Transversal | FR-004, FR-005, FR-012, FR-014, FR-019, FR-022, FR-024 | `tests/e2e/interfaz-y-errores.spec.js` |

El contrato de elementos y textos usado por las pruebas está en
[contracts/interfaz-ui.md](./contracts/interfaz-ui.md). El contrato de datos guardados está en
[contracts/persistencia.schema.json](./contracts/persistencia.schema.json).

## Comprobaciones de cierre

- [ ] `npm test` termina correctamente.
- [ ] Las tareas sobreviven a recargar la página.
- [ ] Las tareas vencidas se distinguen con texto, no solo con color.
- [ ] No se añadieron dependencias de runtime.
- [ ] Toda la interfaz y la documentación están en español.

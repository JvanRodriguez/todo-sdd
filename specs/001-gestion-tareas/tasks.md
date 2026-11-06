---
description: "Lista de tareas para la implementación de la gestión de tareas para estudiantes"
---

# Tareas: Gestión de tareas para estudiantes

**Entrada**: Documentos de diseño en `specs/001-gestion-tareas/`

**Prerrequisitos**: [plan.md](./plan.md) (obligatorio), [spec.md](./spec.md) (obligatorio para
historias), [research.md](./research.md), [data-model.md](./data-model.md),
[contracts/](./contracts/) y [quickstart.md](./quickstart.md).

**Pruebas**: Obligatorias. La especificación (SC-006) y la constitución (Principio III) exigen al
menos una prueba automatizada por criterio de aceptación. Cada historia incluye primero sus
pruebas E2E; deben escribirse y fallar antes de implementar.

**Organización**: Las tareas se agrupan por historia de usuario para permitir implementar y
validar cada incremento de forma independiente.

## Formato: `[ID] [P?] [Historia] Descripción`

- **[P]**: se puede ejecutar en paralelo (archivos distintos, sin dependencias pendientes).
- **[Historia]**: `[US1]`, `[US2]`, `[US3]` o `[US4]`; solo en las fases de historia.
- Todas las descripciones incluyen la ruta exacta del archivo que se va a crear o modificar.

## Convenciones de rutas

Proyecto único en la raíz del repositorio: `index.html`, `styles.css`, `src/`, `scripts/`,
`tests/`, `playwright.config.js` y `package.json`.

---

## Fase 1: Configuración inicial (infraestructura compartida)

**Propósito**: preparar el proyecto estático, el servidor local y la infraestructura de pruebas.

- [ ] T001 Crear `package.json` con `"private": true`, `"type": "module"`, scripts `start` (`node scripts/servidor.mjs`) y `test` (`playwright test`), y la dependencia de desarrollo `@playwright/test`; añadir `node_modules/`, `test-results/` y `playwright-report/` a `.gitignore`.
- [ ] T002 [P] Crear `scripts/servidor.mjs` con un servidor estático mínimo usando `node:http`, `node:fs` y `node:path`, que sirva la raíz en el puerto 4173, resuelva `/` a `index.html` y asigne tipos MIME para HTML, CSS y JavaScript.
- [ ] T003 [P] Crear `playwright.config.js` con `testDir: "tests/e2e"`, proyecto Chromium, `baseURL: "http://localhost:4173"`, `webServer` que ejecute `npm start`, `reuseExistingServer: true` y traza en el primer reintento.
- [ ] T004 [P] Crear `index.html` con `lang="es"`, encabezado `<h1>Mis tareas</h1>`, contenedores vacíos para el formulario de creación, los controles de vista, la lista y la región de mensajes, enlace a `styles.css` y carga de `src/app.js` como módulo.
- [ ] T005 [P] Crear `styles.css` con la maquetación base de una sola columna, tipografía legible, estilos de foco visibles y organización responsive para el formulario y la lista.
- [ ] T006 [P] Crear `tests/apoyo/datos.js` con fábricas de tareas válidas, utilidades de fechas `AAAA-MM-DD` y funciones para sembrar y leer el sobre `{ version: 1, tareas }` de `localStorage` en las pruebas.

---

## Fase 2: Fundamentos (prerrequisitos bloqueantes)

**Propósito**: entidad, validaciones y persistencia compartidas por todas las historias.

**⚠️ CRÍTICO**: ninguna historia puede empezar antes de completar esta fase.

- [ ] T007 Implementar en `src/modelo.js` la creación y validación de la entidad `Tarea` según `data-model.md`: `id` único con `crypto.randomUUID()`; `titulo` recortado entre 1 y 200 caracteres (vacío o solo espacios inválido); `descripcion` de hasta 2000 caracteres con `""` por defecto; `fechaLimite` `null` o `AAAA-MM-DD` de fecha real, permitiendo fechas pasadas; `prioridad` `alta|media|baja` con `media` por defecto; `completada` con `false` por defecto; `creadaEn` en ISO 8601.
- [ ] T008 [P] Implementar `src/almacen.js` para cargar y guardar el sobre versionado en la clave `todo-sdd:tareas:v1`: validar la forma al leer, devolver lista vacía si no existe, informar datos corruptos y aplicar escritura transaccional con `try/catch` en `localStorage.setItem`, sin confirmar el cambio si falla.
- [ ] T009 Implementar en `src/app.js` el arranque de la aplicación: cargar el estado con `almacen.js`, crear el estado en memoria de filtro y orden, renderizar la estructura inicial, conectar la región `aria-live` y dejar preparados los puntos de extensión para las historias; depende de T007 y T008.

**Punto de control**: el modelo, el contrato de persistencia y el arranque de la interfaz están listos; las historias pueden comenzar.

---

## Fase 3: Historia de Usuario 1 — Crear y conservar tareas (Prioridad: P1) 🎯 MVP

**Objetivo**: permitir crear una tarea con solo título y conservarla al recargar, con campos
opcionales y validación en español.

**Prueba independiente**: crear una tarea solo con título y otra con todos los campos; recargar y
comprobar que ambas siguen ahí con sus datos y que un título vacío no crea nada.

### Pruebas de la Historia 1

- [ ] T010 [P] [US1] Escribir primero y comprobar que fallan las pruebas E2E en `tests/e2e/crear-y-persistir.spec.js` para los AC 1-5 de la Historia 1: valores por defecto, creación con todos los campos, título vacío o solo espacios, persistencia tras `page.reload()`, estado vacío inicial y creación con solo título en menos de 10 segundos (SC-001); verificar también la forma del sobre en `localStorage` contra `specs/001-gestion-tareas/contracts/persistencia.schema.json` y los textos de `specs/001-gestion-tareas/contracts/interfaz-ui.md`.

### Implementación de la Historia 1

- [ ] T011 [P] [US1] Añadir a `index.html` el formulario de creación con `data-testid="form-crear"`: título obligatorio `maxlength="200"` con foco inicial, descripción `maxlength="2000"`, fecha `type="date"`, prioridad con `media` seleccionada y botón «Añadir tarea», con etiquetas y mensajes del contrato `contracts/interfaz-ui.md`.
- [ ] T012 [P] [US1] Añadir a `styles.css` los estilos del formulario de creación, de las tarjetas de tarea y del estado vacío sin tareas, manteniendo el foco visible y una jerarquía clara.
- [ ] T013 [P] [US1] Implementar en `src/modelo.js` el formateo de fecha `dd/mm/aaaa` desde `AAAA-MM-DD` y el comparador inicial por fecha límite: fecha más próxima primero, sin fecha al final, desempate por prioridad `alta < media < baja` y después por `creadaEn` descendente, con la tarea añadida después primero si hay empate.
- [ ] T014 [US1] Implementar en `src/app.js` el flujo de creación: leer el formulario, validar con `src/modelo.js`, mostrar «El título es obligatorio.» cuando corresponda, construir la tarea, persistirla con `src/almacen.js`, confirmar el estado solo si el guardado funciona, limpiar el formulario y devolver el foco al título; depende de T011, T013 y T008.
- [ ] T015 [US1] Implementar en `src/app.js` el renderizado de la lista de tareas con título, descripción solo si existe, fecha en `dd/mm/aaaa`, prioridad y los `data-testid` del contrato, aplicando el comparador inicial y mostrando «Aún no tienes tareas. Crea tu primera tarea para empezar.» cuando la lista esté vacía; depende de T014.
- [ ] T016 [US1] Completar en `src/app.js` la carga inicial y el renderizado de tareas guardadas, mostrando «No se pudieron cargar las tareas guardadas.» si el sobre es inválido y conservando los datos existentes tras una recarga; depende de T015.

**Punto de control**: la Historia 1 funciona y se puede validar de forma independiente; es el MVP.

---

## Fase 4: Historia de Usuario 2 — Completar, editar y eliminar tareas (Prioridad: P2)

**Objetivo**: mantener la lista al día mediante completado, edición en línea y eliminación con
confirmación, conservando todos los cambios.

**Prueba independiente**: crear varias tareas, completar una, desmarcarla, editar otra, cancelar
una edición, eliminar una tercera y recargar; todos los cambios deben mantenerse.

### Pruebas de la Historia 2

- [ ] T017 [P] [US2] Escribir primero y comprobar que fallan las pruebas E2E en `tests/e2e/gestionar-tareas.spec.js` para los AC 1-7 de la Historia 2: completar, desmarcar, editar todos los campos, título inválido al editar, cancelar edición, eliminar aceptando y cancelando la confirmación nativa, y persistencia de cada cambio tras recargar; usar los textos y `data-testid` de `contracts/interfaz-ui.md`.

### Implementación de la Historia 2

- [ ] T018 [P] [US2] Ampliar en `src/app.js` el renderizado de cada tarea con la casilla `data-testid="checkbox-completada"`, la etiqueta textual «Completada» cuando corresponda y los botones «Editar» y «Eliminar»; depende de T015.
- [ ] T019 [P] [US2] Añadir a `styles.css` el estado visual de tarea completada, incluyendo la etiqueta textual además del estilo, sin depender solo del color.
- [ ] T020 [US2] Implementar en `src/app.js` el cambio de estado al marcar o desmarcar la casilla, persistirlo inmediatamente y volver a renderizar solo si el guardado tuvo éxito; depende de T018 y T008.
- [ ] T021 [US2] Implementar en `src/app.js` la edición en línea con los campos actuales, validación compartida con la creación, «Guardar cambios» y «Cancelar»; cancelar debe conservar los valores originales y el foco debe volver al elemento editado; depende de T020.
- [ ] T022 [US2] Implementar en `src/app.js` la eliminación con `window.confirm` y el texto «¿Eliminar la tarea “{título}”? Esta acción no se puede deshacer.», persistiendo el borrado solo si se acepta; depende de T021.

**Punto de control**: las Historias 1 y 2 funcionan de forma independiente y conjunta.

---

## Fase 5: Historia de Usuario 3 — Detectar tareas vencidas (Prioridad: P3)

**Objetivo**: señalar de forma textual y visual las tareas pendientes cuya fecha límite ya pasó.

**Prueba independiente**: con tareas de ayer, hoy, mañana, una completada de ayer y una sin fecha,
solo la pendiente de ayer debe mostrar «Vencida».

### Pruebas de la Historia 3

- [ ] T023 [P] [US3] Escribir primero y comprobar que fallan las pruebas E2E en `tests/e2e/vencimiento.spec.js` para los AC 1-4 de la Historia 3, usando el reloj controlado de Playwright: vencida si es anterior a hoy, no vencida si es hoy o futura, deja de estar vencida al completarla y nunca vencida si no tiene fecha o está completada; comprobar el texto «Vencida» y que no depende solo del color.

### Implementación de la Historia 3

- [ ] T024 [P] [US3] Implementar en `src/modelo.js` el cálculo del día local `AAAA-MM-DD` y `esVencida(tarea, hoy)` con la regla `!completada && fechaLimite !== null && fechaLimite < hoy`; hoy no es vencida y el valor nunca se persiste.
- [ ] T025 [US3] Renderizar en `src/app.js` la etiqueta `data-testid="tarea-vencida"` con el texto «Vencida» y un icono o símbolo adicional cuando `esVencida` sea verdadero, recalculándola en cada render; depende de T024 y T018.
- [ ] T026 [P] [US3] Añadir a `styles.css` el estilo de tarea vencida, asegurando que el texto «Vencida» y el icono aporten la información sin depender solo del color.

**Punto de control**: las tres primeras historias funcionan de forma independiente.

---

## Fase 6: Historia de Usuario 4 — Filtrar y ordenar la lista (Prioridad: P4)

**Objetivo**: ofrecer filtros por estado y orden por fecha límite o prioridad, con desempates y
mensajes de lista vacía.

**Prueba independiente**: con tareas pendientes y completadas de distintas prioridades y fechas,
aplicar cada filtro y cada orden, comprobar posiciones, desempates y mensajes de filtro vacío; con
200 tareas, el resultado debe percibirse en menos de 1 segundo.

### Pruebas de la Historia 4

- [ ] T027 [P] [US4] Escribir primero y comprobar que fallan las pruebas E2E en `tests/e2e/filtrar-y-ordenar.spec.js` para los AC 1-6 de la Historia 4: filtros «Todas», «Pendientes» y «Completadas»; orden por fecha con las tareas sin fecha al final; orden por prioridad alta, media y baja; desempates de `data-model.md`; mensajes de filtro vacío; cambio automático a «Todas» al crear con «Completadas» activo; y rendimiento con 200 tareas en menos de 1 segundo (SC-004).
- [ ] T028 [P] [US4] Añadir a `index.html` los controles de filtro `filtro-todas`, `filtro-pendientes` y `filtro-completadas` agrupados con la etiqueta «Filtrar por estado», y el selector `selector-orden` con «Fecha límite» y «Prioridad» según `contracts/interfaz-ui.md`.
- [ ] T029 [P] [US4] Añadir a `styles.css` los estilos de los controles de filtro y orden, manteniendo el foco visible y la coherencia con el formulario.
- [ ] T030 [P] [US4] Implementar en `src/modelo.js` los predicados de filtro y los dos comparadores completos de `data-model.md`: prioridad `alta < media < baja`, fechas ascendentes con las tareas sin fecha al final y desempate final por `creadaEn` descendente, con la tarea añadida después primero si hay empate.
- [ ] T031 [US4] Implementar en `src/app.js` el estado y los manejadores de filtro y orden, aplicar filtro antes de ordenar y renderizar, y cambiar automáticamente a «Todas» tras crear una tarea si el filtro activo era «Completadas»; depende de T028 y T030.
- [ ] T032 [US4] Implementar en `src/app.js` los mensajes de lista vacía según el filtro: «No hay tareas pendientes.» y «No hay tareas completadas.», manteniendo el mensaje inicial de la Historia 1 cuando no existe ninguna tarea; depende de T031.

**Punto de control**: las cuatro historias funcionan de forma independiente y conjunta.

---

## Fase 7: Pulido y aspectos transversales

**Propósito**: cubrir requisitos transversales, accesibilidad, idioma y validación final.

- [ ] T033 [P] Escribir y ejecutar en `tests/e2e/interfaz-y-errores.spec.js` las pruebas transversales de FR-004, FR-005, FR-012, FR-014, FR-019, FR-022 y FR-024: textos en español, límites de 200 y 2000 caracteres, etiquetas de completada y vencida no dependientes solo del color, estados vacíos, formato de fechas y fallo de guardado simulado sustituyendo `localStorage.setItem` por una función que lanza una excepción.
- [ ] T034 [P] Revisar y corregir la accesibilidad y el uso con teclado en `index.html`, `src/app.js` y `styles.css`: etiquetas asociadas, `role="alert"` y `aria-live`, foco tras crear/editar/cancelar, navegación por Tab y activación con Enter.
- [ ] T035 Revisar en `src/`, `index.html`, `styles.css` y la documentación de `specs/001-gestion-tareas/` el cumplimiento de los cinco principios: sin dependencias de runtime, textos y documentación en español, sin abstracciones no pedidas y persistencia inmediata; corregir lo que falte.
- [ ] T036 Ejecutar `npm test` y corregir cualquier fallo hasta que toda la suite pase con un único comando documentado.
- [ ] T037 Ejecutar el recorrido manual de `specs/001-gestion-tareas/quickstart.md` con `npm start`, incluyendo la comprobación de usabilidad de SC-005 y la verificación de persistencia tras recargar; registrar cualquier incidencia antes de dar la funcionalidad por terminada.

---

## Dependencias y orden de ejecución

### Dependencias por fase

- **Fase 1 — Configuración inicial**: sin dependencias; puede empezar de inmediato.
- **Fase 2 — Fundamentos**: depende de la Fase 1 y bloquea todas las historias.
- **Fase 3 — Historia 1 (P1)**: depende de la Fase 2; no depende de otras historias.
- **Fase 4 — Historia 2 (P2)**: depende de la Fase 2 y de la Historia 1 para la lista y la creación; puede desarrollarse después de US1.
- **Fase 5 — Historia 3 (P3)**: depende de la Fase 2 y de la Historia 1 para mostrar tareas con fecha; puede avanzar en paralelo con US2 si hay capacidad.
- **Fase 6 — Historia 4 (P4)**: depende de la Fase 2, de la Historia 1 para la lista y de la Historia 2 para los estados completados; puede avanzar en paralelo con US3 si hay capacidad.
- **Fase 7 — Pulido**: depende de que todas las historias elegidas estén completas.

### Dependencias dentro de cada historia

- Las pruebas de cada historia se escriben primero y deben fallar antes de implementar.
- La entidad y la persistencia (Fase 2) preceden a cualquier flujo de interfaz.
- Los comparadores y predicados de `src/modelo.js` preceden a su uso en `src/app.js`.
- La creación precede al renderizado de la lista; el renderizado precede a completar, editar y eliminar.
- Cada historia debe terminar con su punto de control validado antes de pasar a la siguiente prioridad.

## Oportunidades de paralelización

- **Fase 1**: T002, T003, T004, T005 y T006 pueden ejecutarse en paralelo después de T001.
- **Fase 2**: T007 y T008 pueden ejecutarse en paralelo; T009 depende de ambos.
- **US1**: T011, T012 y T013 pueden ejecutarse en paralelo después de T010; T014, T015 y T016 son secuenciales porque comparten `src/app.js`.
- **US2**: T018 y T019 pueden ejecutarse en paralelo después de T017; T020, T021 y T022 son secuenciales porque comparten `src/app.js`.
- **US3**: T024 y T026 pueden ejecutarse en paralelo después de T023; T025 depende de T024 y T018.
- **US4**: T028, T029 y T030 pueden ejecutarse en paralelo después de T027; T031 y T032 son secuenciales porque comparten `src/app.js`.
- **Pulido**: T033 y T034 pueden ejecutarse en paralelo; T035, T036 y T037 son secuenciales.

## Ejemplo de paralelización: Historia 1

```text
T011 [US1] Añadir el formulario de creación en index.html
T012 [US1] Añadir estilos de formulario, lista y estado vacío en styles.css
T013 [US1] Implementar formato de fecha y comparador inicial en src/modelo.js
```

Después, en secuencia: T014 → T015 → T016 sobre `src/app.js`.

## Ejemplo de paralelización: Historia 4

```text
T028 [US4] Añadir filtros y selector de orden en index.html
T029 [US4] Añadir estilos de filtros y orden en styles.css
T030 [US4] Implementar predicados y comparadores en src/modelo.js
```

Después, en secuencia: T031 → T032 sobre `src/app.js`.

## Estrategia de implementación

### MVP primero (solo Historia 1)

1. Completar la Fase 1: configuración y servidor.
2. Completar la Fase 2: modelo, almacenamiento y arranque.
3. Completar la Fase 3: Historia 1.
4. **DETENERSE Y VALIDAR**: ejecutar `tests/e2e/crear-y-persistir.spec.js` y comprobar que una
   tarea creada con solo título sigue ahí tras recargar.
5. Demostrar el MVP si está listo.

### Entrega incremental

1. Configuración + Fundamentos → base lista.
2. Historia 1 → validar → demostrar (MVP).
3. Historia 2 → validar → demostrar.
4. Historia 3 → validar → demostrar.
5. Historia 4 → validar → demostrar.
6. Pulido → ejecutar toda la suite y el recorrido manual.

### Estrategia con varias personas

1. El equipo completa juntas las Fases 1 y 2.
2. Después de los fundamentos:
   - Persona A: Historia 1 (MVP).
   - Persona B: Historia 2, una vez que exista la lista de la Historia 1.
   - Persona C: Historia 3 e Historia 4, reutilizando la lista y los datos de las historias
     anteriores.
3. Integrar y validar cada historia de forma independiente antes de avanzar.

## Notas

- `[P]` indica archivos distintos y ausencia de dependencias pendientes.
- Las etiquetas `[US1]` a `[US4]` permiten trazar cada tarea hasta su historia y sus criterios.
- Las pruebas se escriben antes de la implementación y deben fallar primero.
- Cada historia debe poder validarse de forma independiente antes de pasar a la siguiente.
- No se añaden dependencias de runtime ni herramientas de build.
- Todos los textos de interfaz, mensajes y documentación se mantienen en español.

# Investigación (Fase 0): Gestión de tareas para estudiantes

**Funcionalidad**: `001-gestion-tareas`
**Fecha**: 2026-11-06
**Estado**: Completa; no quedan `NEEDS CLARIFICATION`.

Este documento resuelve las decisiones técnicas necesarias para el plan. Todas las alternativas
se evaluaron contra el Principio I de la constitución: la solución más simple que cumpla los
requisitos, sin dependencias injustificadas.

## Decisión 1 — Aplicación web estática con JavaScript nativo

**Decisión**: Implementar una aplicación web de una sola pantalla con HTML5, CSS3 y JavaScript
ES2022 mediante módulos ES nativos. Sin framework, sin proceso de compilación y sin dependencias
en tiempo de ejecución.

**Rationale**:
- El dominio es pequeño y la interfaz es una sola pantalla; un framework no aporta valor
  proporcional a su peso y mantenimiento.
- El navegador ya ofrece formularios, validación declarativa, `localStorage`, diálogos nativos y
  módulos ES.
- No hay backend, autenticación ni sincronización, por lo que no se necesita una arquitectura
  cliente-servidor.
- La aplicación puede cargarse desde archivos estáticos y funcionar sin red después de la carga.

**Alternativas consideradas**:
- **React/Vue/Svelte**: descartados por añadir dependencias, build y conceptos que el alcance no
  necesita.
- **Aplicación móvil nativa (Android/iOS)**: descartada por atar la solución a una plataforma y
  multiplicar el esfuerzo; la especificación solo pide un dispositivo personal.
- **Aplicación de escritorio (Electron/Tauri)**: descartada por añadir un runtime y empaquetado
  sin un requisito que lo justifique.

## Decisión 2 — Persistencia con `localStorage`

**Decisión**: Guardar todas las tareas en una única clave `todo-sdd:tareas:v1` de
`localStorage`, con un sobre JSON versionado.

**Rationale**:
- Es una capacidad nativa, disponible sin configuración y suficiente para 200 tareas de texto.
- La escritura es síncrona, lo que permite confirmar el guardado antes de actualizar la interfaz y
  detectar fallos de cuota con `try/catch`.
- No requiere servidor ni permisos adicionales.
- El límite típico de 5 MB es muy superior al tamaño de 200 tareas con descripciones de hasta
  2000 caracteres.

**Alternativas consideradas**:
- **IndexedDB**: capacidad nativa más potente, pero asíncrona y con una API considerablemente más
  compleja; no se necesita para este volumen.
- **Backend/base de datos**: descartado porque la especificación exige datos solo en el
  dispositivo y no hay cuentas.
- **Archivos del sistema**: el navegador no puede escribir archivos de forma transparente sin
  intervención del usuario; rompería el guardado automático.

**Sobre el formato**: el sobre incluye `version` para poder detectar datos de otra versión y
`tareas` como arreglo. Al leer, se valida la forma; si no cumple, se informa en español y se parte
de una lista vacía. No se implementan migraciones porque no existen versiones anteriores en
alcance.

## Decisión 3 — Fecha límite como cadena `AAAA-MM-DD`

**Decisión**: Guardar la fecha límite como cadena de calendario `AAAA-MM-DD`, sin hora, y
compararla lexicográficamente con el día local actual en el mismo formato.

**Rationale**:
- Evita desfases de zona horaria que ocurren al convertir un día a `Date`/UTC.
- El formato ordena correctamente como texto y encaja con `<input type="date">`.
- La condición de vencida es una comparación simple y determinista: una fecha de hoy no es
  anterior a hoy.
- La visualización `dd/mm/aaaa` se construye a partir de los componentes de la cadena, sin
  conversiones.

**Alternativas consideradas**:
- **Timestamp con hora**: añade información que la especificación no pide y complica la
  comparación por día.
- **Objeto `Date` serializado**: introduce ambigüedad de zona horaria y problemas al cruzar
  medianoche.
- **Biblioteca de fechas**: dependencia innecesaria para una comparación de cadenas.

## Decisión 4 — Identificadores y desempate por creación

**Decisión**: Cada tarea recibe un `id` con `crypto.randomUUID()` y guarda `creadaEn` como cadena
ISO 8601. El desempate final del orden es `creadaEn` descendente; si dos tareas comparten el mismo
milisegundo, se considera primero la que se añadió después al arreglo.

**Rationale**:
- `crypto.randomUUID()` es nativo en navegadores modernos y evita colisiones sin dependencias.
- `creadaEn` ya es necesario para el criterio de desempate de la especificación.
- No hace falta un contador persistido ni una secuencia adicional.

**Alternativas consideradas**:
- **Índice del arreglo como identificador**: se rompe al eliminar o reordenar.
- **UUID manual con `Math.random`**: innecesario cuando existe una API nativa.

## Decisión 5 — Pruebas de aceptación con Playwright

**Decisión**: Usar `@playwright/test` como única dependencia de desarrollo, ejecutando la suite
sobre Chromium con `npm test`.

**Rationale**:
- Varios criterios solo se pueden verificar en un navegador real: persistencia tras recargar,
  `localStorage`, confirmación nativa al eliminar y renderizado de estados visuales.
- Playwright permite controlar el reloj para probar vencimientos sin depender de la fecha real.
- Permite sembrar 200 tareas en `localStorage` y medir el tiempo percibido de filtrado y orden.
- Es una dependencia de desarrollo: no se envía al usuario ni afecta al tamaño de la aplicación.

**Alternativas consideradas**:
- **`node:test` puro**: útil para lógica, pero no puede abrir la página, recargarla ni manipular
  el DOM y `localStorage` del navegador.
- **jsdom/Vitest**: simula el DOM, pero no reproduce recargas reales, diálogos nativos ni el
  comportamiento del navegador.
- **Pruebas manuales**: no cumplen el Principio III, que exige pruebas automatizadas.

**Consecuencia operativa**: se documenta `npx playwright install chromium` como paso previo en
`quickstart.md`.

## Decisión 6 — Servidor estático mínimo con Node nativo

**Decisión**: Incluir `scripts/servidor.mjs`, un servidor estático mínimo construido solo con
módulos nativos de Node (`http`, `fs`, `path`), para servir la aplicación en
`http://localhost:4173`.

**Rationale**:
- Los módulos ES no se cargan correctamente desde `file://` en todos los navegadores.
- Evita añadir una dependencia de servidor de desarrollo.
- Playwright puede arrancarlo automáticamente mediante `webServer` y `npm start` sirve para la
  validación manual.

**Alternativas consideradas**:
- **Abrir `index.html` con doble clic**: frágil por las restricciones de origen de los módulos ES.
- **Paquete de servidor (http-server, serve)**: dependencia evitable para una necesidad trivial.
- **Servidor de desarrollo de un framework**: no hay framework.

## Decisión 7 — Edición en línea y confirmación nativa al eliminar

**Decisión**: Editar la tarea sustituyendo su elemento por un formulario en línea. Eliminar usa
`window.confirm` con un mensaje en español que incluye el título.

**Rationale**:
- La edición en línea evita una pantalla o modal adicional y mantiene el contexto de la lista.
- `window.confirm` es nativo, accesible y suficiente para una confirmación binaria; no requiere
  componentes propios.
- Cancelar la edición simplemente vuelve a renderizar los valores originales.

**Alternativas consideradas**:
- **Modal personalizado**: más código, gestión de foco y estilos para una confirmación simple.
- **Edición en otra pantalla**: rompe la simplicidad de una sola pantalla.

## Decisión 8 — Escritura transaccional y errores de guardado

**Decisión**: Aplicar cada mutación sobre una copia, intentar `localStorage.setItem` y solo
confirmar el estado en memoria si la escritura no lanza una excepción. Si falla, mostrar el
mensaje «No se pudo guardar el cambio. Inténtalo de nuevo.» y conservar el estado anterior.

**Rationale**:
- Cumple FR-020, FR-021 y FR-022 sin mostrar como guardado algo que no lo está.
- Es la forma más simple de manejar cuota llena o almacenamiento bloqueado.
- No requiere reintentos, colas ni sincronización.

**Alternativas consideradas**:
- **Actualizar la UI primero y guardar después**: puede mostrar datos que se pierden al recargar.
- **Reintentos automáticos**: añaden complejidad y no resuelven la falta de espacio.

## Decisión 9 — Persistencia de filtro y orden: no persistir

**Decisión**: Mantener el filtro y el orden solo en memoria. Al abrir la aplicación se usan
«Todas» y «Fecha límite».

**Rationale**:
- La especificación lo establece explícitamente en los supuestos.
- Evita estado adicional en `localStorage` y pruebas extra.
- Si se crea una tarea con el filtro «Completadas», el filtro cambia a «Todas»; ese cambio tampoco
  se persiste.

**Alternativas consideradas**:
- **Guardar preferencias de vista**: fuera de alcance y añade una segunda preocupación al
  almacenamiento.

## Decisión 10 — Accesibilidad e idioma

**Decisión**: Usar HTML semántico, etiquetas asociadas a los campos, región `aria-live` para
mensajes y etiquetas textuales «Completada» y «Vencida» además del color. Toda la interfaz y la
documentación se redactan en español.

**Rationale**:
- FR-014 exige que el vencimiento no dependa solo del color.
- El Principio V exige español en toda la interfaz y la documentación.
- Las etiquetas textuales son más simples y robustas que depender de iconos o colores.

**Alternativas consideradas**:
- **Indicadores solo cromáticos**: incumplen FR-014.
- **Librería de componentes accesibles**: dependencia innecesaria para los elementos nativos
  usados.

## Resumen de dependencias

| Dependencia | Tipo | Uso | Decisión |
|-------------|------|-----|----------|
| `@playwright/test` | Desarrollo | Pruebas E2E en navegador real | Aceptada y justificada |
| Ninguna | Runtime | — | La aplicación usa solo capacidades nativas |

## Preguntas resueltas

- **¿Dónde se guardan los datos?** En `localStorage`, clave `todo-sdd:tareas:v1`.
- **¿Cómo se evita el desfase de fechas?** Guardando y comparando cadenas `AAAA-MM-DD`.
- **¿Cómo se garantiza que un cambio no se muestre como guardado si falla?** Con escritura
  transaccional antes de confirmar el estado.
- **¿Cómo se prueban recarga, persistencia y vencimiento?** Con Playwright, recarga real y reloj
  controlado.
- **¿Qué pasa con datos corruptos?** Se informa en español y se parte de una lista vacía, sin
  migraciones.
- **¿Se persisten filtro y orden?** No; se reinician a «Todas» y «Fecha límite» al abrir.

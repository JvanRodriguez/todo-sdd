# Modelo de Datos: Gestión de tareas para estudiantes

**Funcionalidad**: `001-gestion-tareas`
**Fecha**: 2026-11-06
**Fuente**: [spec.md](./spec.md) y [plan.md](./plan.md)

## Entidad: Tarea

Representa una entrega o actividad que el estudiante debe realizar.

| Campo | Tipo | Obligatorio | Valor por defecto | Reglas |
|-------|------|-------------|-------------------|--------|
| `id` | cadena | Sí | Generado al crear | Identificador único; se usa `crypto.randomUUID()`. No cambia al editar. |
| `titulo` | cadena | Sí | — | Se recortan espacios en los extremos. Longitud resultante entre 1 y 200 caracteres. Vacío o solo espacios es inválido. |
| `descripcion` | cadena | No | `""` | Hasta 2000 caracteres. Vacía se guarda como cadena vacía. |
| `fechaLimite` | cadena o `null` | No | `null` | Día del calendario en formato `AAAA-MM-DD`, sin hora. Debe ser una fecha real. Se permite una fecha pasada. |
| `prioridad` | cadena enumerada | Sí | `"media"` | Uno de `"alta"`, `"media"` o `"baja"`. |
| `completada` | booleano | Sí | `false` | `false` = pendiente; `true` = completada. |
| `creadaEn` | cadena ISO 8601 | Sí | Momento de creación | Se usa para el desempate final del orden. No cambia al editar. |

### Campos derivados (no se guardan)

| Campo | Tipo | Cálculo |
|-------|------|---------|
| `vencida` | booleano | `!completada && fechaLimite !== null && fechaLimite < hoyLocal` |
| `estado` | cadena | `completada ? "completada" : "pendiente"` |

`hoyLocal` es el día actual del dispositivo en formato `AAAA-MM-DD`. Una tarea con fecha límite
igual a hoy no está vencida. La condición se recalcula en cada render y nunca se escribe en
`localStorage`.

## Sobre de persistencia

Todas las tareas se guardan en una única entrada de `localStorage`.

- **Clave**: `todo-sdd:tareas:v1`
- **Valor**: JSON serializado con esta forma:

```json
{
  "version": 1,
  "tareas": [
    {
      "id": "8c0f5b1e-7a1d-4f36-9f47-2f7f6f2a9c11",
      "titulo": "Informe de laboratorio de Química",
      "descripcion": "Incluir resultados y conclusiones.",
      "fechaLimite": "2026-11-10",
      "prioridad": "alta",
      "completada": false,
      "creadaEn": "2026-11-06T14:32:10.123Z"
    }
  ]
}
```

- `version` permite identificar el formato. En esta funcionalidad solo existe la versión `1`.
- Al leer, se validan la forma del sobre y cada tarea. Si el JSON no se puede parsear o no cumple
  el contrato, se informa en español y se parte de una lista vacía.
- Al guardar, se serializa el arreglo completo en cada mutación.
- No se guardan el filtro, el orden, el estado de edición ni los mensajes de la interfaz.

El contrato formal está en
[contracts/persistencia.schema.json](./contracts/persistencia.schema.json).

## Reglas de validación

1. **Título requerido**: `titulo.trim()` debe tener entre 1 y 200 caracteres. Si no, la operación
   se rechaza con «El título es obligatorio.».
2. **Descripción**: máximo 2000 caracteres.
3. **Fecha límite**: si no es `null`, debe cumplir `^\d{4}-\d{2}-\d{2}$` y corresponder a una
   fecha real (por ejemplo, `2026-02-30` es inválida).
4. **Prioridad**: debe pertenecer al conjunto `{"alta", "media", "baja"}`.
5. **Estado inicial**: toda tarea nueva se crea con `completada = false`.
6. **Edición**: aplica las mismas reglas que la creación. El `id` y `creadaEn` no cambian.
7. **Normalización**: el título se guarda sin espacios en los extremos. La descripción se guarda
   tal como se introdujo (hasta el límite); una descripción vacía se guarda como `""`.
8. **Fallo de guardado**: si la escritura en `localStorage` falla, ninguna regla anterior se
   considera aplicada y la tarea no se añade ni se modifica en la lista visible.

## Transiciones de estado

### Ciclo de vida de una tarea

```text
                 crear
                   │
                   ▼
             ┌───────────┐   marcar completada   ┌─────────────┐
             │ Pendiente │ ────────────────────► │ Completada  │
             │           │ ◄──────────────────── │             │
             └───────────┘   desmarcar           └─────────────┘
                   │                                   │
                   │ editar                            │ editar
                   └──────────────► misma tarea ◄──────┘

                   eliminar (con confirmación)
                   ─────────────► desaparece de la lista
```

- **Crear**: no existe tarea → tarea pendiente. La prioridad ausente se resuelve como `"media"`;
  fecha y descripción ausentes quedan como `null` y `""`.
- **Completar**: pendiente → completada. Deja de mostrarse como vencida aunque su fecha haya
  pasado.
- **Desmarcar**: completada → pendiente. Si su fecha límite ya pasó, vuelve a mostrarse como
  vencida.
- **Editar**: la tarea conserva `id`, `creadaEn` y `completada`; solo cambian título, descripción,
  fecha límite y prioridad. Quitar la fecha límite deja la tarea sin fecha y nunca vencida.
- **Eliminar**: la tarea se quita del arreglo solo después de la confirmación. No hay papelera ni
  estado intermedio.
- **Cancelar edición**: no hay transición; se conservan los valores anteriores.

## Filtros

| Filtro | Predicado |
|--------|-----------|
| `todas` | Ninguno; incluye pendientes y completadas. |
| `pendientes` | `!completada` |
| `completadas` | `completada` |

El filtro inicial es `todas`. Si se crea una tarea con `completadas` activo, el filtro cambia a
`todas` después de guardar correctamente.

## Orden

El orden se aplica después del filtro. La comparación es estable y usa los siguientes criterios en
orden:

### Orden por fecha límite (inicial)

1. Tareas con fecha antes que tareas sin fecha.
2. Fecha límite ascendente (la más próxima primero).
3. Prioridad: `alta` < `media` < `baja`.
4. `creadaEn` descendente (la más reciente primero).
5. Si `creadaEn` coincide, la tarea añadida después al arreglo va primero.

### Orden por prioridad

1. Prioridad: `alta` < `media` < `baja`.
2. Tareas con fecha antes que tareas sin fecha.
3. Fecha límite ascendente.
4. `creadaEn` descendente (la más reciente primero).
5. Si `creadaEn` coincide, la tarea añadida después al arreglo va primero.

### Orden de prioridades

```text
alta (1)  →  media (2)  →  baja (3)
```

## Invariantes

- No puede existir una tarea con `titulo` vacío después de `trim()`.
- No puede existir una tarea con prioridad fuera del enumerado.
- `id` es único dentro del arreglo.
- `fechaLimite` es `null` o una fecha real en formato `AAAA-MM-DD`.
- `vencida` nunca se almacena; se deduce siempre de `completada`, `fechaLimite` y el día actual.
- El arreglo guardado y el estado en memoria coinciden después de una operación exitosa.
- Si una operación de guardado falla, el arreglo guardado y el estado en memoria siguen siendo los
  anteriores.
- El filtro y el orden no forman parte de los datos persistidos.

## Volumen y rendimiento

- Volumen esperado: 200 tareas o más, con títulos de hasta 200 caracteres y descripciones de hasta
  2000.
- El filtrado y el orden se realizan en memoria sobre el arreglo completo.
- La prueba de rendimiento debe verificar que, con 200 tareas, aplicar un filtro y un orden se
  percibe en menos de 1 segundo (SC-004).

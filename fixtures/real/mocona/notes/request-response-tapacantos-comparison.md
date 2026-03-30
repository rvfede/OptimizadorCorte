# Comparativa Request -> Response de Tapacantos

## Casos comparados

- `03-1f6c2f86-1602-1651951AA`
- `04-12356d43-1554-1651692AA`
- `live-1651862AA`

## Tabla comparativa

| Caso | Flags activos en piezas | Codigos en `request.tapacantos[]` | Codigos con consumo en `response.planilla.tapacantos[]` | Observacion |
| --- | --- | --- | --- | --- |
| `1651951AA` | ninguno | ninguno | ninguno; la salida devuelve `MEL`, `PVC`, `AUX`, `tr1-tr4` en cero | confirma estructura propia de salida aun sin input de canto |
| `1651692AA` | `arr3`, `aba3`, `der3`, `izq3` | `113074`, `9133923`, `9134826`, `9134827`, `13070` | solo `9134826` con `ml = 109.198`, `pegado = 109.189` | no consumen ni `9134827` ni `13070` aunque existan en request |
| `1651862AA` | `aba1` | `9119799`, `9117887`, `9122890` | solo `9119799` con `ml = 0.80012`, `pegado = 0.8` | el indice activo `1` si correlaciona con el codigo 1 del request |

## Dimensiones reportadas en output

### Caso `1651862AA`

Request:

- pieza unica `400 x 1000`
- flags activos: `aba1`

Response:

- `piezas_x_placa[0] = 400 x 1000`

Implicancia:

- la pieza reportada conserva medida nominal, no medida descontada observada

### Caso `1651692AA`

Ejemplos del primer plano de corte:

- `(8) 2480 x 630`
- `(22) 629 x 728`
- `(14) 715 x 629`
- `(2) 712 x 632`

Estas medidas coinciden con las medidas nominales presentes en el request para esas referencias.

Implicancia:

- la evidencia live versionada sigue favoreciendo la hipotesis de serializacion nominal en `piezas_x_placa`

## Lectura operativa actual

Con la evidencia disponible hoy, la lectura mas prudente es esta:

- los flags `arrN/abaN/derN/izqN` siguen funcionando como seleccion de indice `N`
- pero no todo item del array `tapacantos[]` representa consumo de borde real
- el response final elige una vista propia de codigos consumidos y posiciones `trN`
- las dimensiones serializadas de pieza no prueban descuento geometrico en output

## Proxima comprobacion recomendada

- capturar un caso donde el request active un indice distinto de `1` o `3` con response real pequeno
- contrastar si la posicion del indice sigue correlacionando con el codigo consumido

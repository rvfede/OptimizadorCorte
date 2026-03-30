# Semantica de Tapacantos

## 1. Objetivo

Separar la evidencia confirmada de las hipotesis abiertas sobre el comportamiento de `tapacantos`, sus flags por lado, el descuento de dimensiones y el calculo de `ml` y `pegado`.

Este documento busca fijar una base de trabajo prudente para implementacion y pruebas.

## 2. Fuentes usadas

- `docs/LEPTON API/API - JSON.md`
- `docs/LEPTON API/API - JSON (Generico).md`
- `docs/LEPTON API/Ejemplo_Lepton/Ejemplo_request.json`
- `docs/LEPTON API/Ejemplo_Lepton/Resultado_response.json`
- `docs/Proyecto/03-reglas-de-optimizacion.md`
- `docs/Proyecto/10-motor-optimizacion.md`
- `docs/Proyecto/11-impacto-nueva-documentacion-2026-03-29.md`
- `fixtures/real/mocona/requests/`

## 3. Lo que si esta confirmado

### 3.1 Existen flags por lado y por indice

En el contrato de entrada, cada pieza puede traer flags booleanos del tipo:

- `arrN`
- `abaN`
- `derN`
- `izqN`

Lectura funcional confirmada por documentacion y ejemplo historico:

- `arrN`: lado superior usa el tapacanto N
- `abaN`: lado inferior usa el tapacanto N
- `derN`: lado derecho usa el tapacanto N
- `izqN`: lado izquierdo usa el tapacanto N

### 3.2 El ejemplo historico valida la semantica geometrica basica

En el fixture historico de ejemplo aparece una pieza `ESTANTE` de `475 x 450` con:

- `arr1 = true`
- `aba1 = true`
- `der2 = true`
- `izq2 = true`

Y el request trae:

- tapacanto 1 con `espesor = 3`
- tapacanto 2 con `espesor = 0.5`

El output historico observado reduce la pieza a:

- `474 x 444`

Esto coincide con:

- `475 - 0.5 - 0.5 = 474`
- `450 - 3 - 3 = 444`

Conclusion:

- el ejemplo historico confirma que la interpretacion documental es coherente al menos para ese caso

### 3.3 `tapacantos[]` no representa solo cantos fisicos

En los requests reales de `mocona`, el array `tapacantos[]` incluye items de tipos distintos.

Se observaron al menos estos patrones:

- tablero
- corte
- etiquetado de piezas
- canto ABS
- pegado de canto

Conclusion operativa:

- no todo item de `tapacantos[]` debe participar en descuento de dimensiones
- no todo item de `tapacantos[]` debe sumar metros lineales de borde

### 3.4 En trafico real observado, `espesor` llega en `0`

En la evidencia real ya materializada y en la documentacion de impacto se repite este patron:

- items cuya descripcion textual menciona `0.45mm`, `1mm` o `2mm`
- llegan con `espesor = 0`

Esto debilita fuertemente la regla de descuento basada solo en `tapacantos[].espesor` cuando se trabaja con trafico real de `mocona`.

### 3.5 `desp_tapacantos` aparece estable en `0.06`

En los 10 requests productivos versionados se observa consistentemente:

- `desp_tapacantos = 0.06`
- `por_desp_tapacantos = 0`
- `refilado_tapacanto = 0`

Ademas, las pruebas live documentadas sugieren que:

- `ml - pegado = 0.00012` en un caso con dos lados activos

Esto es compatible con interpretar `0.06` como una unidad muy pequena por lado, mas cercana a milimetros que a metros.

### 3.6 `planilla.tapacantos` no es espejo del request

La salida observada no replica simplemente los codigos de entrada.

Comportamientos ya documentados:

- sin tapacantos de entrada: aparecen codigos genericos como `MEL`, `PVC`, `AUX`, `tr1` a `tr4`
- con tapacantos de entrada: aparecen codigos reales y tambien relleno estructural `trN`

Conclusion:

- la serializacion de `planilla.tapacantos` debe modelarse como una capa de salida propia

## 4. Lo que hoy no puede cerrarse

### 4.1 Formula definitiva de descuento por canto en trafico real

El ejemplo historico apoya la regla documental clasica:

```text
base_corte = base_nominal - descuento_izq - descuento_der
altura_corte = altura_nominal - descuento_arr - descuento_aba

descuento_lado = espesor_tapacanto_lado + refilado_tapacanto
```

Pero en trafico real de `mocona`:

- `espesor = 0`
- los responses live observados conservaron medidas nominales en `piezas_x_placa`

Por lo tanto no puede afirmarse aun si:

- Lepton descuenta internamente usando un catalogo por `codigo`
- no descuenta cuando `espesor = 0`
- descuenta internamente pero serializa dimensiones nominales

### 4.2 Unidad exacta de `desp_tapacantos`

La evidencia live sugiere una unidad chica por lado, compatible con milimetros convertidos a metros en el resumen.

Pero todavia no puede cerrarse si `0.06` significa exactamente:

- `0.06 mm` por lado
- otra unidad interna equivalente
- o un parametro con redondeo intermedio antes de serializar

### 4.3 Criterio exacto de `pegado` y `ml`

La lectura hoy mas razonable sigue siendo:

- `pegado`: largo neto de lados tapacanteados
- `ml`: largo neto mas desperdicios fijos y porcentuales

Pero aun faltan responses reales versionados para verificar la formula completa por tipo de item y por codigo.

## 5. Observaciones sobre los fixtures reales versionados

En la coleccion `fixtures/real/mocona/requests/`:

- los 10 requests traen `desp_tapacantos = 0.06`
- solo un caso activa explicitamente flags de lado: `04-12356d43-1554-1651692AA.json`
- ese caso usa `arr3`, `aba3`, `der3`, `izq3`
- la mayoria de los requests productivos traen `tapacantos[]` con items cargados pero sin lados activos en las piezas

Implicancia:

- el trafico real observado hasta ahora no alcanza para extrapolar una semantica universal de todos los indices `N`
- tampoco alcanza para afirmar que todo item de `tapacantos[]` se use como insumo de corte geometrico

## 6. Modelo interno recomendado por ahora

Hasta capturar mas responses reales, conviene separar internamente tres conceptos.

### 6.1 Catalogo bruto de entrada

Representa cada item tal como llega en `tapacantos[]`.

Campos relevantes:

- `codigo`
- `descri`
- `espesor`
- `color`
- `line_style`
- `esp_visual`

### 6.2 Asignacion de lados por pieza

Representa la activacion de `arrN/abaN/derN/izqN` por pieza y por indice `N`.

Esto debe quedar desacoplado del catalogo bruto, porque:

- los flags pueden faltar
- puede haber indices presentes sin semantica fisica cerrada

### 6.3 Clasificacion funcional derivada

Cada item de `tapacantos[]` deberia clasificarse tentativamente en una de estas familias:

- `board_like`
- `cut_service`
- `label_service`
- `edge_material`
- `edge_service`
- `unknown`

Regla importante:

- esta clasificacion debe ser heuristica y revisable, no contractual

## 7. Regla operativa prudente para implementacion

Hasta cerrar mas evidencia, la postura menos riesgosa es esta:

- preservar el request tal como llega
- normalizar flags faltantes a `false`
- no asumir que `espesor` real viene completo en trafico productivo
- separar dimension nominal serializada de dimension interna de corte
- desacoplar el resumen de `planilla.tapacantos` del array de entrada

Si hubiera que implementar antes de cerrar la evidencia live, la mejor estrategia es:

1. soportar el modelo documental clasico como capacidad interna
2. marcar el descuento por `espesor = 0` como regla configurable o verificable por fixture
3. mantener trazabilidad para comparar output propio vs output observado

## 8. Hipotesis de trabajo recomendadas

### H1. `espesor = 0` no implica necesariamente ausencia de espesor fisico

Estado:

- fuerte

Evidencia:

- descripciones textuales con `0.45mm`, `1mm` y `2mm`
- mismo item serializado con `espesor = 0`

### H2. `desp_tapacantos` actua por lado en una unidad menor a metro

Estado:

- media-alta

Evidencia:

- diferencia live `ml - pegado = 0.00012`
- valor estable `0.06`
- compatibilidad numerica con dos lados activos

### H3. La salida de tapacantos es una vista estructurada propia

Estado:

- fuerte

Evidencia:

- presencia de codigos genericos `MEL`, `PVC`, `AUX`, `trN`
- comportamiento no espejo entre request y response

## 9. Experimentos siguientes recomendados

### Experimento 1. Versionar responses reales de 3 casos clave

Casos recomendados:

- `03-1f6c2f86-1602-1651951AA.json`
- `04-12356d43-1554-1651692AA.json`
- caso live equivalente a `1651862AA`

Objetivo:

- comparar `tapacantos[].ml`
- comparar `tapacantos[].pegado`
- verificar si la salida mantiene medidas nominales o medidas descontadas

### Experimento 2. Clasificar items reales por descripcion y codigo

Objetivo:

- construir una tabla tentativa `codigo -> tipo funcional`
- separar materiales de borde de servicios de corte o pegado

### Experimento 3. Correlacionar flags activos con resumen final

Objetivo:

- medir si un lado activo siempre impacta `pegado`
- detectar si algunos codigos del request nunca aparecen como consumo final

## 10. Implicancias para el roadmap

El proyecto ya no deberia tratar `tapacantos` como un problema menor de mapeo directo.

Afecta al menos estas areas:

- normalizacion del request
- calculo de dimensiones internas
- calculo de `ml` y `pegado`
- serializacion de `planilla.tapacantos`
- pruebas de compatibilidad funcional

## 11. Siguiente paso recomendado

El siguiente paso mas valioso ya no es crear mas reglas teoricas.

Ahora conviene:

1. capturar responses reales equivalentes a los fixtures priorizados
2. anexar a este documento una tabla `request -> response.tapacantos`
3. recien entonces fijar reglas mas duras para motor y serializer

## 12. Actualizacion 2026-03-30 por responses live versionados

Ya quedaron guardados estos responses reales:

- `fixtures/real/mocona/responses/03-1f6c2f86-1602-1651951AA.response.json`
- `fixtures/real/mocona/responses/04-12356d43-1554-1651692AA.response.json`
- `fixtures/real/mocona/responses/live-1651862AA.response.json`

Hallazgos nuevos confirmados:

- caso sin tapacantos: la salida devuelve `MEL`, `PVC`, `AUX`, `tr1` a `tr4` con valores en cero
- caso grande `1651692AA`: solo `9134826` devuelve consumo relevante con `ml = 109.198` y `pegado = 109.189`
- caso `1651862AA`: `9119799` devuelve `ml = 0.80012` y `pegado = 0.8`
- la diferencia `0.00012` refuerza fuertemente la hipotesis de `desp_tapacantos = 0.06` por lado en unidad muy chica

Implicancia:

- ya hay evidencia versionada suficiente para sostener que `planilla.tapacantos` es una vista de salida propia
- sigue abierto si el descuento geometrico interno usa catalogo por `codigo`, pero ya no esta abierto que la salida tenga logica estructural independiente

## 13. Tabla comparativa request -> response

Ya existe una comparativa puntual en:

- `fixtures/real/mocona/notes/request-response-tapacantos-comparison.md`

Resumen ejecutivo de esa comparativa:

- `1651951AA`: sin input de cantos, la salida igual devuelve `MEL`, `PVC`, `AUX`, `tr1-tr4` en cero
- `1651692AA`: con activacion intensa de `arr3/aba3/der3/izq3`, solo `9134826` muestra consumo real
- `1651862AA`: con `aba1`, solo `9119799` muestra consumo real y devuelve `ml = 0.80012`, `pegado = 0.8`
- en `1651862AA`, la pieza reportada sigue en `400 x 1000`, igual que en request

Implicancia:

- el indice activo de lado parece seguir correlacionando con la posicion del codigo en el request
- pero la salida sigue filtrando y reestructurando fuertemente que codigos considera consumo real

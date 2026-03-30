# Capturas Live 2026-03-30

## Endpoint usado

- `http://apidev.optimizadoronline.com/optimizar2`
- header `X-User-Id: mocona`

## Requests y responses versionados

### Caso 1. Simple sin tapacantos

Request:

- `fixtures/real/mocona/requests/03-1f6c2f86-1602-1651951AA.json`

Response:

- `fixtures/real/mocona/responses/03-1f6c2f86-1602-1651951AA.response.json`

Hallazgos:

- `cant_placas = 1`
- `cant_piezas_cortadas = 12`
- `m2_utilizados = 4.758`
- `planos_corte = 1`
- `planilla_vid` no vacio, longitud `418`
- `planilla.tapacantos` devuelve `MEL`, `PVC`, `AUX`, `tr1` a `tr4` con `ml = 0` y `pegado = 0`
- se observaron las 4 URLs de artefactos: planos, labels, `.saw` y `.PTX`

### Caso 2. Grande con `arr3/aba3`

Request:

- `fixtures/real/mocona/requests/04-12356d43-1554-1651692AA.json`

Response:

- `fixtures/real/mocona/responses/04-12356d43-1554-1651692AA.response.json`

Hallazgos:

- `cant_placas = 6`
- `cant_piezas_cortadas = 78`
- `m2_utilizados = 30.195`
- `planos_corte = 5`
- `planilla_vid` no vacio, longitud `2483`
- solo el codigo `9134826` devuelve consumo relevante: `ml = 109.198`, `pegado = 109.189`
- los demas codigos observados (`113074`, `9133923`, `9134827`, `13070`, `tr3`, `tr4`) quedan en cero
- se observaron las 4 URLs de artefactos

### Caso 3. Live con `aba1`

Request:

- `fixtures/real/mocona/requests/live-1651862AA.json`

Response:

- `fixtures/real/mocona/responses/live-1651862AA.response.json`

Hallazgos:

- `cant_placas = 1`
- `cant_piezas_cortadas = 2`
- `m2_utilizados = 5.0325`
- `planos_corte = 1`
- `planilla_vid` no vacio, longitud `104`
- para `codigo = 9119799`: `ml = 0.80012`, `pegado = 0.8`
- la diferencia `0.00012` refuerza la hipotesis de que `desp_tapacantos = 0.06` actua por lado en una unidad muy chica
- se observaron las 4 URLs de artefactos

## Implicancias inmediatas

- ya no hace falta hablar de responses live solo en abstracto: ahora hay fixtures reales versionados
- `planilla_vid` puede compararse entre casos de distinta complejidad
- `planilla.tapacantos` confirma logica estructural propia de salida
- la evidencia de `ml - pegado = 0.00012` ya esta respaldada por un response guardado

## Siguiente paso recomendado

- actualizar `docs/Proyecto/12-ingenieria-inversa-planilla-vid.md` con estas longitudes y casos
- actualizar `docs/Proyecto/13-semantica-tapacantos.md` con los valores reales de `ml` y `pegado`
- crear una tabla comparativa request/response por codigo de tapacanto

# Catalogo de Casos

## Fuente base

Los 10 casos de este catalogo salen de:

- `docs/LEPTON API/Nueva Doc 29-3/OPTIMIZACIONES (1).txt`

## Casos versionados

| Caso | Archivo fixture | `pedidos.archivo` | UID marketplace | Hora | Material | Tapacantos | Cantidad total | Observacion |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 01 | `01-d0a2810a-1620-1651904AA.json` | `1651904AA` | `d0a2810a` | `16:20` | `9132307` | 2 | 8 | Variante con items de tablero y corte en `tapacantos[]`. |
| 02 | `02-778ac420-1604-1651119AB.json` | `1651119AB` | `778ac420` | `16:04` | `9122360` | 2 | 5 | Caso simple de compensado con una sola pieza repetida. |
| 03 | `03-1f6c2f86-1602-1651951AA.json` | `1651951AA` | `1f6c2f86` | `16:02` | `9124576` | 0 | 12 | Caso sin tapacantos; recomendado para response live simple. |
| 04 | `04-12356d43-1554-1651692AA.json` | `1651692AA` | `12356d43` | `15:54` | `113074` | 5 | 78 | Caso mas grande; clave para estudiar `arr3/aba3`, `planilla_vid` y semantica de cantos. |
| 05 | `05-a766648-1551-1651943AA.json` | `1651943AA` | `a766648` | `15:51` | `9119804` | 2 | 6 | Caso de fenolico con rotacion deshabilitada. |
| 06 | `06-920002c6-1544-1651003FE.json` | `1651003FE` | `920002c6` | `15:44` | `116897` | 2 | 28 | Caso mediano con muchas piezas y material enchapado. |
| 07 | `07-d0a2810a-1543-1651904AA.json` | `1651904AA` | `d0a2810a` | `15:43` | `9132307` | 0 | 7 | Segunda aparicion real del mismo `archivo`; evidencia directa de no unicidad. |
| 08 | `08-49960f45-1535-1650816AH.json` | `1650816AH` | `49960f45` | `15:35` | `117594` | 0 | 3 | Caso pequeno sin tapacantos sobre fondo MDF. |
| 09 | `09-4ece1e90-1510-1651829AA.json` | `1651829AA` | `4ece1e90` | `15:10` | `9121199` | 2 | 22 | Caso de melamina 12 mm con varias dimensiones largas. |
| 10 | `10-bbab2474-1501-1651895AA.json` | `1651895AA` | `bbab2474` | `15:01` | `9117152` | 3 | 15 | Caso mediano con 3 codigos de canto reales. |

## Hallazgos estructurales ya visibles

- `pedidos.archivo` no es unico.
- `tapacantos[]` mezcla tablero, corte, pegado y cantos fisicos.
- los requests reales no usan siempre la misma cardinalidad de flags `arrN/abaN/derN/izqN`.
- hay casos sin `tapacantos[]` y casos con arrays poblados pero `espesor = 0`.

## Siguiente captura recomendada

Cuando se materialicen responses reales, conviene empezar por estos casos:

- `03-1f6c2f86-1602-1651951AA.json`
- `04-12356d43-1554-1651692AA.json`
- request live equivalente a `1651862AA` desde `OPTIMIZACIONES.txt`

# Responses Reales Capturados

Esta carpeta contiene responses reales capturados contra:

- `http://apidev.optimizadoronline.com/optimizar2`

Header usado:

- `X-User-Id: mocona`

## Responses versionados

- `03-1f6c2f86-1602-1651951AA.response.json`
- `04-12356d43-1554-1651692AA.response.json`
- `live-1651862AA.response.json`

## Hallazgos ya confirmados por estas capturas

- `planilla_vid` se devuelve con contenido no vacio en los 3 casos
- se observaron las 4 URLs de artefactos en los 3 casos: PDF, labels, `.saw` y `.PTX`
- `planilla.tapacantos` muestra logica de salida propia, no espejo directo del request
- el caso `live-1651862AA` confirma `ml = 0.80012` y `pegado = 0.8`

## Referencia de analisis

Ver:

- `fixtures/real/mocona/notes/live-captures-2026-03-30.md`

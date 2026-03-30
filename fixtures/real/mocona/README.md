# Fixtures Reales `mocona`

Coleccion inicial de fixtures reales materializada el `2026-03-30`.

Fuente principal:

- `docs/LEPTON API/Nueva Doc 29-3/OPTIMIZACIONES (1).txt`

Fuente complementaria de pruebas live contra developer:

- `docs/LEPTON API/Nueva Doc 29-3/OPTIMIZACIONES.txt`

## Objetivo

Separar y versionar los requests reales observados del tenant `mocona` para poder:

- construir pruebas de compatibilidad sobre trafico real
- estudiar variaciones del contrato realmente usado
- capturar luego responses reales equivalentes sin depender de logs crudos

## Estructura

- `requests/`: 10 requests productivos extraidos y normalizados como JSON valido
- `responses/`: espacio reservado para versionar responses reales capturados
- `notes/catalog.md`: indice de casos y observaciones de uso

## Regla de naming

Los nombres de archivo siguen este patron:

`NN-uid-hora-archivo.json`

Motivo:

- `pedidos.archivo` no es unico en la evidencia real
- en la coleccion actual `1651904AA` aparece dos veces
- por eso no conviene usar solo `archivo` como identificador de fixture

## Estado actual

- los requests reales ya quedaron materializados
- los responses reales todavia no estan versionados como JSON en este repo
- las pruebas live documentadas hasta ahora siguen referenciadas desde `docs/Proyecto/11-impacto-nueva-documentacion-2026-03-29.md`

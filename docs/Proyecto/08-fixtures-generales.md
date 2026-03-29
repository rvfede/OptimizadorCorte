# Fixtures Generales

## 1. Objetivo del documento

Definir el conjunto inicial de fixtures generales que se usaran para desarrollar, probar y validar la nueva API en ausencia de diferencias confirmadas entre empresas cliente.

## 2. Criterio rector

Hoy no existe evidencia suficiente para afirmar que las empresas cliente usan la API de maneras funcionalmente distintas.

Por lo tanto, la estrategia correcta en esta etapa es:

- asumir un comportamiento general comun
- construir fixtures generales representativos
- evitar complejidad prematura por customizaciones no confirmadas
- incorporar variaciones reales solo cuando aparezca evidencia concreta

## 3. Alcance de los fixtures generales

Los fixtures generales deben cubrir:

- compatibilidad de contrato
- reglas funcionales principales del optimizador
- operacion sync
- operacion async
- consistencia de artefactos y estadisticas

## 4. Conjunto inicial recomendado

### Fixture 01. Request base compatible

Objetivo:

- representar el request funcional minimo compatible con el contrato actual

Debe incluir:

- `pedidos`
- material
- piezas
- tapacantos
- opciones de optimizador

Fuente inicial:

- `docs/LEPTON API/Ejemplo_Lepton/Ejemplo_request.json`

### Fixture 02. Response base compatible

Objetivo:

- representar el response funcional esperado para el request base

Debe incluir:

- `planilla`
- `planilla_vid`
- planos de corte
- sobrantes
- tapacantos
- URLs de artefactos

Fuente inicial:

- `docs/LEPTON API/Ejemplo_Lepton/Resultado_response.json`

### Fixture 03. Caso general sync

Objetivo:

- validar que el sistema puede devolver el resultado funcional completo en modo sync

Debe verificar:

- contrato de salida actual
- consistencia interna de totales
- estructura exacta esperada por el sistema integrador

### Fixture 04. Caso general async

Objetivo:

- validar que el sistema puede aceptar un job async y luego devolver el mismo resultado funcional final

Debe cubrir:

- creacion de job
- consulta de estado
- recuperacion de resultado
- igualdad funcional con el caso sync equivalente

### Fixture 05. Caso general con tapacantos

Objetivo:

- validar descuentos por espesor y calculo de consumo de cantos

Debe cubrir:

- descuento de dimensiones de pieza
- `ml` y `pegado`
- impacto de `desp_tapacantos` y `por_desp_tapacantos` cuando se agreguen variantes

### Fixture 06. Caso general con sobrantes reutilizables

Objetivo:

- validar reglas de consolidacion de sobrantes y calculo de `m2_sobrantes_utilizables`

### Fixture 07. Caso general con repeticion de patrones

Objetivo:

- validar consolidacion de `planos_corte` y uso correcto de `cant`

### Fixture 08. Caso general con artefactos

Objetivo:

- validar presencia y consistencia de `url_planos`, `url_labels`, `url_cnc` y `url_aux_cnc` segun corresponda

## 5. Organizacion sugerida en el repo

Estructura sugerida futura:

- `fixtures/general/sync/`
- `fixtures/general/async/`
- `fixtures/general/artifacts/`

Cada fixture deberia tener como minimo:

- request
- response esperado o propiedades esperadas
- notas del caso

## 6. Regla de evolucion

Mientras no exista evidencia de diferencias relevantes entre empresas cliente, cualquier nuevo fixture debe entrar primero como fixture general.

Solo deberian crearse fixtures especificos por empresa cliente si se confirma alguna de estas condiciones:

- usa una parte distinta del contrato
- requiere timeouts o comportamiento operativo distinto
- depende de artefactos o configuraciones exclusivas
- presenta restricciones reales de integracion no compartidas por el resto

## 7. Decision operativa actual

En esta etapa:

- no se modelaran variantes por empresa cliente
- no se asumiran contratos especiales por tenant
- se trabajara con fixtures generales representativos del uso comun esperado

## 8. Proximo paso sugerido

El siguiente paso natural despues de este documento es:

- materializar los primeros fixtures reales en el repo a partir del ejemplo actual
- definir luego `09-errores-y-estados.md` o equivalente para cerrar el comportamiento operativo de jobs


## 9. Actualizacion 2026-03-29: fixtures reales recomendados

A partir de la nueva evidencia y de las pruebas live contra developer, ya no conviene trabajar solo con fixtures generales abstractos.

Se recomienda agregar una segunda familia de fixtures:

- `fixtures/real/mocona/requests/`
- `fixtures/real/mocona/responses/`
- `fixtures/real/mocona/notes/`

Conjunto inicial minimo recomendado:

### Fixture real 01. Caso simple sin tapacantos

Fuente sugerida:

- request basado en `1651951AA`
- response real capturado en developer el 2026-03-29

Objetivo:

- validar estructura completa del response real
- validar `planilla_vid` no vacio
- validar URLs reales de artefactos

### Fixture real 02. Caso con tapacanto simple

Fuente sugerida:

- request basado en `1651862AA`
- response real capturado en developer el 2026-03-29

Objetivo:

- validar `tapacantos[].ml` y `pegado`
- validar comportamiento de pieza con `aba1 = true`
- validar que el response conserva medidas nominales observadas

### Fixture real 03. Caso con `arr3/aba3`

Fuente sugerida:

- request de prueba live basado en `113074`
- response real capturado en developer el 2026-03-29

Objetivo:

- validar codigos reales de tapacanto en salida
- validar `planilla_vid` con patron simple
- reforzar ingenieria inversa de consumo de canto

### Fixture real 04. Coleccion de requests productivos observados

Fuente sugerida:

- los 10 requests de `Nueva Doc 29-3`

Objetivo:

- cubrir variabilidad real del tenant `mocona`
- derivar casos sinteticos faltantes desde evidencia concreta

## 10. Proximo paso sugerido

El siguiente paso natural ya no es solo materializar fixtures desde el ejemplo historico.

Ahora conviene:

1. guardar requests y responses reales como fixtures versionados
2. crear un documento especifico para ingenieria inversa de `planilla_vid`
3. crear un documento especifico para semantica de tapacantos y unidades observadas

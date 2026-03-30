# Handoff Para Nueva Sesion

## Objetivo

Este documento sirve como punto de arranque para continuar el trabajo en una nueva sesion sin perder el contexto real del proyecto al `2026-03-30`.

## Prompt recomendado

```text
Estamos trabajando en `C:\REPOS\OptimizadorCorte`.

Quiero continuar la documentacion y diseño de una API de optimizacion de corte que debe reemplazar una API existente, manteniendo compatibilidad funcional y estructural.

Antes de proponer nada, revisa estos documentos del repo:

- `docs/Proyecto/README.md`
- `docs/Proyecto/02-contrato-api.md`
- `docs/Proyecto/03-reglas-de-optimizacion.md`
- `docs/Proyecto/06-decisiones-tecnicas.md`
- `docs/Proyecto/08-fixtures-generales.md`
- `docs/Proyecto/10-motor-optimizacion.md`
- `docs/Proyecto/12-ingenieria-inversa-planilla-vid.md`
- `docs/Proyecto/13-semantica-tapacantos.md`
- `fixtures/real/mocona/notes/live-captures-2026-03-30.md`
- `fixtures/real/mocona/notes/planilla-vid-comparison.md`
- `fixtures/real/mocona/notes/request-response-tapacantos-comparison.md`
- `docs/Proyecto/99-handoff-nueva-sesion.md`

Contexto importante:

- hoy hay 1 unico sistema integrador y 4 empresas cliente
- el repo sigue sin codigo de aplicacion; hoy es principalmente documental y de fixtures
- ya se materializaron 10 requests reales de `mocona`
- ya se capturaron 3 responses reales del endpoint developer
- `planilla_vid` es campo de output, no de input
- `pedidos.cliente` se observa como identificador del tenant o empresa cliente
- el mayor riesgo de compatibilidad fina sigue siendo `planilla_vid`, y eso solo seria bloqueo serio si el integrador lo usa activamente

Despues de leer eso, quiero que continúes con: [ACA PONES LA TAREA]
```

## Estado actual resumido

### Lo que ya esta suficientemente claro para desarrollar

- contrato base de request y response
- arquitectura objetivo `async-first`
- decision de motor guillotina propio
- actor model del negocio: empresa cliente, sistema integrador, tenant
- coleccion inicial de fixtures reales para pruebas de compatibilidad

### Lo que no esta cerrado al 100%

- serializacion completa token por token de `planilla_vid`
- semantica final de `tapacantos`
- si el motor descuenta internamente medidas pero serializa dimensiones nominales
- formula exacta de algunos campos resumen como `ml_bordes`

### Evaluacion de riesgo

- el proyecto no esta en riesgo existencial
- el riesgo principal es de paridad fina, no de viabilidad general
- el unico candidato real a bloqueo fuerte es `planilla_vid` si el integrador depende de ese campo para dibujar o interpretar planos

## Evidencia nueva ya materializada

### Fixtures reales versionados

Requests reales:

- `fixtures/real/mocona/requests/`

Responses reales:

- `fixtures/real/mocona/responses/03-1f6c2f86-1602-1651951AA.response.json`
- `fixtures/real/mocona/responses/04-12356d43-1554-1651692AA.response.json`
- `fixtures/real/mocona/responses/live-1651862AA.response.json`

Notas de analisis:

- `fixtures/real/mocona/notes/live-captures-2026-03-30.md`
- `fixtures/real/mocona/notes/planilla-vid-comparison.md`
- `fixtures/real/mocona/notes/request-response-tapacantos-comparison.md`

## Hallazgos clave vigentes

### 1. `planilla_vid`

Se confirmo con evidencia versionada que:

- siempre llega en output cuando el caso fue exitoso
- no viene vacio en los 3 responses reales capturados
- tiene cuerpo geometrico variable
- usa `-2` como separador por `plano_corte`
- usa `-3` antes de una tabla final compacta
- la tabla final usa `6` tokens por cada `plano_corte`
- la placa util aparece en decimas de milimetro

Lo que sigue abierto:

- significado exacto de los 4 valores variables del bloque final
- semantica completa del cuerpo geometrico

### 2. `tapacantos`

Se confirmo con evidencia versionada que:

- `tapacantos[]` en request no contiene solo cantos fisicos; tambien aparecen tablero, corte, etiquetado y pegado
- en trafico real de `mocona`, `espesor = 0` incluso cuando la descripcion textual indica espesores no nulos
- `planilla.tapacantos` no es espejo directo del request
- en `1651862AA`, con `aba1`, solo `9119799` consume y devuelve `ml = 0.80012`, `pegado = 0.8`
- en `1651692AA`, con activacion fuerte de `arr3/aba3/der3/izq3`, solo `9134826` devuelve consumo relevante
- los responses versionados siguen favoreciendo que `piezas_x_placa` serializa medidas nominales

### 3. Metricas y artefactos

Se confirmo que los responses reales devuelven:

- `planilla`
- `planilla_vid`
- `fracc_ultima_placa`
- `url_planos`
- `url_labels`
- `url_cnc`
- `url_aux_cnc`

## Documentos clave actualizados

- `docs/Proyecto/08-fixtures-generales.md`
- `docs/Proyecto/12-ingenieria-inversa-planilla-vid.md`
- `docs/Proyecto/13-semantica-tapacantos.md`
- `docs/Proyecto/99-handoff-nueva-sesion.md`

## Siguiente foco recomendado

Hoy hay dos caminos razonables, segun la intencion de la proxima sesion.

### Opcion A. Cerrar un poco mas la ingenieria inversa

Elegir esta opcion si el objetivo es reducir mas riesgo antes de programar.

Orden recomendado:

1. capturar mas responses reales pequenos con indices de canto distintos de `1` y `3`
2. seguir comparando tokenizacion de `planilla_vid` entre casos
3. intentar identificar la semantica de los 4 valores variables del bloque final de `6` tokens
4. anexar tablas comparativas adicionales a los docs `12` y `13`

### Opcion B. Empezar a desarrollar ya

Elegir esta opcion si el objetivo es ganar velocidad de implementacion sin esperar cierre total de la ingenieria inversa.

Orden recomendado:

1. crear skeleton `.NET 8` de la API
2. modelar request/response exactos del contrato
3. montar tests de compatibilidad sobre `fixtures/real/mocona/requests` y `fixtures/real/mocona/responses`
4. dejar `planilla_vid` y serializer de `tapacantos` como capas desacopladas y explicitamente pendientes de cierre total
5. implementar el pipeline de jobs y el parser antes del motor final

## Nota operativa

La estrategia correcta para continuar es:

- usar siempre evidencia versionada antes que memoria de sesion
- separar comportamiento observado de hipotesis de trabajo
- no tratar como bloqueante todo lo que aun no tiene semantica fina cerrada
- asumir que ya hay base suficiente para empezar a desarrollar si se acepta un enfoque por capas

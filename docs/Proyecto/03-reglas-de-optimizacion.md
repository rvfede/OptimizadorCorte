# Reglas de Optimizacion

## 1. Objetivo del documento

Documentar las reglas funcionales y operativas que debe reproducir la nueva API para comportarse de forma compatible con la implementacion actual.

Este documento separa:

- reglas confirmadas por documentacion y ejemplo
- reglas inferidas a partir del ejemplo real
- reglas pendientes de validacion adicional

## 2. Fuentes utilizadas

Fuentes actualmente disponibles en el repo:

- `docs/LEPTON API/Ejemplo_Lepton/Ejemplo_request.json`
- `docs/LEPTON API/Ejemplo_Lepton/Resultado_response.json`
- `docs/LEPTON API/API - JSON.md`
- `docs/LEPTON API/API - JSON (Generico).md`

## 3. Criterio de interpretacion

La documentacion disponible muestra variantes del contrato y del nivel de detalle. Para este proyecto, la referencia prioritaria debe ser la variante realmente usada por la integracion actual, es decir, la observada en los JSON de ejemplo del proyecto.

En consecuencia:

- el ejemplo real prevalece sobre una descripcion generica cuando haya diferencias de estructura
- la documentacion Markdown se usa para completar significado funcional, defaults y enumeraciones
- cualquier regla no confirmada debe quedar marcada como pendiente

## 4. Reglas confirmadas

### 4.1 Campos omitidos y defaults

Segun la documentacion, los campos no especificados toman valores por defecto, excepto cuando se trata de campos obligatorios.

Implicancia para la nueva API:

- el parser debe tolerar campos opcionales ausentes
- la capa de normalizacion debe completar defaults antes de optimizar
- la validacion debe distinguir entre ausencia valida y ausencia invalida

### 4.2 Dimension util de la placa

La documentacion indica explicitamente:

- medida optimizable en X = `material.base - refilado_x`
- medida optimizable en Y = `material.altura - refilado_y`

Por lo tanto, la optimizacion no debe operar sobre la placa nominal completa sino sobre la placa descontando refilados.

### 4.3 Rotacion de piezas

Reglas confirmadas:

- cada pieza tiene una bandera propia de rotacion: `rota`
- el material informa si tiene `vetas`
- si el material tiene veta, la rotacion no es libre y debe respetar la orientacion del material
- si el material no tiene veta, la pieza solo puede rotar si `rota = true`

Interpretacion operativa recomendada:

- permiso final de rotacion = condicion del material + condicion de la pieza

### 4.4 Tapacantos y medida optimizada de la pieza

La documentacion indica que `refilado_tapacanto` se combina con el espesor del canto para determinar la medida a optimizar de una pieza.

El ejemplo real confirma esta regla.

Caso confirmado en el ejemplo:

- pieza input `ESTANTE`: `475 x 450`
- lados con tapacanto: `arr1`, `aba1`, `der2`, `izq2`
- espesores observados: `tapacanto 1 = 3`, `tapacanto 2 = 0.5`
- pieza optimizada en output: `474 x 444`

La diferencia coincide con:

- base: `475 - 0.5 - 0.5 = 474`
- altura: `450 - 3 - 3 = 444`

Regla operativa derivada:

- la medida de corte de la pieza se reduce por cada lado que lleva tapacanto
- la reduccion por lado depende del espesor del tapacanto de ese lado
- ademas, si `refilado_tapacanto` es mayor que cero, ese valor debe sumarse a la reduccion por cada lado con canto

Formula operativa recomendada:

```text
base_corte = base_nominal - descuento_izq - descuento_der
altura_corte = altura_nominal - descuento_arr - descuento_aba

descuento_lado = espesor_tapacanto_lado + refilado_tapacanto
```

Nota:

- esta formula esta confirmada por documentacion y por el ejemplo para el caso `refilado_tapacanto = 0`
- queda pendiente validarla con ejemplos donde `refilado_tapacanto > 0`

### 4.5 Metros lineales de tapacantos

La documentacion indica:

- `desp_tapacantos` agrega desperdicio fijo por lado al calculo de metros lineales
- `por_desp_tapacantos` agrega desperdicio porcentual

Tambien muestra el criterio general:

```text
total_ml = suma de lados con tapacanto
```

Y para una pieza con los 4 lados tapacanteados:

```text
2 * (base + desp_tapacantos) + 2 * (altura + desp_tapacantos)
```

El ejemplo real muestra que:

- cuando `desp_tapacantos = 0`
- y `por_desp_tapacantos = 0`

entonces `ml` y `pegado` coinciden en el response.

Regla operativa recomendada:

- `pegado` representa el largo neto realmente pegado
- `ml` representa el consumo total incluyendo desperdicio fijo y/o porcentual
- si no hay desperdicio configurado, ambos valores coinciden

### 4.6 Sobrantes reutilizables

La documentacion indica que `min_base`, `min_altura` y `min_sup` determinan si un sobrante es reutilizable o desperdicio.

Regla funcional confirmada:

- el sistema distingue entre sobrantes totales y sobrantes reutilizables
- los sobrantes reutilizables se listan y consolidan en el resultado

Del ejemplo real se confirma ademas:

- `cant_sobrantes_utilizables = sum(sobrantes[].cant)`
- `m2_sobrantes_utilizables = suma del area de cada sobrante reutilizable por su cantidad`

### 4.7 Agrupacion de planos de corte

Del response real se confirma:

- `planos_corte` no representa cada placa individual sino patrones de corte agrupados
- cada patron tiene un campo `cant` que indica cuantas veces se repite ese esquema
- `cant_placas = sum(planos_corte[].cant)`

Esto implica que el motor debe tener una fase de consolidacion de esquemas equivalentes.

### 4.8 Cantidad de piezas por plano

La documentacion y el ejemplo confirman:

```text
planos_corte[].cant_piezas = sum(planos_corte[].piezas_x_placa[].cant) * planos_corte[].cant
```

### 4.9 Totales globales del resultado

Del ejemplo real se confirman las siguientes relaciones:

```text
planilla.cant_piezas_cortadas = suma de piezas solicitadas
planilla.cant_placas = suma de placas usadas
planilla.m2_utilizados = area total de placas consumidas
planilla.m2_cortados + planilla.m2_sobrantes = planilla.m2_utilizados
planilla.ptje_aprov ~= planilla.m2_cortados / planilla.m2_utilizados * 100
```

Observacion importante:

- en el output actual, `m2_utilizados` representa el area total de placa consumida por la optimizacion, no solo el area de piezas utiles

### 4.10 Estructura de salida esperada

El sistema debe devolver como minimo:

- resumen estadistico global
- placas usadas consolidadas
- patrones de corte consolidados
- piezas por placa o patron
- sobrantes por placa
- sobrantes consolidados reutilizables
- consumo de tapacantos
- URLs de artefactos
- `planilla_vid`

## 5. Reglas de configuracion del optimizador

Estas reglas estan explicitadas en la documentacion y deben ser respetadas por la capa de configuracion, aunque parte de su implementacion pueda quedar en etapas posteriores.

### 5.1 Tipo de cortes

`opciones_optimizador.tipo_cortes` define la filosofia de corte principal:

- `0`: cortes verticales
- `1`: cortes horizontales
- `2`: cortes combinados

### 5.2 Fases

`opciones_optimizador.fases` define la cantidad de fases de corte:

- `1`: 2 fases
- `2`: 3 fases
- `3`: 4 fases
- `4`: 5 fases
- `99`: N fases

### 5.3 Unificar areas

`unificar_areas = true` permite evitar ciertos cortes si eso produce un sobrante mayor y mas reutilizable.

### 5.4 Precorte

`precorte = true` habilita un corte previo cercano a la mitad de la placa antes de la secuencia principal.

### 5.5 Modo full

`modo_full = true` ejecuta varias combinaciones internas de parametros y conserva el mejor resultado en terminos de placas y esquemas.

### 5.6 Fraccionamiento de ultima placa

`op_frac` controla opciones de fraccionamiento de la ultima placa. La documentacion define una serie de flags y combinaciones posibles.

## 6. Reglas inferidas pero aun no cerradas

### 6.1 Criterio exacto de reutilizacion de sobrantes

Se sabe que intervienen `min_base`, `min_altura` y `min_sup`, pero aun no esta explicitado en los ejemplos si la condicion exacta es:

- cumplir simultaneamente los tres umbrales
- cumplir una combinacion parcial
- o aplicar una regla especial segun maquina o configuracion

Hipotesis de trabajo recomendada:

- considerar reutilizable solo si cumple los tres minimos

### 6.2 Semantica precisa de `ml_bordes`

El campo `ml_bordes` forma parte del resumen, pero no coincide directamente con la simple suma de `tapacantos[].ml` del ejemplo.

Por ahora debe tratarse como un metadato calculado por la logica actual cuya formula exacta sigue pendiente.

### 6.3 Semantica precisa de `cant_desp_sierra`

El nombre sugiere cantidad de desplazamientos, eventos de sierra o cortes elementales, pero todavia no esta cerrada la formula exacta.

### 6.4 Semantica exacta de `planilla_vid`

La documentacion indica que contiene informacion geometrica del plano y que existe documentacion separada para interpretarlo.

Mientras no este esa documentacion:

- debe preservarse como parte obligatoria del contrato de salida
- no conviene rediseñarlo ni eliminarlo

## 7. Puntos abiertos por validar con mas evidencia

- diferencias exactas entre la variante generica y la variante concreta del contrato
- significado exacto de todos los flags binarios de impresion y etiquetas que realmente usa el sistema integrador
- reglas especificas por `maquina_sel`
- alcance real de `pantografo_sel`, `shape` y `mecanizados` en la implementacion objetivo
- campos obligatorios reales cuando faltan datos en el request

## 8. Recomendacion para implementacion

La implementacion deberia organizarse en este orden:

1. normalizacion del request
2. calculo de dimensiones efectivas de placas y piezas
3. optimizacion de ubicacion y cortes
4. consolidacion de patrones equivalentes
5. calculo de sobrantes y estadisticas
6. construccion exacta del response compatible

## 9. Proximo paso sugerido

A partir de este documento, el siguiente artefacto util es:

- `05-casos-de-prueba.md`

Ese documento deberia listar casos minimos para validar:

- rotacion
- veta
- refilado de placa
- refilado por tapacanto
- sobrantes reutilizables
- repeticion de patrones de corte
- coherencia de totales del response


## 10. Actualizacion 2026-03-29 a partir de requests reales y pruebas live

### 10.1 Semantica observada de `pedidos.cliente`

La evidencia real observada contradice la interpretacion previa de `pedidos.cliente` como cliente final comercial.

En los requests reales capturados:

- `pedidos.cliente = mocona`
- el mismo valor coincide con el tenant o empresa cliente observado en auth

Hipotesis de trabajo recomendada:

- tratar `pedidos.cliente` como campo opaco de compatibilidad
- no reinterpretarlo como cliente final sin evidencia adicional

### 10.2 `m2_utilizados` parece usar area nominal de placa

En pruebas reales contra developer se observo que `m2_utilizados` coincide con:

```text
material.base * material.altura / 1.000.000
```

mas que con el area util refilada.

Hipotesis de trabajo recomendada:

- el motor podria optimizar sobre placa util refilada
- pero el response compatible deberia evaluar si debe reportar metricas usando placa nominal

### 10.3 Las dimensiones de pieza en el response no confirmaron descuento por tapacanto

En los casos live probados con tapacantos activos:

- `piezas_x_placa[].base` y `piezas_x_placa[].altura` conservaron las medidas nominales de entrada

Implicancia:

- la API actual podria descontar internamente y reportar nominal
- o podria no descontar cuando `espesor = 0`
- esta regla ya no debe tratarse como cerrada

### 10.4 Unidad probable de `desp_tapacantos`

En pruebas live se observo que la diferencia `ml - pegado` es compatible con aplicar `desp_tapacantos = 0.06` como una unidad muy pequena por lado, consistente con milimetros convertidos a metros.

Hipotesis de trabajo recomendada:

- no interpretar `desp_tapacantos` directamente como metros lineales
- validar su unidad exacta con mas fixtures reales

### 10.5 Estructura real de `planilla.tapacantos`

Las pruebas live muestran dos comportamientos:

- sin tapacantos de entrada: aparecen codigos genericos como `MEL`, `PVC`, `AUX`, `tr1` a `tr4`
- con tapacantos de entrada: aparecen codigos reales y tambien posiciones `trN`

Implicancia:

- `planilla.tapacantos` debe modelarse como salida estructurada propia del sistema actual
- no como simple reflejo del array de entrada

### 10.6 Estado de `planilla_vid`

`planilla_vid` ya no debe considerarse un campo solo contractual sin evidencia.

La evidencia live confirma:

- viene con contenido real
- cambia segun el patron de corte
- justifica una linea especifica de ingenieria inversa

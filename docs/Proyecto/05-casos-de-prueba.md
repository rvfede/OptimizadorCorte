# Casos de Prueba

## 1. Objetivo del documento

Definir un conjunto minimo de casos de prueba para validar que la nueva API conserva compatibilidad funcional y estructural con el sistema actual.

## 2. Estrategia de pruebas

Las pruebas deben cubrir dos niveles:

- compatibilidad de contrato
- compatibilidad funcional del resultado

Cada caso deberia conservar:

- request de entrada
- response esperado o propiedades esperadas del response
- observaciones sobre tolerancias permitidas

## 3. Reglas generales de validacion

### 3.1 Compatibilidad de contrato

En todos los casos se debe validar:

- presencia de nodos raiz esperados
- nombres exactos de propiedades
- tipos de datos compatibles
- presencia de `planilla_vid`
- presencia de URLs esperadas, aunque puedan venir vacias segun el caso

### 3.2 Compatibilidad funcional

En los casos que apliquen se debe validar:

- cantidad total de piezas cortadas
- cantidad total de placas
- dimensiones de piezas optimizadas
- sobrantes reutilizables
- agrupacion de patrones de corte
- consistencia interna de totales y subtotales

### 3.3 Tolerancias numericas

Hasta definir criterio final, se recomienda:

- comparacion exacta para enteros y booleanos
- tolerancia pequena para floats derivados de areas, porcentajes y longitudes
- comparacion exacta de estructura aunque cambie el orden de algunos elementos, si el sistema integrador no depende de ese orden

## 4. Casos minimos recomendados

### Caso 01. Pedido minimo sin tapacantos

Objetivo:

- validar flujo basico sin descuentos por cantos

Input sugerido:

- 1 placa
- 1 pieza rectangular
- sin veta
- sin tapacantos
- sin refilados

Validaciones esperadas:

- la pieza sale con las mismas dimensiones nominales
- `cant_piezas_cortadas` coincide con la cantidad pedida
- no hay consumo de tapacantos
- no hay descuentos por refilado

### Caso 02. Pieza con rotacion permitida

Objetivo:

- validar que una pieza puede rotarse para entrar en la placa

Input sugerido:

- material sin veta
- pieza con `rota = true`
- la pieza solo entra si se rota

Validaciones esperadas:

- la pieza es efectivamente optimizada en orientacion rotada
- el resultado sigue siendo estructuralmente compatible

### Caso 03. Pieza con rotacion prohibida

Objetivo:

- validar que `rota = false` bloquea la rotacion

Input sugerido:

- material sin veta
- pieza con `rota = false`
- dimensiones que solo entrarian si se rotara

Validaciones esperadas:

- la pieza no se rota
- si no entra, el caso debe fallar o reflejarse segun la logica esperada del sistema actual

### Caso 04. Material con veta

Objetivo:

- validar restriccion global de orientacion por veta

Input sugerido:

- `material.vetas = true`
- una o mas piezas con `rota = true`

Validaciones esperadas:

- la veta restringe la rotacion efectiva
- el patron de corte respeta la orientacion del material

### Caso 05. Refilado de placa

Objetivo:

- validar que `refilado_x` y `refilado_y` reducen el area optimizable de la placa

Input sugerido:

- placa con dimensiones conocidas
- refilados distintos de cero
- piezas que solo entren en la placa nominal pero no en la placa refilada

Validaciones esperadas:

- la optimizacion usa `base - refilado_x`
- la optimizacion usa `altura - refilado_y`
- los totales reflejan esa reduccion operativa

### Caso 06. Pieza con tapacantos y espesor real

Objetivo:

- validar el descuento de dimensiones por espesor de tapacanto

Input sugerido:

- una pieza con tapacantos en lados opuestos
- espesores conocidos y distintos de cero

Validaciones esperadas:

- la medida optimizada se reduce en los lados correspondientes
- el descuento coincide con la suma de espesores aplicados

### Caso 07. Pieza con tapacantos y refilado de tapacanto

Objetivo:

- validar la combinacion entre `espesor` y `refilado_tapacanto`

Input sugerido:

- una pieza con tapacantos activos
- `refilado_tapacanto > 0`

Validaciones esperadas:

- la medida optimizada descuenta espesor + refilado por cada lado con canto
- `pegado` y `ml` reflejan correctamente la configuracion del pedido

### Caso 08. Desperdicio fijo de tapacantos

Objetivo:

- validar `desp_tapacantos`

Input sugerido:

- una pieza con 2 o 4 lados tapacanteados
- `desp_tapacantos > 0`

Validaciones esperadas:

- `ml` es mayor que `pegado`
- la diferencia coincide con el desperdicio fijo por lado

### Caso 09. Desperdicio porcentual de tapacantos

Objetivo:

- validar `por_desp_tapacantos`

Input sugerido:

- una pieza con tapacantos
- porcentaje distinto de cero

Validaciones esperadas:

- `ml` incorpora el incremento porcentual
- `pegado` conserva el largo neto pegado

### Caso 10. Sobrante no reutilizable

Objetivo:

- validar descarte de sobrantes pequenos

Input sugerido:

- `min_base`, `min_altura`, `min_sup` altos
- recortes finales por debajo de umbral

Validaciones esperadas:

- los recortes no aparecen en `sobrantes`
- `cant_sobrantes_utilizables = 0` o coincide con lo esperado

### Caso 11. Sobrante reutilizable

Objetivo:

- validar alta de sobrantes utiles

Input sugerido:

- pieza que deje un recorte grande y reutilizable
- minimos de reutilizacion que permitan conservarlo

Validaciones esperadas:

- el recorte aparece en `sobrantes_x_placa`
- el recorte aparece consolidado en `sobrantes`
- `m2_sobrantes_utilizables` coincide con el area consolidada

### Caso 12. Agrupacion de patrones identicos

Objetivo:

- validar consolidacion de esquemas repetidos

Input sugerido:

- piezas que produzcan varias placas con el mismo patron

Validaciones esperadas:

- existe un solo elemento en `planos_corte` para ese patron
- `planos_corte[].cant` refleja la cantidad de repeticiones
- `cant_placas` coincide con la suma de `cant`

### Caso 13. Multiples patrones distintos

Objetivo:

- validar coexistencia de varios esquemas

Input sugerido:

- conjunto de piezas heterogeneo que fuerce mas de un patron de corte

Validaciones esperadas:

- `planos_corte` contiene mas de un esquema
- cada esquema tiene su propio subtotal correcto

### Caso 14. Consistencia interna del response

Objetivo:

- validar formulas globales del resultado

Validaciones esperadas:

- `cant_piezas_cortadas = sum(cantidad input)`
- `cant_placas = sum(planos_corte[].cant)`
- `cant_sobrantes_utilizables = sum(sobrantes[].cant)`
- `m2_cortados + m2_sobrantes = m2_utilizados`
- `ptje_aprov ~= m2_cortados / m2_utilizados * 100`

### Caso 15. Compatibilidad estructural total

Objetivo:

- validar que el sistema integrador no requiera adaptaciones

Validaciones esperadas:

- request aceptado con la estructura actual
- response devuelto con la estructura actual
- campos clave presentes: `planilla`, `planos_corte`, `tapacantos`, `sobrantes`, `url_planos`, `url_labels`, `planilla_vid`

## 5. Casos especiales a cubrir mas adelante

- piezas con `shape` distinto de rectangular
- piezas con `mecanizados`
- maquinas especificas en `maquina_sel`
- generacion real de CNC y auxiliares
- escenarios con `modo_full`
- escenarios con `precorte`
- escenarios con `op_frac`

## 6. Orden recomendado de implementacion de pruebas

1. casos 01, 06 y 14
2. casos 02, 03 y 04
3. casos 10, 11, 12 y 13
4. casos 07, 08 y 09
5. casos especiales de maquina, CNC y mecanizados

## 7. Proximo paso sugerido

El siguiente artefacto util deberia ser:

- `06-decisiones-tecnicas.md`

Ese documento deberia responder como se implementara internamente:

- lenguaje y framework
- estrategia del motor de optimizacion
- modelo de dominio interno
- almacenamiento de artefactos
- estrategia de compatibilidad para `planilla_vid`


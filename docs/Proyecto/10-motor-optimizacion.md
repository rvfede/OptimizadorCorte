# Motor de Optimizacion

## 1. Objetivo del documento

Describir el diseno tecnico del motor de optimizacion de corte: el algoritmo elegido, la estructura interna de datos, las fases de procesamiento y las decisiones de implementacion.

## 2. Tipo de problema

El problema de optimizacion de corte es un **2D Cutting Stock Problem con restriccion de corte guillotina**.

Caracteristicas del problema:

- se parte de placas rectangulares de dimensiones fijas
- se deben ubicar piezas rectangulares dentro de cada placa
- los cortes van de borde a borde de la region disponible (corte guillotina)
- algunas piezas tienen restriccion de orientacion por veta del material
- el objetivo es minimizar la cantidad de placas usadas y maximizar el aprovechamiento

La restriccion de corte guillotina simplifica significativamente el espacio de soluciones respecto a un bin-packing general.

## 3. Decision de implementacion

Se implementa el algoritmo guillotina en casa, sin usar una libreria externa.

Motivos:

- el contrato de salida requiere informacion estructural del corte (posiciones, sobrantes, patrones) que ninguna libreria produce en el formato necesario
- los parametros de configuracion del dominio (`fases`, `modo_full`, `tipo_cortes`, etc.) no tienen equivalente en solvers genericos
- tener el arbol de cortes interno disponible es necesario para poder serializar `planilla_vid` cuando se entienda su formato
- el algoritmo guillotina esta bien documentado y es implementable con esfuerzo acotado

## 4. Algoritmo guillotina

### 4.1 Principio de funcionamiento

El algoritmo opera recursivamente sobre regiones rectangulares disponibles en la placa.

Pseudocodigo base:

```
funcion guillotina(region, piezas_pendientes):
    para cada pieza que entra en la region:
        colocar pieza en esquina superior izquierda de la region
        region_der = region a la derecha de la pieza (hasta borde)
        region_inf = region debajo de la pieza (hasta borde)
        resultado_der = guillotina(region_der, piezas_restantes)
        resultado_inf = guillotina(region_inf, piezas_restantes)
        calcular desperdicio total
        registrar combinacion como candidata
    retornar la mejor combinacion encontrada
```

Variante con corte horizontal primero o vertical primero segun `tipo_cortes`.

### 4.2 Tipo de cortes

El parametro `tipo_cortes` define como se orienta cada corte:

- `0` vertical: el primer corte siempre divide la region en izquierda/derecha
- `1` horizontal: el primer corte siempre divide la region en arriba/abajo
- `2` combinado: se prueban ambas orientaciones y se conserva el mejor resultado

### 4.3 Fases

El parametro `fases` define cuantas pasadas se realizan con distintas ordenaciones del listado de piezas.

Comportamiento esperado:

- en cada fase se intenta una ordenacion diferente (por area descendente, por altura, por base, etc.)
- se conserva el resultado de menor cantidad de placas o mayor yield
- mas fases implica mejor resultado potencial a costo de mayor tiempo de computo

Mapeo de valores segun contrato:

| valor | fases |
|-------|-------|
| 1     | 2     |
| 2     | 3     |
| 3     | 4     |
| 4     | 5     |
| 99    | N (sin limite practico) |

### 4.4 Modo full

`modo_full = true` ejecuta 8 combinaciones de parametros internos y conserva el mejor resultado.

Las 8 combinaciones surgen de variar:

- tipo de corte inicial (horizontal vs vertical)
- orden de piezas (area descendente, dimension mayor, etc.)
- orientacion preferida de las piezas en cada region

Criterio de seleccion: menor cantidad de placas; en caso de empate, mayor yield.

### 4.5 Unificar areas

`unificar_areas = true` permite evitar un corte auxiliar si eso produce un sobrante mayor y mas reutilizable.

Comportamiento esperado: si dos sobrantes adyacentes separados por un corte de sierra suman un area mayor al umbral `min_sup`, y sus dimensiones combinadas superan `min_base` y `min_altura`, se omite ese corte y se registra el sobrante unificado.

### 4.6 Precorte

`precorte = true` habilita un corte transversal cercano a la mitad de la placa antes de comenzar la secuencia principal.

Objetivo: en maquinas con limitaciones de recorrido del carro, el precorte permite procesar mitades independientes. El motor debe respetar esta configuracion y modelar el precorte como dos sub-regiones independientes.

### 4.7 Fraccionamiento de ultima placa

`op_frac` controla el tratamiento de la ultima placa cuando no se llena completamente.

Opciones definidas en el contrato: cuarto, mitad, tercios, cruz, rotacion, etc.

El fraccionamiento puede cambiar como se reparte el sobrante de la ultima placa y afecta las metricas de desperdicio.

## 5. Preprocesamiento antes de optimizar

Antes de ejecutar el algoritmo, el motor debe aplicar estas transformaciones al input:

### 5.1 Dimension util de la placa

```
base_util  = material.base   - refilado_x
altura_util = material.altura - refilado_y
```

El algoritmo opera sobre `base_util` x `altura_util`, no sobre las dimensiones nominales.

### 5.2 Dimension de corte de cada pieza

Cada pieza se reduce por el tapacanto de cada lado:

```
base_corte   = base_nominal   - descuento_izq  - descuento_der
altura_corte = altura_nominal - descuento_arr  - descuento_aba

descuento_lado = espesor_tapacanto_lado + refilado_tapacanto
```

Si un lado no tiene tapacanto, su descuento es 0.

El algoritmo opera sobre `base_corte` x `altura_corte`. El contrato de salida reporta los campos en la dimension de corte.

### 5.3 Rotacion permitida

```
puede_rotar = (material.vetas == false) AND (pieza.rota == true)
```

Si el material tiene veta, la pieza no puede rotar independientemente del valor de `rota`.

### 5.4 Desperdicio de sierra

Cada corte consume `desp_sierra` milimetros adicionales que se descuentan del espacio disponible antes de colocar la siguiente pieza.

## 6. Post-procesamiento despues de optimizar

### 6.1 Consolidacion de patrones equivalentes

Dos placas son equivalentes si contienen las mismas piezas en las mismas posiciones (o posiciones rotadas simétricamente).

El resultado debe agrupar placas equivalentes en un mismo `plano_corte` con el campo `cant` indicando la cantidad de repeticiones.

Condicion: `sum(planos_corte[].cant) == cant_placas`.

### 6.2 Clasificacion de sobrantes

Un sobrante es reutilizable si cumple simultaneamente:

- `base >= material.min_base`
- `altura >= material.min_altura`
- `base * altura >= material.min_sup`

Los sobrantes que no cumplen estas condiciones son desperdicio puro.

### 6.3 Calculo de tapacantos

Para cada tipo de tapacanto:

```
ml_neto_pieza = suma de longitudes de lados con ese tapacanto en la pieza (dimension de corte)
pegado = suma de ml_neto de todas las piezas
ml     = pegado + desperdicio_fijo + (pegado * por_desp_tapacantos / 100)

donde desperdicio_fijo = cant_piezas_con_ese_tapacanto * desp_tapacantos
```

### 6.4 Metricas globales

```
cant_placas            = sum(planos_corte[].cant)
cant_piezas_cortadas   = sum de todas las piezas del pedido
m2_utilizados          = cant_placas * base_util * altura_util / 1.000.000
m2_cortados            = suma de area de todas las piezas cortadas
m2_sobrantes           = m2_utilizados - m2_cortados
ptje_aprov             = m2_cortados / m2_utilizados * 100
```

## 7. Estructura interna de datos

### 7.1 PlateLayout

Representa el resultado de optimizar una placa.

Campos minimos:

- `root: CutNode` — raiz del arbol de cortes
- `pieces: List<PlacedPiece>` — piezas ubicadas con posicion y orientacion
- `waste_areas: List<WasteArea>` — sobrantes con posicion y dimensiones
- `yield_percent: decimal` — aprovechamiento de esta placa

### 7.2 CutNode

Representa un nodo del arbol de cortes guillotina.

Campos minimos:

- `type: enum (Piece, Waste, Cut)` — tipo de nodo
- `x, y: int` — posicion de la esquina superior izquierda
- `width, height: int` — dimensiones
- `cut_direction: enum (Vertical, Horizontal)` — si es un corte
- `left: CutNode` — sub-region izquierda o superior
- `right: CutNode` — sub-region derecha o inferior

### 7.3 OptimizationResult

Resultado completo de la optimizacion.

Campos minimos:

- `plate_layouts: List<PlateLayout>` — una entrada por placa usada
- `consolidated_patterns: List<CutPattern>` — patrones agrupados
- `global_metrics: OptimizationMetrics` — metricas globales
- `edge_banding_summary: List<EdgeBandingResult>` — consumo por tipo de tapacanto

## 8. Manejo de `planilla_vid`

El campo `planilla_vid` aparece en el response como un string serializado con informacion geometrica del plano. Su formato no esta documentado en las fuentes disponibles.

Estrategia de implementacion:

- el motor debe mantener el arbol de cortes (`CutNode`) disponible en el resultado interno
- en el response, `planilla_vid` se devuelve como string vacio hasta que se documente el formato
- cuando se obtengan mas ejemplos o se documente el campo, la serializacion se puede implementar sin tocar el algoritmo

Este campo NO debe eliminarse del response ni ignorarse en el contrato. Debe estar presente aunque vacio.

## 9. Criterio de calidad del resultado

El resultado se considera valido si:

- la estructura del response es compatible con el contrato
- `m2_utilizados + m2_sobrantes == m2_cortados` (con tolerancia de redondeo)
- `sum(planos_corte[].cant) == cant_placas`
- `yield > 0` y `yield <= 100`
- ninguna pieza supera las dimensiones de la placa util
- el resultado es razonablemente optimo (no necesariamente identico al sistema anterior)

Los resultados no necesitan ser bit-a-bit identicos a los del sistema anterior. La equivalencia se mide en terminos de resultado optimo plausible y estructura de contrato correcta.

## 10. Iteraciones y mejoras

El motor debe diseñarse para ser iterable. La secuencia de mejora esperada es:

1. algoritmo guillotina basico funcional (primera iteracion)
2. incorporar `fases` con distintas ordenaciones
3. incorporar `modo_full`
4. incorporar `unificar_areas` y `op_frac`
5. implementar `precorte`
6. serializar `planilla_vid` cuando se entienda el formato
7. optimizaciones de performance si el tiempo de computo es un problema real

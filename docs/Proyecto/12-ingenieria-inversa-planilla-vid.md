# Ingenieria Inversa de `planilla_vid`

## 1. Objetivo

Documentar la evidencia disponible y las primeras hipotesis verificables sobre el formato de `planilla_vid`.

Este documento no intenta cerrar aun la serializacion completa.

Su objetivo en esta etapa es:

- separar lo observado de lo inferido
- identificar patrones repetibles
- definir una estrategia de captura y validacion

## 2. Fuentes usadas

- `docs/LEPTON API/API - JSON.md`
- `docs/LEPTON API/API - JSON (Generico).md`
- `docs/LEPTON API/Ejemplo_Lepton/Resultado_response.json`
- `docs/Proyecto/02-contrato-api.md`
- `docs/Proyecto/11-impacto-nueva-documentacion-2026-03-29.md`
- `fixtures/real/mocona/requests/`

## 3. Evidencia documental confirmada

La documentacion disponible afirma explicitamente:

- `planilla_vid` contiene datos geometricos del plano de cortes
- existe o existio documentacion detallada adicional no presente hoy en el repo
- el JSON extendido esta pensado para dibujar el plano sin interpretar `planilla_vid`

Esto implica:

- `planilla_vid` no es un identificador arbitrario
- codifica geometria o secuencia de cortes
- conviene estudiarlo como serializacion compacta del layout y no como campo textual opaco sin estructura

## 4. Evidencia observada en el ejemplo historico

En `Resultado_response.json`:

- `planilla_vid` tiene longitud `9902`
- usa `;` como separador
- se obtienen `2108` tokens al hacer `split(';')`
- los valores observados son predominantemente enteros
- aparecen tambien valores negativos, al menos `-2` y `-3`

Prefijo observado:

```text
24442;324;0;0;0;21118;4942;14892;4900;14850;...
```

Cola observada:

```text
...;24400;21400;5;32;0;0;24400;21400;1;46;0;0;...
```

## 5. Hallazgos fuertes

### 5.1 La unidad interna parece estar en decimas de milimetro

En el ejemplo historico:

- `material.base = 2450`
- `material.altura = 2150`
- `refilado_x = 10`
- `refilado_y = 10`

La placa util resultante seria:

- `2450 - 10 = 2440 mm`
- `2150 - 10 = 2140 mm`

En `planilla_vid` aparecen repetidamente:

- `24400`
- `21400`

Hipotesis fuerte:

- muchas coordenadas y dimensiones de `planilla_vid` estan expresadas en decimas de milimetro

Esto explica por que:

- `24400 = 2440 * 10`
- `21400 = 2140 * 10`

### 5.2 La cola de `planilla_vid` correlaciona con `planos_corte[]`

En el ejemplo historico:

- `planilla.planos_corte` tiene `13` elementos

En la cola de `planilla_vid` aparecen `13` repeticiones consecutivas de bloques que empiezan con:

```text
24400;21400;...
```

Los indices de inicio de esos bloques son:

- `2030`
- `2036`
- `2042`
- `2048`
- `2054`
- `2060`
- `2066`
- `2072`
- `2078`
- `2084`
- `2090`
- `2096`
- `2102`

Cada bloque final ocupa `6` tokens y sigue este patron:

```text
base_util_x10;altura_util_x10;A;B;C;D
```

Con correlacion ya observable:

- `A` coincide con `planos_corte[].cant`
- `B` coincide con `planos_corte[].desp_sierra`

Ejemplos confirmados:

- bloque `24400;21400;5;32;0;0` coincide con plano 1: `cant = 5`, `desp_sierra = 32`
- bloque `24400;21400;1;46;0;0` coincide con plano 2: `cant = 1`, `desp_sierra = 46`
- bloque `24400;21400;2;10;0;0` coincide con plano 12: `cant = 2`, `desp_sierra = 10`

Conclusion operativa:

- la cola de `planilla_vid` no parece ser geometria libre completa
- incluye al menos un resumen serializado por patron de corte

### 5.3 La mayor parte del contenido previo a la cola parece describir geometria

Antes de esos 13 bloques finales, el string contiene:

- secuencias largas de enteros
- repeticiones de coordenadas y dimensiones
- algunos marcadores negativos

Ejemplos observados:

- `14892`
- `4942`
- `4900`
- `14850`
- `14592`
- `642`
- `14550`
- `600`
- `-2`
- `-3`

Hipotesis de trabajo:

- el cuerpo principal de `planilla_vid` serializa nodos, rectangulos o segmentos del arbol de cortes
- los enteros negativos pueden funcionar como sentinelas o delimitadores de seccion

## 6. Lo que aun no puede afirmarse

Con la evidencia actual todavia no puede cerrarse:

- el significado exacto de cada token del cuerpo principal
- si el orden responde a DFS, BFS u otra serializacion del arbol de cortes
- si las piezas se serializan con medida de corte interna o medida nominal reportada
- la semantica de los dos ultimos campos de cada bloque final de 6 tokens
- la relacion exacta entre `planilla_vid` y coordenadas de `piezas_x_placa`

## 7. Hipotesis de trabajo recomendadas

### Hipotesis H1. Unidad base

Las dimensiones y coordenadas principales estan en decimas de milimetro.

Estado:

- fuerte

Evidencia:

- coincidencia exacta entre `24400 x 21400` y la placa util del ejemplo

### Hipotesis H2. Seccion final por `plano_corte`

La cola contiene una tabla compacta con un registro por patron de corte.

Estado:

- fuerte

Evidencia:

- `13` bloques finales
- `13` elementos en `planos_corte[]`
- correlacion exacta de `cant` y `desp_sierra`

### Hipotesis H3. Cuerpo principal como serializacion del arbol de cortes

La seccion previa a la cola representa rectangulos, cortes o piezas del layout.

Estado:

- media

Evidencia:

- alta densidad de enteros geometricos repetidos
- presencia de sentinelas negativos
- documentacion que define a `planilla_vid` como datos geometricos

## 8. Experimentos siguientes recomendados

### Experimento 1. Capturar 3 responses reales

Prioridad:

- alta

Casos recomendados:

- fixture simple sin tapacantos: `03-1f6c2f86-1602-1651951AA.json`
- fixture grande con `arr3/aba3`: `04-12356d43-1554-1651692AA.json`
- caso live equivalente a `1651862AA`

Objetivo:

- comparar longitud total
- comparar cantidad de bloques finales
- validar si `base_util_x10` y `altura_util_x10` cambian segun material y refilado

### Experimento 2. Correlacionar cola con `planos_corte[]`

Prioridad:

- alta

Objetivo:

- verificar si todos los bloques finales mantienen el esquema: `base_util_x10;altura_util_x10;cant;desp_sierra;X;Y`
- identificar si `X` o `Y` correlacionan con sobrantes, `m2_util` o cantidad de subregiones

### Experimento 3. Detectar sentinelas

Prioridad:

- media

Objetivo:

- ubicar sistematicamente donde aparecen `-2` y `-3`
- verificar si separan cuerpo geometrico de tabla final

### Experimento 4. Comparar contra JSON extendido si aparece

Prioridad:

- media

Objetivo:

- si se consigue una variante de response con geometria expandida, mapear cada estructura contra tokens de `planilla_vid`

## 9. Implicancias para implementacion

Por ahora no conviene prometer serializacion completa de `planilla_vid`.

Si se avanza a skeleton o motor antes de cerrar este punto, la postura correcta es:

- modelar internamente el arbol de cortes
- preservar un serializer desacoplado
- tratar `planilla_vid` como gap funcional pendiente, no como campo prescindible

## 10. Siguiente documento recomendado

Despues de este documento, el siguiente mas valioso es:

- `docs/Proyecto/13-semantica-tapacantos.md`

Motivo:

- es la otra gran ambiguedad abierta reforzada por evidencia real
- impacta medidas, `ml`, `pegado` y posiblemente la serializacion observada

## 11. Actualizacion 2026-03-30 por responses live versionados

Ya quedaron guardados estos responses reales:

- `fixtures/real/mocona/responses/03-1f6c2f86-1602-1651951AA.response.json`
- `fixtures/real/mocona/responses/04-12356d43-1554-1651692AA.response.json`
- `fixtures/real/mocona/responses/live-1651862AA.response.json`

Nuevos hallazgos concretos:

- en los tres casos `planilla_vid` vino no vacio
- longitudes observadas: `418`, `2483` y `104`
- la longitud del string crece de forma consistente con la complejidad del caso
- esto refuerza la hipotesis de que `planilla_vid` serializa geometria o estructura de corte y no solo metadata final

Implicancia:

- el siguiente paso correcto ya no es discutir si `planilla_vid` existe realmente
- el siguiente paso correcto es comparar tokenizacion y estructura entre estos tres casos versionados

## 12. Estructura minima ya defendible

A partir de los tres responses live versionados, la estructura minima hoy defendible es esta:

1. cuerpo geometrico variable
2. un separador `-2` por cada `plano_corte`
3. un separador `-3` antes de la tabla final
4. una tabla final de `6` tokens por cada `plano_corte`

Nueva evidencia concreta:

- `1651951AA`: `1` plano, `1` token `-2`, `6` tokens finales tras `-3`
- `1651692AA`: `5` planos, `5` tokens `-2`, `30` tokens finales tras `-3`
- `1651862AA`: `1` plano, `1` token `-2`, `6` tokens finales tras `-3`

Nota importante:

- en los casos reales versionados, el cuarto valor del bloque final no coincide de forma directa con `desp_sierra`
- por lo tanto esa correlacion ya no debe asumirse como valida sin mas evidencia

Ver comparativa completa en:

- `fixtures/real/mocona/notes/planilla-vid-comparison.md`

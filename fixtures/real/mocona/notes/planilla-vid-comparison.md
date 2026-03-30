# Comparativa de `planilla_vid`

## Casos comparados

- `03-1f6c2f86-1602-1651951AA.response.json`
- `04-12356d43-1554-1651692AA.response.json`
- `live-1651862AA.response.json`

## Resumen cuantitativo

| Caso | Longitud chars | Tokens | `planos_corte` | `cant_placas` | `-2` observados | `-3` observados |
| --- | --- | --- | --- | --- | --- | --- |
| `1651951AA` | `418` | `92` | `1` | `1` | `1` | `1` |
| `1651692AA` | `2483` | `540` | `5` | `6` | `5` | `1` |
| `1651862AA` | `104` | `26` | `1` | `1` | `1` | `1` |

## Hallazgos fuertes

### 1. La cola sigue siendo una tabla compacta de 6 tokens por `plano_corte`

Casos observados:

- `1651951AA`: despues de `-3` quedan `6` tokens
- `1651692AA`: despues de `-3` quedan `30` tokens = `5 * 6`
- `1651862AA`: despues de `-3` quedan `6` tokens

Esto confirma una regla mas fuerte:

- la cola de `planilla_vid` contiene un bloque fijo de `6` tokens por cada elemento de `planos_corte[]`

### 2. `-2` parece cerrar una seccion geometrica por plano

Casos observados:

- `1651951AA`: `1` ocurrencia de `-2` y `1` plano de corte
- `1651692AA`: `5` ocurrencias de `-2` y `5` planos de corte
- `1651862AA`: `1` ocurrencia de `-2` y `1` plano de corte

Lectura recomendada:

- cada `-2` funciona como separador o cierre de una seccion geometrica de un patron
- `-3` separa el cuerpo geometrico de la tabla final compacta

### 3. La placa util sigue apareciendo en decimas de milimetro

Valores observados en la cola final:

- `1651951AA`: `25900;18200`
- `1651692AA`: `27400;18200`
- `1651862AA`: `27400;18200`

Esto coincide con:

- `(material.base - refilado_x) * 10`
- `(material.altura - refilado_y) * 10`

## Lo que sigue abierto

El bloque final parece seguir este patron:

```text
base_util_x10;altura_util_x10;A;B;C;D
```

Lo que ya puede afirmarse:

- `base_util_x10` y `altura_util_x10` estan identificados
- hay un bloque por `plano_corte`

Lo que aun no puede afirmarse con seguridad:

- significado exacto de `A`, `B`, `C` y `D`
- `B` no coincide de forma directa con `desp_sierra` en los casos reales versionados
- `C` y `D` todavia no correlacionan de forma obvia con `sobrantes_len`, `m2_util` o `cant_piezas`

## Secciones geometricas antes de `-2`

Longitudes observadas por seccion en `1651692AA`:

- `96`
- `84`
- `90`
- `114`
- `120`

Esto sugiere que:

- cada plano serializa una cantidad variable de primitivas o nodos
- la geometria no usa un bloque fijo por plano

## Implicancia operativa actual

El formato ya no debe tratarse como string opaco puro.

La estructura minima hoy defendible es:

1. cuerpo geometrico con una seccion variable por plano
2. separador `-2` por plano
3. separador `-3` antes de la tabla final
4. tabla final de `6` tokens por `plano_corte`

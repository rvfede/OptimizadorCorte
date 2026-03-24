# Contrato API

## 1. Objetivo del documento

Documentar la estructura actual del contrato de entrada y salida que la nueva API debe preservar para poder reemplazar a la implementacion existente sin romper integraciones.

## 2. Fuente actual de referencia

Este documento se basa en:

- `docs/LEPTON API/Ejemplo.json`
- `docs/LEPTON API/Resultado.json`

Cuando los PDFs sean convertidos a Markdown, este documento debe validarse y ajustarse contra esa fuente funcional adicional.

## 3. Criterio de compatibilidad

La compatibilidad requerida en esta etapa es estructural.

Esto implica:

- mantener los nombres exactos de las propiedades
- mantener la jerarquia actual del JSON
- mantener los tipos de datos esperados
- mantener la semantica observable por el sistema integrador
- evitar cambios incompatibles en request y response

## 3.1 Terminologia de actores

En este documento se distinguen tres conceptos:

- empresa cliente: empresa autenticada por API key que usa el servicio
- sistema integrador: software que llama a la API en nombre de la empresa cliente
- cliente final: valor de pedidos.cliente dentro del contrato funcional
## 4. Contrato de entrada

### 4.1 Nodo raiz

El request actual tiene como nodo raiz:

- `pedidos`: objeto

Nota: aunque semanticamente representa un pedido, el nombre del nodo actual es `pedidos` y debe preservarse mientras exista compatibilidad con el sistema actual.

### 4.2 Estructura general del request

```json
{
  "pedidos": {
    "cliente": "...",
    "archivo": "...",
    "obs1": "...",
    "obs2": "...",
    "fecha": "...",
    "hora": "...",
    "desp_sierra": 0,
    "desp_tapacantos": 0,
    "por_desp_tapacantos": 0,
    "refilado_tapacanto": 0,
    "refilado_x": 0,
    "refilado_y": 0,
    "maquina_sel": 0,
    "material": {},
    "hoja": {},
    "opciones_planilla": {},
    "datos_empresa": {},
    "opciones_optimizador": {},
    "opciones_etiquetas": {},
    "tapacantos": [],
    "piezas": []
  }
}
```

### 4.3 Campos directos de `pedidos`

| Campo | Tipo observado | Descripcion funcional |
| --- | --- | --- |
| `cliente` | string | Nombre del cliente final del pedido. |
| `archivo` | string | Identificador o nombre del archivo/pedido. |
| `obs1` | string | Observacion libre 1. |
| `obs2` | string | Observacion libre 2. |
| `fecha` | string | Fecha del pedido o corrida. |
| `hora` | string | Hora del pedido o corrida. |
| `desp_sierra` | number | Espesor o desperdicio de sierra. |
| `desp_tapacantos` | number | Parametro asociado a tapacantos. |
| `por_desp_tapacantos` | number | Parametro porcentual asociado a tapacantos. |
| `refilado_tapacanto` | number | Parametro de refilado vinculado a tapacanto. |
| `refilado_x` | number | Refilado en eje X de placa. |
| `refilado_y` | number | Refilado en eje Y de placa. |
| `maquina_sel` | number | Codigo o selector de maquina. |

### 4.4 Objeto `pedidos.material`

| Campo | Tipo observado | Descripcion funcional |
| --- | --- | --- |
| `codigo` | string | Codigo interno del material. |
| `descripcion` | string | Descripcion visible del material. |
| `vetas` | boolean | Indica si el material tiene veta. |
| `peso` | number | Peso relativo o factor del material. |
| `base` | number | Ancho de la placa madre. |
| `altura` | number | Alto de la placa madre. |
| `min_base` | number | Minimo de sobrante utilizable en base. |
| `min_altura` | number | Minimo de sobrante utilizable en altura. |
| `min_sup` | number | Minimo de superficie utilizable. |

### 4.5 Objeto `pedidos.hoja`

| Campo | Tipo observado | Descripcion funcional |
| --- | --- | --- |
| `margen_x` | number | Margen de hoja o plano. |

### 4.6 Objeto `pedidos.opciones_planilla`

| Campo | Tipo observado |
| --- | --- |
| `planillas_x_hoja` | number |
| `cant_fil` | number |
| `cant_col` | number |
| `escala` | number |
| `opciones_impresion` | number |
| `tipo_cotas` | number |
| `logo_bmp` | string |
| `logo_ancho` | number |
| `logo_alto` | number |
| `logo_ox` | number |
| `logo_oy` | number |

### 4.7 Objeto `pedidos.datos_empresa`

| Campo | Tipo observado |
| --- | --- |
| `empresa` | string |
| `direccion` | string |
| `telefono` | string |
| `email` | string |

### 4.8 Objeto `pedidos.opciones_optimizador`

| Campo | Tipo observado |
| --- | --- |
| `tipo_cortes` | number |
| `fases` | number |
| `tmax` | number |
| `unificar_areas` | boolean |
| `blocking` | boolean |
| `modo_full` | boolean |
| `mcm` | boolean |
| `apila` | number |
| `op_frac` | number |

### 4.9 Objeto `pedidos.opciones_etiquetas`

| Campo | Tipo observado |
| --- | --- |
| `opciones` | number |
| `opciones_impresion` | number |
| `papel_dx` | number |
| `papel_dy` | number |
| `margen_sup` | number |
| `margen_inf` | number |
| `margen_izq` | number |
| `sep_x` | number |
| `sep_y` | number |
| `dx` | number |
| `dy` | number |
| `cant_col` | number |
| `cant_fil` | number |
| `pdx` | number |
| `EF` | number |

### 4.10 Array `pedidos.tapacantos`

Cada elemento del array representa un tipo de tapacanto disponible para el pedido.

| Campo | Tipo observado | Descripcion funcional |
| --- | --- | --- |
| `codigo` | string | Identificador del tapacanto. |
| `color` | number | Color o codigo grafico. |
| `line_style` | number | Estilo de linea para visualizacion. |
| `esp_visual` | number | Espesor visual. |
| `espesor` | number | Espesor real del tapacanto. |

### 4.11 Array `pedidos.piezas`

Cada elemento del array representa una pieza a fabricar.

| Campo | Tipo observado | Descripcion funcional |
| --- | --- | --- |
| `cantidad` | number | Cantidad de piezas iguales. |
| `base` | number | Ancho de pieza. |
| `altura` | number | Alto de pieza. |
| `descri` | string | Descripcion principal. |
| `descri_aux` | string | Descripcion auxiliar. |
| `rota` | boolean | Indica si la pieza puede rotarse. |
| `barcode` | number | Identificador adicional o codigo. |

Ademas, cada pieza incluye flags booleanos por lado para hasta 7 tipos de tapacanto:

- `arr1` a `arr7`
- `aba1` a `aba7`
- `der1` a `der7`
- `izq1` a `izq7`

Interpretacion funcional actual:

- `arrN`: lado superior usa el tapacanto N
- `abaN`: lado inferior usa el tapacanto N
- `derN`: lado derecho usa el tapacanto N
- `izqN`: lado izquierdo usa el tapacanto N

## 5. Contrato de salida

### 5.1 Nodo raiz

El response actual tiene dos nodos raiz:

- `planilla`: objeto principal de resultado
- `planilla_vid`: string serializado complementario

### 5.2 Estructura general del response

```json
{
  "planilla": {
    "cant_placas": 0,
    "cant_piezas_cortadas": 0,
    "m2_utilizados": 0,
    "m2_cortados": 0,
    "m2_sobrantes": 0,
    "m2_sobrantes_utilizables": 0,
    "ptje_aprov": 0,
    "cant_desp_sierra": 0,
    "ml_desp_sierra": 0,
    "ml_bordes": 0,
    "placas_usadas": [],
    "planos_corte": [],
    "tapacantos": [],
    "cant_sobrantes_utilizables": 0,
    "sobrantes": [],
    "url_planos": "...",
    "url_cnc": "...",
    "url_aux_cnc": "...",
    "url_labels": "..."
  },
  "planilla_vid": "..."
}
```

### 5.3 Resumen principal en `planilla`

| Campo | Tipo observado | Descripcion funcional |
| --- | --- | --- |
| `cant_placas` | number | Cantidad total de placas utilizadas. |
| `cant_piezas_cortadas` | number | Cantidad total de piezas cortadas. |
| `m2_utilizados` | number | Metros cuadrados utilizados. |
| `m2_cortados` | number | Metros cuadrados efectivamente cortados. |
| `m2_sobrantes` | number | Metros cuadrados sobrantes. |
| `m2_sobrantes_utilizables` | number | Sobrantes reutilizables. |
| `ptje_aprov` | number | Porcentaje de aprovechamiento. |
| `cant_desp_sierra` | number | Cantidad total de cortes o eventos de sierra. |
| `ml_desp_sierra` | number | Metros lineales asociados al desperdicio de sierra. |
| `ml_bordes` | number | Metros lineales de bordes o tapacantos. |

### 5.4 Array `planilla.placas_usadas`

Agrupa tipos de placas utilizadas en la corrida.

| Campo | Tipo observado |
| --- | --- |
| `cant` | number |
| `base` | number |
| `altura` | number |
| `es_placa_entera` | boolean |
| `stock_id` | number |
| `descri` | string |

### 5.5 Array `planilla.planos_corte`

Cada elemento representa un patron o plano de corte repetible.

| Campo | Tipo observado |
| --- | --- |
| `cant` | number |
| `base` | number |
| `altura` | number |
| `es_placa_entera` | boolean |
| `stock_id` | number |
| `descri` | string |
| `ptje_desp` | number |
| `desp_sierra` | number |
| `ml_sierra` | number |
| `m2_cortados` | number |
| `m2_util` | number |
| `cant_piezas` | number |
| `piezas_x_placa` | array |
| `sobrantes_x_placa` | array |

#### 5.5.1 Array `planilla.planos_corte[].piezas_x_placa`

| Campo | Tipo observado |
| --- | --- |
| `nro_pedido` | number |
| `ref_pieza` | string |
| `cant` | number |
| `base` | number |
| `altura` | number |
| `detalle` | string |

#### 5.5.2 Array `planilla.planos_corte[].sobrantes_x_placa`

En el ejemplo observado, cada sobrante incluye:

| Campo | Tipo observado |
| --- | --- |
| `base` | number |
| `altura` | number |

### 5.6 Array `planilla.tapacantos`

Resume el consumo final por tipo de tapacanto.

| Campo | Tipo observado |
| --- | --- |
| `codigo` | string |
| `ml` | number |
| `pegado` | number |

### 5.7 Sobrantes globales

| Campo | Tipo observado |
| --- | --- |
| `cant_sobrantes_utilizables` | number |
| `sobrantes` | array |

Cada elemento de `planilla.sobrantes` incluye:

| Campo | Tipo observado |
| --- | --- |
| `cant` | number |
| `base` | number |
| `altura` | number |

### 5.8 URLs de entregables

| Campo | Tipo observado | Descripcion funcional |
| --- | --- | --- |
| `url_planos` | string | URL del PDF de planos de corte. |
| `url_cnc` | string | URL de archivo CNC, si existe. |
| `url_aux_cnc` | string | URL de archivo CNC auxiliar, si existe. |
| `url_labels` | string | URL del PDF de etiquetas. |

### 5.9 Campo `planilla_vid`

`planilla_vid` es un string largo serializado cuyo formato interno aun no esta documentado en detalle. Debe preservarse como parte del contrato de salida mientras el sistema integrador lo necesite.

## 6. Pendientes de validacion

Queda por validar contra la documentacion PDF convertida a Markdown:

- significado exacto de cada parametro numerico
- obligatoriedad real de cada campo
- valores por defecto esperados
- reglas de negocio asociadas a cada opcion del optimizador
- semantica exacta de `planilla_vid`


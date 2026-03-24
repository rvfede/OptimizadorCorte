Optimizer API - JSON
INTRODUCCIÓN
La API del optimizador de cortes expone un servicio Web que recibe un JSON Request
con los datos de un pedido (medidas del material, piezas a cortar, parámetros de la
máquina, configuración del optimizador, opciones visuales, entre otros).

El servidor optimiza dicho pedido y genera un archivo PDF con el plano de corte y los
archivos CNC propios para la máquina seleccionada, en caso de haberlos.

Estos archivos se alojan en los servidores de Amazon (por tiempo ilimitado) y, como se
detallará más adelante, su URL correspondiente forma parte del JSON Response, junto
con el resto de los datos relevantes del resultado de la optimización.

JSON REQUEST – Headers
En una primera etapa, el cliente puede hacer uso del entorno de pruebas genérico
proporcionado por Lepton, el cual viene dado por la siguiente URL:

http://apidev.optimizadoronline.com/optimizar

Para acceder, es necesario solicitar un User ID e incluirlo en el header “X-User-Id” , de
la siguiente forma:

En una etapa posterior, si el cliente contrata una URL dedicada, el User ID ya no será
necesario.

JSON REQUEST – Esquema general

Los datos del pedido deben estar contenidos en un archivo JSON. Aquellos campos que
no se especifiquen tomarán los valores por defecto, siempre y cuando no se trate de un
campo obligatorio.

A continuación, se describen todos los campos que pueden estar presentes:

{
"pedido":
{
"cliente": ,
"archivo": ,
"cnc_prefix": <prefijo de los archivos CNC del pantógrafo>,
"obs1": <observación 1>,
"obs2": <observación 2>,
"fecha": ,
"hora": ,
"desp_sierra": ,
"desp_tapacantos" : ,
"por_desp_tapacantos" : ,
"refilado_tapacanto" : ,
"refilado_x" : ,
"refilado_y" : ,
"maquina_sel": <máquina a utilizar>,
"pantografo_sel": <pantógrafo a utilizar>,
"maq_apila": <parámetro apila del optimizador de cortes>,
"planilla_vid": <datos geométricos del plano de cortes>,
"datos_empresa": {
"empresa" : ,
"direccion" : <dirección de la empresa>,
"telefono" : <teléfono de la empresa>,
"email" :
},
"material": {
"codigo" : <código del material>,
"descripcion" : <descripción del material>,
"vetas": ,
"base": ,
"altura": ,
"min_base" : <mínima base de sobrante reutilizable>,
"min_altura" : <mínima altura de sobrante reutilizable>,
"min_sup" : <mínima superficie de sobrante reutilizable>,
"peso" : ,
"espesor" :
},
"hoja": {
"margen_x" : <margen en X de la hoja de impresión>,
},
"opciones_planilla": {
"opciones_impresion": <flag de opciones de impresión>,
"x_opciones_impresion": <flag de opciones de impresión extendidas>,
"planillas_x_hoja" : ,
"cant_fil" : ,
"cant_col" : ,

"escala" : <escala de impresión de los planos>,
"tipo_cotas" : <estilo de línea de las cotas>,
"logo_bmp" : ,
"logo_ancho" : ,
"logo_alto" : ,
"logo_ox" : ,
"logo_oy" : ,
},
"opciones_optimizador": {
"tmax": <tiempo máximo de ejecución del algoritmo de optimización>,
"tipo_cortes": <filosofía de corte>,
"fases": ,
"unificar_areas": <indica si se unifican las áreas o no>,
"blocking": <indica si se aplica este criterio de optimización o no>,
"apila": <altura máxima que pueden ocupar las placas apiladas>,
"precorte": ,
"modo_full": ,
"mcm": <indica si se aplica este criterio de optimización o no>,
"op_frac": <opciones de fraccionamiento de la última placa>,
},
"opciones_etiquetas": {
"opciones": <flag de opciones de visualización>,
"opciones_impresion": <flag de opciones de impresión>,
"papel_dx": <ancho de la hoja de impresión>,
"papel_dy": <alto de la hoja de impresión>,
"margen_sup": <margen superior de la hoja de impresión>,
"margen_inf": <margen inferior de la hoja de impresión>,
"margen_izq": <margen izquierdo de la hoja de impresión>,
"sep_x": <separación en ancho entre las etiquetas>,
"sep_y": <separación en alto entre las etiquetas>,
"dx": ,
"dy": ,
"cant_fil": ,
"cant_col": ,
"pdx": <tamaño del esquema de la pieza en cada etiqueta>,
"ef":
},
"tapacantos" : [
{
"codigo": <código del tapacantos>,
"descri": <descripción del tapacantos>,
"espesor": ,
"esp_visual": ,
"color": ,
"line_style": <estilo de línea del tapacantos>
},
... 1 conjunto para cada tapacantos distinto utilizado ...
],
"piezas" : [
{
"cantidad": ,
"base": ,
"altura": ,
"descri": <descripción de la pieza>,
"descri_aux": <descripción auxiliar de la pieza>,
"rota": ,
"barcode": <código de barras de la pieza>,
"arr": <código del tapacantos de arriba de la pieza>,
"aba": <código del tapacantos de abajo de la pieza>,
"der": <código del tapacantos de la derecha de la pieza>,

"izq": <código del tapacantos de izquierda de la pieza>,
"shape": ,
"mecanizados":
{
"tipo_obj": ,
"cara": ,
"x": <posición en x del mecanizado>,
"y": <posición en y del mecanizado>,
"dx": ,
"dy": ,
"penetracion": <penetración en z del mecanizado>,
"diametro": <diámetro del mecanizado>,
}
},
... 1 conjunto para cada pieza distinta utilizada ...
]
}
}

JSON REQUEST – Detalle

A continuación, se describen en detalle los campos pertenecientes al JSON Request,
según el conjunto al que pertenecen:

Conjunto “pedido” (nivel 0)

“cliente”
Nombre del cliente que se visualizará en el encabezado del PDF.

“archivo”
Nombre del archivo que se visualizará en el encabezado del PDF.

“cnc_prefix”
Prefijo de los archivos CNC del pantógrafo.

“obs1”
Observación N°1.

“obs2”
Observación N°2.

“fecha”
Fecha del pedido en formato string. Si no se indica, el valor por defecto será UTC.
NOTA: El sistema no valida que el formato sea correcto, sino que lo muestra
exactamente como lo ingresó el usuario

“hora”
Hora del pedido en formato string. Si no se indica, el valor por defecto será UTC.
NOTA: El sistema no valida que el formato sea correcto, sino que lo muestra
exactamente como lo ingresó el usuario

“desp_tapacantos”
Desperdicio de los tapacantos (en ml). Cuando se calculan los metros lineales de
tapacantos por pieza, se le suma este valor en concepto de desperdicio.
Por ejemplo, si una pieza de base x altura tiene tapacantos en los cuatro lados, el total de
metros lineales de tapacantos será de:
2 * (base + desp_tapacantos) + 2 * (altura + desp_tapacantos).

“por_desp_tapacantos”
Ídem desp_tapacantos pero medido en porcentajes.

“desp_sierra”
Desperdicio de la sierra.

“refilado_tapacanto”
Refilado de los tapacantos. Indica cuánto se le descuenta a una pieza en el lado que
lleva canto para pegar dicho canto.
Este parámetro se combina con el espesor del canto para determinar la medida a
optimizar de una pieza.

“refilado_x”
Refilado en X de la placa. Indica cuánto se le descuenta a la base de la placa, en
concepto de refilado.
La medida a optimizar de la placa será de:
base - refilado_x

“refilado_y”
Refilado en Y de la placa. Indica cuánto se le descuenta a la altura de la placa, en
concepto de refilado.
La medida a optimizar de la placa será de:
altura - refilado_y

“maquina_sel”
Indica la máquina a utilizar.
Valores posibles:
 Máquina genérica 0
 Giben 3
 Giben N 12
 Giben G-Drive 13
 OSI XML 14
 Holzma 15
 SCM Cutplan 16

“pantografo_sel”
Indica el pantógrafo a utilizar para los mecanizados.
Valores posibles:
 Máquina no habilitada 0
 Vitap 1
 Xilogplus SCM 2
 Homag 3
 Biesse 4
 Giben 5
 Biesse CID 6
 Vertimac 7

 CRS Exata 8
 Part Nesting 9
 TPA 10
 Biesse CIX 11
NOTA: solo válido para aquellos usuarios que tengan habilitado el mecanizado CNC

“maq_apila”
Parámetro apila del optimizador de cortes.

“planilla_vid”
Datos geométricos del plano de cortes.
Para más información, solicitar documentación detallada.

Conjunto “datos_empresa” (nivel 1)

"empresa"
Nombre de la empresa.

"direccion"
Dirección de la empresa.

"telefono"
Teléfono de la empresa.

"email"
Email de la empresa.

Conjunto “material” (nivel 1)

“codigo”
Código del material de la placa.

“descripcion”
Descripción del material de la placa.

“vetas”
Indica si las piezas pueden rotar o no.
Valores posibles:
 true
 false

“base”
Base de la placa.

“altura”
Altura de la placa.

“min_base”
Mínima base de sobrante reutilizable. En conjunto con “min_altura” y “min_sup” ,
determina si un recorte sobrante es reutilizable o desperdicio.
En caso de ser reutilizable, el recorte se identificará y enumerará en el plano de corte.

“min_altura”
Mínima altura de sobrante reutilizable. En conjunto con “min_base” y “min_sup” ,
determina si un recorte sobrante es reutilizable o desperdicio.
En caso de ser reutilizable, el recorte se identificará y enumerará en el plano de corte.

“min_sup”
Mínima superficie de sobrante reutilizable. En conjunto con “min_base” y
“min_altura” , determina si un recorte sobrante es reutilizable o desperdicio.
En caso de ser reutilizable, el recorte se identificará y enumerará en el plano de corte.

“peso”
Peso de la placa.

“espesor”
Espesor de la placa.

Conjunto “hoja” (nivel 1)

“margen_x”
Margen en X de la hoja de impresión del PDF.

Conjunto “opciones_planilla” (nivel 1)
“opciones_impresion”
Opciones de visualización del PDF. Se expresa como una sumatoria de potencias de

Por ejemplo, si se quieren activar los flags OP_IMPRIMO_COTAS (1),

“x_opciones_impresion”
Opciones de visualización extendidas del PDF. Se expresa como una sumatoria de
potencias de dos, según el siguiente listado de flags:
 OPX_VER_CARPETA 1
 OPX_VER_APLICACION 2
 OPX_MSG_ALL_PAGES 4
 OPX_AJUSTAR_ESCALA_X 8

“planillas_x_hoja”
Cantidad de planos por hoja. Si el valor ingresado es superior a 6, se activa el modo
zoom. Para éste, pueden configurarse la cantidad de filas y de columnas según los
campos “cant_fil” y “cant_col” , respectivamente.

“cant_fil”
Cantidad de filas por hoja (modo zoom).

“cant_col”
Cantidad de columnas por hoja (modo zoom).

“escala”
Escala de impresión de los planos. Por defecto es 0, con lo cual se ajusta el plano al
tamaño máximo que pueda ocupar.

“tipo_cotas”
Estilo de línea de las cotas.
Valores posibles:
 COTAS_NORMALES 0
 COTAS_GRANDES 1
 COTAS_CHICAS 2

“logo_bmp”
URL del logo de la empresa.

“logo_ancho”
Ancho del logo de la empresa.

“logo_alto”
Alto del logo de la empresa.

“logo_ox”
Origen en X del logo de la empresa.

“logo_oy”
Origen en Y del logo de la empresa.

Conjunto “opciones_optimizador” (nivel 1)

“tmax”
Tiempo máximo de ejecución del algoritmo de optimización medido en pasos
algorítmicos (1 seg = 2800 pasos algorítmicos).

“tipo_cortes”
Filosofía de corte. Indica la dirección del corte principal.
Valores posibles:
 Cortes verticales 0
 Cortes horizontales 1
 Cortes combinados 2

“fases”
Cantidad de fases de corte.
Valores posibles:
 2 fases 1
 3 fases 2
 4 fases 3
 5 fases 4
 N fases 99

“unificar_areas”
En caso de poder unirse dos sobrantes separados por un corte de sierra, intenta no
realizar el corte y así obtener un sobrante más grande.
Valores posibles:
 true
 false

“blocking”
Opción interna del proceso de optimización.
Valores posibles:
 true
 false

“apila”
Altura máxima (en mm) que pueden ocupar las placas apiladas.

“precorte”
Realiza un corte cercano a la mitad de la placa previo a la secuencia de cortes.
Valores posibles:
 true
 false

“modo_full”
Se encarga de realizar la misma optimización con 8 combinaciones de parámetros
distintas, quedándose con el mejor resultado obtenido en cuanto a cantidad de placas y
de esquemas.
Valores posibles:
 true
 false

“mcm”
Opción interna del proceso de optimización.
Valores posibles:
 true
 false

“op_frac”
Opciones de fraccionamiento de la última placa.
Valores posibles:
 F_PLACA_ENTERA 0
 F_1_4_HORIZONTAL 1
 F_1_2_HORIZONTAL 2
 F_3_4_HORIZONTAL 4
 F_1_4_VERTICAL 8
 F_1_2_VERTICAL 16
 F_3_4_VERTICAL 32
 F_1_4_CRUZ 64
 F_ROTA_PLACA 128
 F_1_24 256
 F_PACK 6169
 F_PACK2 6170
 F_1_3_HORIZONTAL 512
 F_1_3_VERTICAL 1024
 F_2_3_HORIZONTAL 2048
 F_2_3_VERTICAL 4096
 F_BIT_ROTA 131072

Conjunto “opciones_etiquetas” (nivel 1)

“opciones”
Opciones de visualización de las etiquetas. Se expresa como una sumatoria de potencias
de dos, según el siguiente listado de flags:
 ET_MATERIAL 1
 ET_CLIENTE 2
 ET_PLANILLA 4
 ET_ESQUEMA 8
 ET_CODIGO_BARRA 16
 ET_BMP 32
 ET_BARCODE_PIEZA 64
 ET_NRO_ETIQUETA 128
 ET_DETALLE 512
 ET_MEDIDA 1024
 ET_AJUSTAR_ESQUEMA 2048
 ET_TAPACANTOS 4096
 ET_FECHA 8192
 ET_COTAS_PIEZA 16384
 ET_BASE_LARGA 32768
 ET_AUTONUMERICO 65536

“opciones_impresion”
Opciones de impresión de las etiquetas. Se expresa como una sumatoria de potencias de
dos, según el siguiente listado de flags:
 ET_IMPRIMO_CORTES 1
 ET_IMPRIMO_SOBRANTES 2
 ET_ESPACIO_PLACAS 4
 ET_GUARDO_FILA 8
 ET_IMPRIMO_TODAS 16
 ET_ORDEN_DE_INGRESO 32
 ET_SALTO_PAGINA 64

“papel_dx”
Ancho de la hoja de impresión.

“papel_dy”
Alto de la hoja de impresión.

“margen_sup”
Margen superior de la hoja de impresión.

“margen_inf”
Margen inferior de la hoja de impresión.

“margen_izq”
Margen izquierdo de la hoja de impresión.

“sep_x”
Separación en ancho entre las etiquetas.

“sep_y”
Separación en alto entre las etiquetas.

“dx”
Ancho de cada etiqueta.

“dy”
Alto de cada etiqueta.

“cant_fil”
Cantidad de filas.

“cant_col”
Cantidad de columnas.

“pdx”
Tamaño del esquema de la pieza en cada etiqueta.

“ef”
Escala de la fuente en cada etiqueta. Por defecto es 0.

Conjunto “tapacantos” (nivel 1)

Está estructurado como una lista de tapacantos (separados por comas), con los
siguientes campos:

“codigo”
Código del material del tapacantos.

“descri”
Descripción del material del tapacantos.

“espesor”
Espesor del material del tapacantos.

“esp_visual”
Espesor visual del tapacantos en el PDF.

“color”
Color del tapacantos en el PDF. El formato utilizado es RGB decimal.

“line_style”
Estilo de línea del tapacantos en el PDF.
Valores posibles:
 PS_NULL 0
 PS_SOLID 1
 PS_DASH 2
 PS_DASHDOT 3
 PS_DOT 4
 PS_DASHDOTDOT 5
 PS_DASH2 6
 PS_DOUBLE 7

Conjunto “piezas” (nivel 1)

Está estructurado como una lista de piezas con los siguientes campos:

“cantidad”
Cantidad de repeticiones de la pieza.

“base”
Base de la pieza.

“altura”
Altura de la pieza.

“id”
Identificador de la pieza.

“descri”
Descripción de la pieza.

“descri_aux”
Descripción auxiliar de la pieza. Se muestra en las etiquetas.

“rota”
Indica si la pieza puede rotar o no.
Valores posibles:
 true
 false

“barcode”
Código de barras de la pieza.

“arr”
Código del tapacantos de arriba de la pieza.

“aba”
Código del tapacantos de abajo de la pieza.

“der”
Código del tapacantos de derecha de la pieza.

“izq”
Código del tapacantos de izquierda de la pieza.

“shape”
Forma especial de la pieza. Solo se utiliza para piezas con forma no rectangular.
Valores posibles:
 SP_SIN_FORMA 0
 SP_CIRCULO 1
 SP_ELIPSE 2
 SP_RECT_REDONDEADO 3
 SP_TRIANGULOS_VERT 4
 SP_TRIANGULOS_HORZ 5
 SP_PUERTA 6
 SP_PUERTA2 7
 SP_SEMICIRCULO 8
 SP_TRIANGULO 9
 SP_HEXAGONO 10
 SP_TRAPECIO_VERT 11
 SP_TRAPECIO_HORZ 12
 SP_TRAPECIO 13
 SP_CORTE_V 14
 SP_CORTE_H 15
 SP_MACRO_PIEZA 1000

Conjunto “mecanizados” (nivel 2)
Forma parte del conjunto “piezas” e indica los parámetros de mecanizado para la pieza
en cuestión. Está formado por los siguientes campos:
“tipo_obj”
Tipo de objeto del mecanizado.
 OP_IMPRIMO_COTAS dos, según el siguiente listado de flags:
 OP_IMPRIMO_COLORES
 OP_AJUSTAR_ESCALA
 OP_BASE_MAS_LARGA
 OP_BASE_MAS_CORTA
 OP_COTAS_ROTADAS
 OP_LOGOTIPO
 OP_ENCA_CHICO
 OP_SIN_ENCA
 OP_TEXTURAS
 OP_VER_MEDIDA_ORIGINAL
 OP_ENCA_COLORES
 OP_MUEBLES_2_COL
 OP_MUEBLES_BMP
 OP_1FASE
 OP_2FASE
 OP_NFASE
 OP_VER_SECUENCIA
 OP_IMPRIMO_PRECIOS
 OP_ORIGEN_ABAJO
 OP_ORIGEN_DERECHA
 OP_APAISADO
 OP_VER_TIEMPOS
 OP_VER_TAPACANTOS
 OP_LISTA_TAPACANTOS
 OP_MSG_CONFORME
 OP_VER_M2
 OP_SOLAPAR_PLACAS
 OP_PIEZAS2COL
 OP_IMPRIMO_SUP
 OP_VER_MAQUINA
 OP_VER_USUARIO
 OP_STES_ACHURADOS
1 + 2 + 128 = OP_IMPRIMO_COLORES (2) y OP_ENCA_CHICO (128), el valor a ingresar es:
 OBJ_LINEA Valores posibles:
 OBJ_RECTANGULO
 OBJ_CIRCULO
 OBJ_CURVA
 OBJ_BEZIER
 OBJ_TEXTO
 OBJ_POLIGONO
 OBJ_POCKET
 OBJ_POCKET_CIRCULAR
 OBJ_AGUJERO
 OBJ_CLOCK
 OBJ_POLICURVA
 OBJ_RASTER
 OBJ_RANURA
 OBJ_PATH
 OBJ_AGUJERO_CANTO
 OBJ_AGUJERO_INF
 OBJ_RANURA_CANTO
 OBJ_AGUJERO_SUP
 OBJ_LINEA_CANTO
 OBJ_PUERTA1
 OBJ_PUERTA2
 OBJ_PUERTA3
 OBJ_PUERTA4
 OBJ_PUERTA5
 OBJ_PUERTA6
“cara”
Cara de la pieza a mecanizar.
Valores posibles:
 Cara de adelante 0
 Cara de atrás 1
 Canto izquierdo 2
 Canto derecho 3
 Canto superior 4
 Canto inferior 5

“x”
Posición en X del mecanizado.

“y”
Posición en Y del mecanizado.

“dx”
Ancho del tipo de objeto.

“dy”
Alto del tipo de objeto.

“penetracion”
Penetración en Z del mecanizado.

“diametro”
Diámetro del mecanizado.

JSON RESPONSE – Esquema general

Numero de lote = <número de lote>
planchas cortadas =
pedidos cortados =
pedidos faltantes =
listo → barra de estado

{"planilla":
{
"cant_placas": ,
"cant_piezas_cortadas": ,
"m2_utilizados": ,
"m2_cortados": ,
"m2_sobrantes": ,
"m2_sobrantes_utilizables": ,
"ptje_aprov": ,
"cant_desp_sierra": ,
"ml_desp_sierra": ,
"ml_bordes": ,
"placas_usadas": [
{
"cant": <cantidad de repeticiones de la placa en la optimización>,
"base": ,
"altura": ,
"es_placa_entera": ,
"stock_id": ,
"descri": <descripción de la placa>,
},
... 1 conjunto para cada placa distinta utilizada ...
],
"planos_corte": [
{
"cant": ,
"base": ,
"altura": ,
"es_placa_entera": ,
"stock_id": ,
"descri": <descripción de la placa>,
"ptje_desp": ,
"desp_sierra": ,
"ml_sierra": ,
"m2_cortados": ,
"m2_util": ,
"piezas_x_placa": [
{
"nro_pedido": <número de pedido>,
"ref_pieza": ,
"cant": ,
"base": ,
"altura": ,
"detalle":
},
... 1 conjunto para cada id de pieza distinto en la placa ...
],
"cant_piezas": ,

"sobrantes_x_placa": [
{
"base": ,
"altura":
},
... 1 conjunto para cada sobrante en la placa ...
]
},
... 1 conjunto para cada plano de corte distinto ...
],
"tapacantos": [
{
"codigo": <código del tapacantos>,
"ml": ,
"pegado":
},
... 1 conjunto para cada código de tapacantos distinto ...
],
"cant_sobrantes_utilizables": ,
"sobrantes": [
{
"cant": ,
"base": ,
"altura":
},
... 1 conjunto para cada medida de sobrante distinta ...
],
"url_planos": ,
"url_cnc": ,
"url_aux_cnc": ,
"url_labels": ,
},
"planilla_vid":
}

JSON RESPONSE – Detalle

El formato de JSON extendido está pensado para dibujar el plano sin tener que
interpretar el planilla_vid , y se compone por las estructuras detalladas a continuación.

Información general de la optimización

Incluye número de lote, planchas cortadas, pedidos cortados, pedidos faltantes y un
porcentaje de estado.

Resultados de la optimización

Incluye información relevante de la optimización. A destacar:
 Una serie de estadísticas de la optimización, como la cantidad de placas, m2
utilizados, porcentaje de aprovechamiento, desplazamientos y desperdicio de la
sierra, entre otros.
 Una lista de las placas utilizadas (“placas_usadas”) con información relevante de
cada una.
 Una lista de los planos de corte obtenidos (“planos_corte”) con información
relevante de cada uno.
 Una lista de los tapacantos utilizados (“tapacantos”) con información relevante
de cada uno.
 Una lista de los sobrantes obtenidos (“sobrantes”) con información relevante de
cada uno.
 Una serie de URLs con los archivos generados.

Información geométrica de la secuencia de cortes

Detallada en el campo “planilla_vid”.
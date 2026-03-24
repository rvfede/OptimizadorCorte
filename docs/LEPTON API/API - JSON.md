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

JSON REQUEST – Esquema general
Los datos del pedido deben estar contenidos en un archivo JSON. Aquellos campos que
no se especifiquen tomarán los valores por defecto, siempre y cuando no se trate de un
campo obligatorio.

A continuación, se describen todos los campos que pueden estar presentes:

{
"apikey": ,
"pedidos": {
"modificada": ,
"cliente": ,
"archivo": ,
"obs1": <observación 1>,
"obs2": <observación 2>,
"desp_sierra": ,
"desp_tapacantos" : ,
"por_desp_tapacantos" : ,
"refilado_tapacanto" : ,
"refilado_x" : ,
"refilado_y" : ,
"material":{
"codigo" : <código del material>,
"descripcion" : <descripción del material>,
"vetas": ,
"base": ,
"altura": ,
"min_base" : <mínima base de sobrante reutilizable>,
"min_altura" : <mínima altura de sobrante reutilizable>,
"min_sup" : <mínima superficie de sobrante reutilizable>
},
"datos_empresa":{
"empresa" : ,
"direccion" : <dirección de la empresa>,
"telefono" : <teléfono de la empresa>,
"email" :
},
"hoja":{
"margen_x" : <margen en X de la hoja de impresión>,
"margen_y" : <margen en Y de la hoja de impresión>,
"ancho" : <ancho de la hoja de impresión>,
"alto" : <alto de la hoja de impresión>
},
"opciones_planilla":{
"opciones_impresion": <flag de opciones de impresión>,
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
"margen_x" :
},
"opciones_optimizador": {
"maquina_sel": <máquina a utilizar>,
"maq_opciones": ,
"tmax": <tiempo máximo de ejecución del algoritmo de optimización>,
"tipo_cortes": <filosofía de corte>,
"fases": ,
"unificar_areas": <indica si se unifican las áreas o no>,
"blocking": <indica si se aplica este criterio de optimización o no>,
"apila": <altura máxima que pueden ocupar las placas apiladas>,
"precorte": ,
"modo_full": ,
"min_esquemas": ,
"mcm": <indica si se aplica este criterio de optimización o no>,
"op_frac": <opciones de fraccionamiento de la última placa>,
},
"tapacantos" : [
{
"codigo": <código del tapacantos>,
"espesor": ,
"esp_visual": ,
"color": ,
"line_style": <estilo de línea del tapacantos>
}
],
"piezas" : [
{
"cantidad": ,
"base": ,
"altura": ,
"descri": <descripción de la pieza>,
"barcode": <código de barras de la pieza>,
"tapacantos”:
{
"arriba”: <código del tapacantos>,
"abajo”: <código del tapacantos>,
"izquierda”: <código del tapacantos>,
"derecha”: <código del tapacantos>
}
]
}
}

JSON REQUEST – Detalle
A continuación, se describen en detalle los campos pertenecientes al JSON Request,
según el conjunto al que pertenecen:

{ Sin conjunto }

“apikey”
Identificador del usuario del sistema. Se puede usar una genérica para hacer pruebas
sobre el entorno de desarrollo. Una vez contratado el servicio, el desarrollador deberá
usar su propia API key.

{ Conjunto “pedidos” }

“modificada”
Indica si la planilla fue modificada manualmente o no. Aplica para interfaces que
permiten realizar cambios manuales.
Valores posibles:
 true
 false

“cliente”
Nombre del cliente que se visualizará en el encabezado del PDF.

“archivo”
Nombre del archivo que se visualizará en el encabezado del PDF.

“obs1”
Observación N°1.

“obs2”
Observación N°2.

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

{ Conjunto “datos_empresa” }

"empresa"
Nombre de la empresa.

"direccion"
Dirección de la empresa.

"telefono"
Teléfono de la empresa.

"email"
Email de la empresa.

{ Conjunto “material” }

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

{ Conjunto “hoja” }

“margen_x”
Margen en X de la hoja de impresión del PDF.

“margen_y”
Margen en Y de la hoja de impresión del PDF.

“ancho”
Ancho de la hoja de impresión del PDF.

“alto”
Alto de la hoja de impresión del PDF.

{ Conjunto “opciones_planilla” }

“opciones_impresion”
Opciones de visualización del PDF. Se expresa como una sumatoria de potencias de
dos, según el siguiente listado de flags:
 OP_IMPRIMO_COTAS 1
 OP_IMPRIMO_COLORES 2
 OP_AJUSTAR_ESCALA 4
 OP_BASE_MAS_LARGA 8
 OP_BASE_MAS_CORTA 16
 OP_COTAS_ROTADAS 32
 OP_LOGOTIPO 64
 OP_ENCA_CHICO 128
 OP_SIN_ENCA 256
 OP_TEXTURAS 512
 OP_VER_MEDIDA_ORIGINAL 1024
 OP_ENCA_COLORES 2048
 OP_MUEBLES_2_COL 4096
 OP_MUEBLES_BMP 8192
 OP_1FASE 16384
 OP_2FASE 32768
 OP_NFASE 32768
 OP_VER_SECUENCIA 65536
 OP_IMPRIMO_PRECIOS 131072
 OP_ORIGEN_ABAJO 262144
 OP_ORIGEN_DERECHA 524288

 OP_APAISADO 1048576
 OP_VER_TIEMPOS 2097152
 OP_VER_TAPACANTOS 4194304
 OP_LISTA_TAPACANTOS 8388608
 OP_MSG_CONFORME 16777216
 OP_VER_M2 33554432
 OP_SOLAPAR_PLACAS 67108864
 OP_PIEZAS2COL 134217728
 OP_IMPRIMO_SUP 268435456
 OP_VER_MAQUINA 536870912
 OP_VER_USUARIO 1073741824
 OP_STES_ACHURADOS 2147483648
Por ejemplo, si se quieren activar los flags OP_IMPRIMO_COTAS (1),
OP_IMPRIMO_COLORES (2) y OP_ENCA_CHICO (128), el valor a ingresar es:
1 + 2 + 128 = 131

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

“margen_x”
Margen izquierdo de la planilla de corte.

{ Conjunto “opciones_optimizador” }

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

“maq_opciones”
Opciones de la máquina. En general no se modifica, ya que se utilizan los valores por
defecto de la API. Pero, en caso de modificarlo, se expresa como una sumatoria de
potencias de dos, según el siguiente listado de flags:
 CNC_MAQUINA_HORIZONTAL 1
 CNC_UNIFICAR_AREAS 2
 CNC_ROTAR_PLACAS 4
 CNC_BLOCKING 8
 CNC_REORDENAR 16
 CNC_MODO_FULL 32
 CNC_PRECORTE 64
 CNC_CORTES_V 128
 CNC_CORTES_H 256

 CNC_DISCO_I 512
 CNC_DISCO_C 1024
 CNC_BANDAS_INVERSAS 8192
 CNC_ORDEN_INVERSO 16384
 CNC_CORTES_INTERNOS 32768
 CNC_CORTES_ASC_X 65536
 CNC_CORTES_ASC_Y 131072
 CNC_CORTES_LIFO 262144
 CNC_STE_ARRIBA 524288
 CNC_STE_IZQUIERDA 1048576
 CNC_MAQUINA_VIDRIO 2097152
 CNC_REFILADOS 4194304
 CNC_NO_ORDENAR 8388608
 CNC_REFILA_STOCK 16777216
 CNC_CORTES_VIRTUALES 33554432
 CNC_ESPEJAR_CORTES 67108864
 CNC_OCULTAR_MAQ 134217728
 CNC_COBRAR_REFILADOS 268435456
 CNC_CORTES_FINALES 536870912
 CNC_REFILAR_4_LADOS 1073741824
 CNC_NO_ORDENAR_SUBBANDA 2147483648
A continuación, se indican en detalle algunos de los flags:
 8192 Aplica orden inverso en bandas
 16384 Aplica orden inverso en subbandas
 32768 Orden XYZ en cortes (seccionadora manual)
 524288 Ubica sobrantes arriba
 1048576 Ubica sobrantes a izquierda
 2097152 Unifica cortes (optimiza el recorrido del cabezal de corte)
 4194304 Incluye refilados en la secuencia de cortes
 8388608 No ordena bandas
 16777216 Refila tanto placas como recortes de stock
 33554432 Genera cortes virtuales al final de la placa
 67108864 Espeja los cortes
 134217728 Oculta la máquina para que no sea visible al seleccionarla
 268435456 Cobra refilados al cliente
 536870912 Genera cortes reales al final de la placa
 1073741824 Refila los 4 lados de la pieza

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

“min_esquemas”
Indica si se prioriza minimizar la cantidad de esquemas sobre la cantidad de placas, en
casos donde la diferencia en cuanto a placas es despreciable.
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

{ Conjunto “tapacantos” }

Está estructurado como una lista de tapacantos (separados por comas), con los
siguientes campos:

“codigo”
Código del material del tapacantos.

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

{ Conjunto “piezas” }

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

“girar”
Indica si la pieza puede girar o no.
Valores posibles:
 true
 false

“barcode”
Código de barras de la pieza.

“tapacantos”
Es una estructura en sí misma, dentro del conjunto de piezas. Debe indicarse el código
de tapacantos a utilizar para cada uno de los cuatro lados de la pieza.

JSON RESPONSE – Esquema general
Numero de lote = <número de lote>
planchas cortadas =
pedidos cortados =
pedidos faltantes =
listo → barra de estado

{"resultado": {
"cant_placas": ,

"planos_corte": [
{
"id": <número de plano>,
"cant": ,
"estadisticas":
{
"ptje_desp": ,
"desp_sierra": ,
"ml_sierra": ,
"m2_cortados": ,
"m2_util":
},

"cortes": [
{
"id_corte": ,
"dimOriginal": {"base": , "altura": },
"dimCortado": {"base": , "altura": },
"posicion": {"base": <pos_x>, "altura": <pos_y>}
},
... 1 conjunto para cada pieza en la placa ...
],

"piezas_x_placa": [
{"id_corte": , "cant": , "base":
, "altura": },
... 1 conjunto para cada id de pieza distinto en la placa ...
],

"cant_piezas": <sum(piezas_x_placa[id_corte].cant) * cant_placas>,

"sobrantes_x_placa": [
{"dim": {"base": , "altura": },
"posicion":{"base": <pos_x>, "altura": <pos_y>}},
... 1 conjunto para cada sobrante en la placa ...
],

... 1 conjunto para cada plano de corte ...
],

"tapacantos": [
{"codigo": <código tap>, "ml": , "pegado": },
... 1 conjunto para cada código de tapacantos distinto ...
],

"cant_sobrantes_utilizables": ,

"sobrantes": [
{"cant": , "base": , "altura": },
... 1 conjunto para cada medida de sobrante distinta ...
],

"estadisticas": {
"cant_piezas_cortadas": ,
"m2_utilizados": ,
"m2_cortados": ,
"m2_sobrantes": ,
"m2_sobrantes_utilizables": ,
"ptje_aprov": ,
"cant_desp_sierra": ,
"ml_desp_sierra": ,
"ml_bordes":
},
"url_planos": ,
"url_xml": ,
"url_saw": ,
"url_cut": ,
"url_ac": ,
"url_ad":
},
"planilla_vid":
}

JSON RESPONSE – Detalle
El formato de JSON extendido está pensado para dibujar el plano sin tener que
interpretar el planilla_vid , y se compone por las estructuras detalladas a continuación.

Información general de la optimización

Incluye número de lote, planchas cortadas, pedidos cortados, pedidos faltantes y un
porcentaje de estado.

Resultado propiamente dicho

En primer lugar, nos indica las placas totales utilizadas para la optimización
(“cant_placas”).
Luego, muestra un conjunto con todos los planos de corte obtenidos (“planos_corte”).

Cada plano de corte está compuesto por el id de plano, cantidad de placas, una serie de
estadísticas (porcentaje de desperdicio, desperdicio de la sierra, ml de sierra, m
cortados y m2 utilizados), los cortes propiamente dichos, las piezas por placa, cantidad
de piezas y sobrantes por placa.

Cada corte (“cortes”) está compuesto por el id de corte (id > 0 se corresponde con una
pieza, id = 0 con un sobrante), la dimensión original (base y altura), la dimensión
cortada (base y altura) y la posición en el plano (base y altura en los ejes cartesianos,
tomando como (0,0) el extremo superior izquierdo).

Cada pieza por placa (“piezas_x_placa”) contiene información básica de cada pieza o
corte, como ser id, cantidad, base y altura.

A su vez, se indica la cantidad de piezas totales en dicho plano de corte (“cant_piezas”).
Este valor se obtiene sumando la cantidad de cada pieza en la placa y multiplicándolo
por la cantidad de placas en el plano de corte.

Cada sobrante por placa (“sobrantes_x_placa”) está compuesto por la dimensión del
sobrante (base y altura) y la posición en el plano (base y altura en los ejes cartesianos,
tomando como (0,0) el extremo superior izquierdo).

Datos estadísticos generales de la optimización

Una vez finalizado el conjunto de plano de cortes, se muestra un resumen de la
optimización total. A saber: cantidad de sobrantes utilizables, cantidad de cada medida
de sobrante y datos estadísticos (cantidad de piezas cortadas, m2 utilizados, m
cortados, m2 de sobrantes, m2 de sobrantes utilizables, porcentaje de aprovechamiento,
cantidad de desperdicio de la sierra, ml de desperdicio de la sierra y ml de bordes).

Por último, se indican las URL para obtener el PDF de los planos de corte y los archivos
CNC generados.
# Glosario

## Objetivo

Unificar el significado de los terminos usados en la documentacion del proyecto para evitar ambiguedades entre negocio, integracion y contrato funcional.

## Terminos principales

### API actual

Servicio de optimizacion hoy consumido por el ecosistema existente y que este proyecto busca reemplazar sin romper integraciones.

### Nueva API

Servicio de optimizacion que se desarrollara en este proyecto como reemplazo compatible de la API actual.

### Empresa cliente

Empresa vendedora de materiales que utiliza el optimizador y posee una API key propia.

En el estado actual del proyecto:

- existen 4 empresas cliente

### Sistema integrador

Software de gestion que consume la API en nombre de una empresa cliente.

En el estado actual del proyecto:

- existe 1 unico sistema integrador
- ese sistema integrador esta conectado con 4 empresas cliente

### Cliente final

Cliente comercial de la empresa cliente.

Importante:

- con la evidencia real observada hasta 2026-03-29 no puede afirmarse que este concepto corresponda al campo `pedidos.cliente`
- en los requests reales capturados, `pedidos.cliente` toma el valor `mocona`, que coincide con el tenant o empresa cliente autenticada

### `pedidos.cliente`

Campo del contrato funcional cuyo significado historico exacto sigue abierto.

Segun la evidencia real observada hasta 2026-03-29:

- funciona como identificador del tenant o empresa cliente
- no se comporta como identificador de cliente final comercial
- conviene preservarlo tal como llega por compatibilidad, sin reinterpretarlo internamente hasta tener mas evidencia

### Tenant

Unidad logica de aislamiento del sistema. En este proyecto, un tenant equivale a una empresa cliente autenticada por API key.

### API key

Credencial de acceso asociada a una empresa cliente, utilizada para autenticar requests hacia la API.

### Contrato funcional

Estructura de datos de negocio que describe el pedido de optimizacion y su resultado.

Incluye principalmente:

- request de optimizacion
- response de optimizacion
- `planilla`
- `planilla_vid`

### Contrato operativo

Capa adicional necesaria para operar la API en sync o async.

Incluye principalmente:

- creacion de jobs
- consulta de estado
- recuperacion de resultado
- callbacks
- metadata operativa

### Modo sync

Forma de consumo en la que el cliente recibe el resultado funcional completo en la misma interaccion HTTP, si el procesamiento finaliza dentro de la ventana esperada.

### Modo async

Forma de consumo en la que el pedido crea un job y el resultado se recupera posteriormente mediante polling o callback.

### Job

Unidad operativa interna que representa una ejecucion de optimizacion.

### Request funcional

Payload del pedido que describe material, piezas, tapacantos, parametros de corte y opciones de optimizacion.

### Response funcional

Resultado final del optimizador con `planilla`, `planilla_vid`, metricas, sobrantes y URLs de artefactos.

### Artefactos

Archivos derivados generados por el proceso de optimizacion.

Ejemplos:

- PDF de planos
- etiquetas
- archivos CNC
- archivos auxiliares CNC

### Uso general

Patron representativo del uso comun esperado de la API.

Ejemplos:

- uso sync compatible
- uso async compatible
- uso general de mayor volumen
- uso general con artefactos CNC

## Regla de lectura recomendada

Cuando en la documentacion aparezca el termino `cliente`, debe verificarse el contexto:

- si aparece como `pedidos.cliente`, debe interpretarse como campo opaco del contrato y no como cliente final por defecto
- si aparece en arquitectura, auth o multi-tenancy, normalmente significa `empresa cliente`
- si se habla del software que llama a la API, el termino correcto es `sistema integrador`
- si se habla del comprador comercial del vendedor, el termino correcto es `cliente final`

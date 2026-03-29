# Impacto de Nueva Documentacion 2026-03-29

## 1. Objetivo

Registrar el impacto de la nueva evidencia agregada en `docs/LEPTON API/Nueva Doc 29-3` sobre el proyecto de reemplazo de la API de optimizacion de corte.

La nueva evidencia no reemplaza la documentacion previa. La complementa con:

- endpoints reales de integracion
- header de autenticacion observado
- requests reales de produccion
- vinculacion entre requests y URLs del sistema integrador

## 2. Nuevas fuentes analizadas

- `docs/LEPTON API/Nueva Doc 29-3/api_test.md`
- `docs/LEPTON API/Nueva Doc 29-3/OPTIMIZACIONES.txt`
- `docs/LEPTON API/Nueva Doc 29-3/OPTIMIZACIONES (1).txt`
- `docs/LEPTON API/Nueva Doc 29-3/Lepton Optimizer _ XCONS Service Center.pdf`

## 3. Hallazgos nuevos confirmados

### 3.1 Endpoints reales

Se observa evidencia explicita de los endpoints:

- developer: `http://apidev.optimizadoronline.com/optimizar2`
- production: `https://apilepton.optimizadoronline.com/optimizar2`

Esto da una referencia operacional concreta para comparar el comportamiento de la API actual contra la futura implementacion.

### 3.2 Header de autenticacion observado

Se observa evidencia explicita del header:

- `X-User-Id: mocona`

Esto fortalece la hipotesis de que el sistema actual autentica o enruta requests al menos en parte por header, y no solo por contenido del payload.

### 3.3 Requests reales de produccion

Los archivos nuevos aportan 10 requests reales de un tenant concreto:

- `cliente = mocona`

Valor principal de esta evidencia:

- permite materializar fixtures reales de entrada
- permite observar que campos aparecen realmente en trafico productivo
- permite distinguir entre contrato teorico y contrato realmente usado

### 3.4 Vinculo con la UX del sistema integrador

Se observan URLs de marketplace del sistema integrador asociadas a requests concretos.

Esto habilita trazabilidad futura entre:

- una cotizacion o pedido del integrador
- el request enviado a la API
- la respuesta generada por el optimizador

## 4. Impacto sobre el contrato de entrada

### 4.1 El parser debe tolerar payloads sparsos

En varios requests reales, las piezas no incluyen todos los flags de tapacanto cuando no se usan.

Implicancia:

- el parser no debe asumir presencia completa de `arrN`, `abaN`, `derN`, `izqN`
- la normalizacion debe completar faltantes con `false`

### 4.2 `pedidos.archivo` no debe tratarse como identificador unico

En la nueva evidencia hay repeticion del mismo valor de `archivo` en distintos registros.

Implicancia:

- no usar `archivo` como primary key interna
- no usar `archivo` como unico criterio de idempotencia
- si se necesita idempotencia, debe definirse otra estrategia

### 4.3 El contrato observado real es bastante estable

En los 10 requests nuevos se observa alta estabilidad en varios campos:

- `tipo_cortes = 2`
- `fases = 2`
- `blocking = false`
- `modo_full = false`
- `op_frac = 0`
- `desp_sierra = 4.4`
- `desp_tapacantos = 0.06`
- `por_desp_tapacantos = 0`
- `refilado_tapacanto = 0`
- `refilado_x = 10`
- `refilado_y = 10`
- `hoja.margen_x = 100`
- `maquina_sel = 15`
- `maq_param1 = /DESCRICANTOS`

Implicancia:

- estos valores sirven como base de fixtures generales de uso real comun
- siguen sin invalidar la necesidad de soportar otras variantes del contrato

## 5. Impacto sobre tapacantos

### 5.1 `tapacantos[]` no contiene solo cantos fisicos

En los requests reales, el array incluye distintos tipos de item:

- tablero
- corte
- etiquetado de piezas
- canto ABS
- pegado de canto

Implicancia:

- el modelo interno no debe asumir que todo item de `tapacantos[]` participa en descuento de medidas
- tampoco debe asumir que todo item computa metros lineales de canto
- conviene clasificar internamente los items segun evidencia y reglas confirmadas

### 5.2 `espesor` llega en `0` aun cuando la descripcion textual indica espesores reales

Este es el hallazgo tecnico mas importante de la nueva evidencia.

Se observa que items cuya descripcion menciona `0.45mm`, `1mm` o `2mm` llegan con:

- `espesor = 0`

Implicancia fuerte:

- la regla documental actual que descuenta dimensiones usando `tapacantos[].espesor` queda debilitada para el trafico real observado
- no esta claro si:
  - el sistema integrador no completa ese valor
  - Lepton resuelve espesores por catalogo interno usando `codigo`
  - el descuento real no depende de este campo en estos clientes
  - la descripcion textual sea solo informativa

Conclusion operativa:

- el proyecto no debe cerrar definitivamente la formula de descuento por tapacanto usando solo los requests nuevos
- esta pasa a ser una pregunta abierta prioritaria

## 6. Impacto sobre el motor de optimizacion

### 6.1 La decision de motor propio no cambia

La nueva evidencia no contradice la decision ya tomada de implementar un algoritmo guillotina propio.

Sigue siendo valida porque:

- los archivos nuevos refuerzan el conocimiento del input real
- no aportan un formato alternativo de output que simplifique el problema
- no reducen la necesidad de construir `planos_corte`, sobrantes, metricas y futuras serializaciones propias

### 6.2 La validacion de rotacion sigue parcialmente abierta

En los requests nuevos:

- todos los materiales observados tienen `vetas = true`

Implicancia:

- esta evidencia no alcanza para validar casos reales de `vetas = false`
- la regla de rotacion libre sigue apoyandose en documentacion previa o fixtures sinteticos

### 6.3 Los casos reales refuerzan necesidades de normalizacion

El motor debera asumir una etapa de preprocesamiento robusta para:

- completar flags faltantes
- clasificar items de tapacanto
- derivar defaults operativos
- desacoplar input ruidoso del modelo interno de optimizacion

## 7. Impacto sobre arquitectura y operacion

### 7.1 Compatibilidad de autenticacion cobra mas peso

La nueva evidencia obliga a explicitar mejor la capa de compatibilidad con la API actual.

Implicancia:

- el skeleton de la nueva API deberia soportar el header actual observado
- la autenticacion por API key definida para la nueva implementacion debe convivir con una estrategia concreta de transicion si el integrador actual depende de `X-User-Id`

### 7.2 Sigue sin haber evidencia nueva del flujo async real

Los nuevos archivos aportan requests de entrada y datos operativos de acceso, pero no agregan:

- respuestas async
- estados de job
- callbacks
- polling real

Implicancia:

- la estrategia `async-first` no cambia
- el documento `07-contrato-async.md` sigue siendo una propuesta de diseno, no una reconstruccion de comportamiento observado

## 8. Impacto sobre fixtures y pruebas

### 8.1 Los nuevos requests deben materializarse como fixtures reales

Nueva prioridad recomendada:

1. crear una carpeta de fixtures reales basada en estos 10 requests
2. etiquetar cada fixture con observaciones del caso
3. usar el endpoint developer para intentar capturar responses reales equivalentes

### 8.2 Conviene separar fixtures por nivel de confianza

Se recomienda distinguir:

- fixtures documentales: derivados de ejemplo y PDFs previos
- fixtures reales: derivados de trafico observado
- fixtures sinteticos: creados para cubrir reglas no observadas en produccion

## 9. Riesgos nuevos o reforzados

### 9.1 Riesgo de sobreinferir desde un solo tenant

Toda la nueva evidencia real observada corresponde a `mocona`.

Implicancia:

- mejora mucho el conocimiento de un caso productivo
- pero no alcanza para concluir comportamiento general de las 4 empresas cliente

### 9.2 Riesgo de asumir semantica incorrecta de tapacantos

Como `espesor = 0` en casos que textualizan espesores no nulos, existe riesgo de modelar mal:

- descuento de dimensiones
- calculo de `pegado`
- calculo de `ml`

### 9.3 Riesgo de asumir sync definitivo

La evidencia nueva muestra requests reales del contrato actual, pero no debe reinterpretarse como invalidacion del diseno `async-first`.

## 10. Decisiones que no cambian

La nueva documentacion no cambia estas decisiones ya tomadas:

- backend en `ASP.NET Core Web API` con `.NET 8`
- arquitectura `async-first`
- soporte `sync` como adaptador de compatibilidad
- motor guillotina implementado en casa
- `planilla_vid` presente pero vacio hasta documentar su formato
- priorizacion de PDF antes que CNC

## 11. Acciones recomendadas

### Prioridad alta

- materializar los 10 requests nuevos como fixtures reales
- probar el endpoint developer con esos requests para capturar responses
- comparar responses reales contra la documentacion previa
- documentar formalmente el header `X-User-Id`
- abrir una linea de investigacion especifica sobre `tapacantos[].espesor`

### Prioridad media

- actualizar `02-contrato-api.md` con evidencia de trafico real
- actualizar `03-reglas-de-optimizacion.md` marcando la nueva ambiguedad sobre espesores
- actualizar `08-fixtures-generales.md` para incorporar fixtures reales y no solo generales

## 12. Conclusion

La nueva documentacion mejora mucho el conocimiento del sistema actual en dos frentes:

- compatibilidad operativa real
- estructura realmente usada del request

Su valor principal no esta en cambiar la arquitectura decidida, sino en reducir incertidumbre sobre:

- como entra el trafico real
- que campos usa realmente el integrador
- que supuestos documentales previos deben revisarse con mas cuidado

La mayor novedad tecnica es la tension entre la regla documental del espesor de tapacanto y los requests reales donde `espesor = 0`.

Ese punto debe considerarse abierto hasta capturar y analizar respuestas reales del endpoint developer.

## 13. Hallazgos adicionales de pruebas live contra endpoint developer

Despues de crear este documento se realizaron pruebas reales contra:

- `http://apidev.optimizadoronline.com/optimizar2`
- header `X-User-Id: mocona`

Estas pruebas agregan evidencia nueva que no estaba presente en los archivos de entrada.

### 13.1 El response real confirma la estructura raiz esperada

Los responses observados devuelven:

- `planilla`
- `planilla_vid`

Esto confirma que el endpoint developer sigue usando el contrato de salida general ya documentado.

### 13.2 `planilla_vid` ya no es solo un campo opaco teorico

Las pruebas reales devolvieron `planilla_vid` con contenido no vacio.

Implicancia:

- ya existe evidencia real suficiente para iniciar ingenieria inversa de su formato
- deja de ser solo un placeholder contractual abstracto
- conviene crear fixtures especificos de `planilla_vid` cuanto antes

### 13.3 `m2_utilizados` parece calcularse con la placa nominal, no con la placa refilada

En al menos un caso probado, `m2_utilizados` coincide con `material.base * material.altura` y no con `base refilada * altura refilada`.

Implicancia:

- la documentacion interna actual sobre metricas debe revisarse
- es posible que el motor opere sobre placa util pero reporte metricas globales usando dimensiones nominales de placa

### 13.4 Los descuentos por tapacanto no se reflejan en las dimensiones de pieza observadas en el response

En los casos probados con tapacantos activos, las piezas en `piezas_x_placa` conservaron las medidas nominales de entrada.

Implicancia:

- la API real podria estar reportando medida nominal aunque corte con otra medida interna
- o podria no estar descontando espesor cuando `espesor = 0`
- este punto queda definitivamente abierto y requiere mas evidencia antes de fijar reglas del motor

### 13.5 `desp_tapacantos = 0.06` parece comportarse como valor muy pequeno por lado, compatible con milimetros y no con metros

En dos pruebas reales, `ml - pegado = 0.00012`, lo que coincide con dos lados activos por `0.06` milimetros convertidos a metros.

Implicancia:

- la unidad operativa de `desp_tapacantos` debe revisarse
- es probable que no represente metros lineales directos sino una unidad menor aplicada por lado

### 13.6 El resumen de `tapacantos` no siempre devuelve solo los codigos del request

Se observaron dos comportamientos:

- sin tapacantos de entrada: aparecen codigos genericos como `MEL`, `PVC`, `AUX`, `tr1` a `tr4`
- con tapacantos de entrada: aparecen los codigos reales observados y se completan posiciones restantes con codigos `trN`

Implicancia:

- la salida de `tapacantos` tiene una logica estructural propia
- no debe modelarse como simple espejo del array de entrada

### 13.7 Las URLs de artefactos reales usan bucket dev y extensiones concretas

Las pruebas devolvieron URLs reales para:

- PDF de planos
- labels PDF
- archivo `.saw`
- archivo `.PTX`

Implicancia:

- ya hay evidencia operacional concreta para `url_planos`, `url_labels`, `url_cnc` y `url_aux_cnc`
- la idea previa de que CNC podia venir vacio en primeras iteraciones sigue siendo una decision valida para la nueva API, pero ya no describe lo que devuelve hoy el sistema observado

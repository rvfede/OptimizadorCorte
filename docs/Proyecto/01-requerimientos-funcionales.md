# Requerimientos Funcionales

## 1. Objetivo

Desarrollar una API de optimizacion de corte que reemplace a la API actualmente consumida por el sistema existente, manteniendo compatibilidad funcional y estructural con el contrato de entrada y salida actual.

## 2. Objetivo principal de negocio

La API debe recibir un pedido de corte de placas y devolver un resultado de optimizacion utilizable por el sistema actual sin requerir cambios en la integracion existente.

## 3. Requisito principal de compatibilidad

La API nueva debe ser compatible con la estructura actual de datos utilizada por el sistema integrador.

Esto implica:

- Mantener la misma jerarquia general del JSON de entrada.
- Mantener la misma jerarquia general del JSON de salida.
- Mantener los mismos nombres de propiedades.
- Mantener los mismos tipos de datos esperados.
- Mantener los campos que hoy son consumidos por el sistema externo.
- Evitar cambios breaking en la interfaz.

## 3.1 Restricciones iniciales de la solucion

En esta primera etapa:

- no se desarrollara frontend
- el producto a entregar es una API consumible por sistemas externos
- la autenticacion sera por API key
- la nueva API debe poder reemplazar a la actual en 4 empresas cliente con sus sistemas integradores ya conectados

Implicancia:

- la estabilidad del contrato y la operacion multi-tenant del servicio tienen prioridad sobre funcionalidades accesorias
- la observabilidad, la trazabilidad y el manejo de errores deben pensarse para integraciones server-to-server y por empresa cliente
## 3.2 Terminologia funcional

Para evitar confusion entre identidades distintas del negocio:

- empresa cliente: empresa vendedora de materiales autenticada por API key
- sistema integrador: software de gestion que consume la API
- cliente final: valor del campo pedidos.cliente dentro del pedido
## 4. Requerimientos funcionales del input

La API debe aceptar un JSON con la misma estructura general observada en el contrato actual.

El input debe permitir informar como minimo:

- Datos generales del pedido:
  - cliente
  - archivo
  - observaciones
  - fecha
  - hora
- Parametros de corte:
  - desperdicio de sierra
  - parametros de tapacantos
  - refilados
  - seleccion de maquina
- Configuracion del material:
  - codigo
  - descripcion
  - veta
  - peso
  - dimensiones de placa
  - restricciones minimas de corte o sobrante
- Configuracion de hoja o planilla.
- Opciones del optimizador.
- Configuracion de etiquetas.
- Datos de empresa.
- Definicion de tapacantos.
- Lista de piezas a cortar.

Cada pieza debe poder informar como minimo:

- cantidad
- base
- altura
- descripcion
- descripcion auxiliar
- si puede rotarse
- identificacion adicional si existe
- lados con tapacanto asociados

## 5. Requerimientos funcionales del proceso

La API debe:

- Interpretar el pedido recibido respetando la semantica actual del contrato.
- Optimizar el corte de piezas sobre placas.
- Considerar dimensiones de placas y piezas.
- Considerar rotacion permitida o no permitida.
- Considerar desperdicio de sierra.
- Considerar restricciones de material, como veta si aplica.
- Considerar refilados si forman parte del calculo.
- Considerar tapacantos asociados a los lados de cada pieza.
- Generar un resultado consistente con el formato esperado por el sistema integrador.

## 6. Requerimientos funcionales del output

La API debe devolver un JSON con la misma estructura general observada en el contrato actual.

El output debe incluir como minimo:

- Un resumen general de optimizacion.
- Cantidad de placas utilizadas.
- Cantidad de piezas cortadas.
- Metros cuadrados utilizados, cortados y sobrantes.
- Porcentaje de aprovechamiento.
- Datos de desperdicio de sierra.
- Metros lineales de bordes o tapacantos.
- Detalle de placas usadas.
- Detalle de planos o patrones de corte.
- Piezas asignadas a cada placa.
- Sobrantes generados por placa.
- Resumen de tapacantos requeridos.
- Cantidad y detalle de sobrantes utilizables.
- Referencias a entregables derivados:
  - plano PDF
  - etiquetas
  - CNC
  - auxiliares CNC si aplica

## 7. Requerimientos de compatibilidad de salida

La API debe preservar:

- Nombres de propiedades esperadas por el sistema actual.
- Estructura de nodos y colecciones.
- Presencia de campos actualmente utilizados aunque internamente se calculen de otra forma.
- Formatos de valores compatibles con la integracion actual.

## 8. Requerimientos de reemplazo

La API debe poder utilizarse como reemplazo de la implementacion actual con minimo impacto en el sistema existente.

Para eso debe:

- Aceptar el mismo tipo de request.
- Devolver el mismo tipo de response.
- No exigir cambios de mapeo en el sistema integrador salvo casos excepcionales y explicitamente documentados.

## 9. Reglas de evolucion

En esta etapa no se deben introducir cambios incompatibles en el contrato.

Cualquier mejora o extension debe cumplir una de estas condiciones:

- ser interna y transparente para la empresa cliente y su sistema integrador
- agregarse como campo opcional sin romper compatibilidad
- documentarse como version nueva del contrato

## 10. Fuentes de referencia

La base documental inicial para este contrato es:

- `docs/LEPTON API/Ejemplo.json`
- `docs/LEPTON API/Resultado.json`
- documentacion PDF asociada en `docs/LEPTON API`

## 11. Proximo paso

El siguiente documento debe definir el contrato exacto de entrada y salida campo por campo, indicando:

- nombre del campo
- tipo
- nivel jerarquico
- obligatoriedad
- significado funcional
- observaciones de compatibilidad



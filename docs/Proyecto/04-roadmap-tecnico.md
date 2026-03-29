# Roadmap Tecnico

## 1. Objetivo del roadmap

Planificar la implementacion de una API compatible con el contrato actual de optimizacion de corte, minimizando riesgo de ruptura sobre el sistema que hoy consume esa interfaz.

## 2. Principio rector

La prioridad del proyecto no es innovar primero en el algoritmo sino asegurar reemplazo compatible.

Orden correcto de trabajo:

1. congelar el contrato actual
2. construir pruebas de compatibilidad
3. implementar una API equivalente
4. reemplazar internamente el motor de optimizacion
5. recien despues optimizar o extender comportamiento

## 3. Etapas propuestas

### Etapa 0. Consolidacion documental

Objetivo:

- consolidar la especificacion funcional y tecnica del contrato actual

Entregables:

- `01-requerimientos-funcionales.md`
- `02-contrato-api.md`
- documentacion revisada con los PDFs convertidos a Markdown

Criterio de salida:

- existe una version acordada del contrato de request y response

### Etapa 1. Definicion de compatibilidad verificable

Objetivo:

- transformar la compatibilidad en algo testeable

Tareas:

- armar fixtures reales de request y response
- guardar ejemplos representativos en el repo
- definir que campos deben matchear exactamente
- definir tolerancias numericas si corresponden
- identificar campos opcionales y campos siempre requeridos

Entregables:

- carpeta de fixtures de compatibilidad
- checklist de validacion de contrato
- matriz de casos de prueba

Criterio de salida:

- cualquier implementacion puede validarse automaticamente contra el contrato esperado

### Etapa 2. Diseno de la API de reemplazo

Objetivo:

- definir la nueva implementacion manteniendo la misma interfaz externa

Tareas:

- definir endpoint o endpoints equivalentes
- decidir formato de validacion de entrada
- decidir estrategia de generacion de archivos derivados
- definir estructura interna de modelos de dominio
- mapear contrato externo a modelo interno

Entregables:

- especificacion tecnica de endpoints
- modelos internos del dominio
- estrategia de adaptacion request/response

Criterio de salida:

- hay una arquitectura clara para implementar sin tocar el contrato externo

### Etapa 3. Implementacion minima compatible

Objetivo:

- tener una primera version ejecutable del servicio

Tareas:

- crear skeleton del servicio
- implementar parser del request actual
- implementar serializer del response actual
- devolver datos de prueba compatibles para validar integracion
- exponer generacion basica de artefactos si aplica

Entregables:

- servicio levantando localmente
- primer request compatible
- primer response compatible

Criterio de salida:

- el sistema consumidor puede integrarse con la nueva API aunque el motor aun no este completo

### Etapa 4. Implementacion del motor de optimizacion

Objetivo:

- resolver funcionalmente el calculo de corte respetando el contrato actual

El motor se implementa como un algoritmo guillotina propio. Ver decision y diseno completo en `docs/Proyecto/10-motor-optimizacion.md`.

#### Etapa 4.1 Modelo interno de corte

Tareas:

- definir clase `PlateLayout` que representa el arbol de cortes de una placa
- definir clase `CutNode` con tipo (pieza, sobrante, corte), dimensiones y posicion
- definir clase `OptimizationResult` con patrones y metricas
- separar el modelo interno del contrato de salida

#### Etapa 4.2 Algoritmo guillotina base

Tareas:

- implementar recursion guillotina con `tipo_cortes` (vertical, horizontal, combinado)
- implementar manejo de veta: restriccion de rotacion segun material
- implementar ajuste de dimensiones efectivas por tapacantos antes de optimizar
- implementar calculo de desperdicio de sierra en cada corte

#### Etapa 4.3 Optimizacion sobre el base

Tareas:

- implementar `fases`: multiples pasadas con distintas ordenaciones de piezas, conservar mejor resultado
- implementar `modo_full`: correr 8 combinaciones de parametros y conservar el de mayor yield
- implementar `op_frac`: logica de fraccionamiento de la ultima placa incompleta
- implementar `unificar_areas`: fusion de sobrantes separados por corte si produce sobrante mayor reutilizable
- implementar `precorte`: corte previo cercano a mitad de placa si esta habilitado

#### Etapa 4.4 Post-procesamiento y consolidacion

Tareas:

- detectar y consolidar patrones de corte equivalentes en `planos_corte`
- calcular metricas globales: `m2_utilizados`, `m2_cortados`, `m2_sobrantes`, `ptje_aprov`, `cant_placas`
- identificar sobrantes reutilizables segun `min_base`, `min_altura`, `min_sup`
- calcular consumo de tapacantos por tipo (`ml`, `pegado`) segun configuracion de desperdicio
- mantener el arbol de cortes disponible para futura serializacion de `planilla_vid`

#### Etapa 4.5 Serializacion del response

Tareas:

- mapear modelo interno al contrato de salida JSON
- producir `planilla`, `planos_corte`, `sobrantes`, `tapacantos`, `placas_usadas`
- incluir `planilla_vid` preservando el comportamiento observado; mientras no se entienda su formato, capturar y versionar ejemplos reales para ingenieria inversa
- validar coherencia de totales antes de serializar

Entregables:

- motor funcional de optimizacion
- resultados comparables con fixtures existentes

Criterio de salida:

- la API produce resultados estructuralmente compatibles y funcionalmente plausibles
- el yield es razonable y los totales son coherentes internamente

### Etapa 5. Generacion de entregables

Objetivo:

- cubrir los artefactos que el sistema actual espera consumir o mostrar

Tareas:

- generar PDF de planos
- generar etiquetas
- definir alcance real de CNC y auxiliar CNC
- definir almacenamiento y URLs de salida

Entregables:

- generacion de `url_planos`
- generacion de `url_labels`
- estrategia para `url_cnc` y `url_aux_cnc`

Criterio de salida:

- el flujo operativo completo puede ejecutarse fin a fin

### Etapa 6. Validacion de reemplazo

Objetivo:

- probar el reemplazo en condiciones reales o casi reales

Tareas:

- comparar respuestas contra fixtures historicos
- correr pruebas de integracion con cada sistema integrador
- medir tiempos de respuesta
- revisar diferencias numericas o funcionales
- corregir incompatibilidades detectadas

Entregables:

- reporte de compatibilidad
- lista de gaps cerrados
- criterio de go-live

Criterio de salida:

- la nueva API esta lista para sustituir a la actual

## 3.1 Restricciones operativas actuales

El roadmap debe asumirse con estas restricciones:

- primera etapa sin frontend
- autenticacion de empresas cliente por API key
- reemplazo directo de una API ya usada por 4 sistemas externos

Implicancia tecnica:

- el servicio debe priorizar compatibilidad de contrato y disponibilidad
- conviene evitar dependencias innecesarias de frontend, sesiones o auth interactiva
- la infraestructura debe ser apta para integraciones server-to-server estables
## 4. Riesgos principales

- documentacion incompleta o ambigua de algunos campos
- reglas de negocio implicitas no visibles en los JSON de ejemplo
- dependencia real del sistema integrador sobre campos poco documentados
- complejidad de replicar exactamente `planilla_vid`
- diferencia entre compatibilidad estructural y compatibilidad funcional real

## 5. Estrategia recomendada

La estrategia recomendada es incremental:

- primero compatibilidad de contrato
- despues compatibilidad de integracion
- despues compatibilidad funcional
- y recien al final mejoras de algoritmo o performance

## 6. Proximos documentos recomendados

- `03-reglas-de-optimizacion.md`
- `05-casos-de-prueba.md`
- `06-decisiones-tecnicas.md`




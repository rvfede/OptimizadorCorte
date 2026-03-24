# Decisiones Tecnicas

## 1. Objetivo del documento

Registrar las decisiones tecnicas iniciales para implementar la nueva API de optimizacion de corte con foco en compatibilidad, operacion estable y velocidad de ejecucion del proyecto.

## 2. Contexto operativo confirmado

Estado actual del proyecto:

- primera etapa sin frontend
- el producto a entregar es una API consumida por sistemas externos
- objetivo inmediato: reemplazar la API actual en 4 empresas cliente con sus sistemas integradores ya conectados`r`n- hoy existe 1 unico sistema integrador para esas 4 empresas cliente
- la autenticacion sera por API key
- se prioriza conservar el comportamiento y contrato actuales mientras no exista una razon fuerte para cambiarlo
- se desea trazabilidad completa al inicio, pero con retencion temporal de payloads
- existe disposicion a migrar sistemas integradores a un modelo asincrono si eso mejora el sistema a largo plazo

## 2.1 Terminologia recomendada

Para evitar ambiguedades en el proyecto se recomienda usar estos terminos:

- empresa cliente: empresa vendedora de materiales que contrata o utiliza el optimizador y posee una API key propia
- sistema integrador: software de gestion del vendedor que consume la API en nombre de la empresa cliente
- cliente final: cliente comercial del vendedor, representado por el campo pedidos.cliente en el contrato funcional

Esta distincion es importante porque el campo cliente ya existe dentro del pedido y no debe confundirse con la identidad autenticada de la empresa que consume la API.
## 3. Decision de arquitectura principal

### 3.1 Modelo general

La API se implementara como un servicio backend unico, con arquitectura orientada a jobs.

Decision:

- arquitectura `async-first` 
- el sistema soportara operacion `sync` y `async` 
- el modo asincrono es la direccion objetivo del producto y de la interfaz publica
- el modo sincrono, si existe, debe tratarse como una capa de compatibilidad transitoria o una capacidad acotada, no como el camino principal

Interpretacion tecnica:

- toda optimizacion se modela internamente como un job
- el producto puede exponer ambos modos de consumo 
- la interfaz asincrona debe ser el contrato preferido hacia futuro
- la interfaz sincrona no debe condicionar el diseño central del sistema
- si algun consumidor necesita sync al inicio, debe resolverse como adaptador y no como base del producto

### 3.2 Compatibilidad externa

Mientras el objetivo sea reemplazar la API actual sin friccion:

- se mantendra el contrato actual de request y response en la medida necesaria para la transicion
- se mantendra la forma actual de autenticacion mientras no haya una limitacion tecnica relevante
- cualquier mejora de buenas practicas debe evaluarse por impacto sobre las 4 empresas cliente y sus sistemas integradores

Decision adicional:

- si migrar sistemas integradores a async mejora robustez, escalabilidad y experiencia de usuario, esa migracion debe considerarse una opcion valida y deseable

## 4. Stack recomendado

### 4.1 Backend

- `ASP.NET Core Web API`
- `.NET 8`
- `System.Text.Json`
- `FluentValidation`

Motivos:

- contrato fuertemente tipado
- buena performance para una API de calculo
- excelente soporte para Docker y CI/CD
- muy buen fit para desarrollo asistido por IA

### 4.2 Testing

- `xUnit`
- `FluentAssertions`
- tests de compatibilidad contra fixtures reales de request/response

### 4.3 Empaquetado y despliegue

- `Docker`
- `GitHub Actions`

## 5. Infraestructura recomendada

### 5.1 Runtime principal

Recomendacion principal:

- `Google Cloud Run`

Motivos:

- muy buen encastre con servicios .NET dockerizados
- pago por uso con una entrada de costo baja
- mas adecuado para una API critica que free tiers con sleep agresivo

### 5.2 Base de datos

Recomendacion inicial:

- `Supabase Postgres`

Uso previsto:

- empresas cliente (tenants)
- API keys
- jobs de optimizacion
- auditoria basica
- estado de procesamiento
- metricas agregadas
- errores y eventos operativos

### 5.3 Storage de artefactos

Recomendacion inicial:

- `Supabase Storage` o almacenamiento compatible tipo objeto

Uso previsto:

- PDF de planos
- etiquetas
- archivos CNC y auxiliares si aplica

## 6. Modelo operativo recomendado

### 6.1 Async-first como camino principal

La recomendacion es adoptar modo asincrono como contrato preferido.

#### Modo asincrono

Flujo sugerido:

1. el cliente envia el pedido
2. la API valida, persiste y crea un job
3. la API devuelve `job_id` y estado inicial
4. el cliente consulta estado y resultado

Ventajas:

- mejor control de tiempos largos
- menor riesgo de timeout
- mejor trazabilidad operativa
- mejor escalabilidad para cargas pesadas o generacion de artefactos

#### Modo sincrono

El modo sincrono, si se expone, debe definirse como una capacidad secundaria.

Flujo sugerido:

1. el cliente envia el pedido
2. la API crea internamente el job
3. la API intenta resolverlo dentro de una ventana corta de espera
4. si termina a tiempo, devuelve el response compatible completo
5. si no termina a tiempo, debe derivar al flujo asincrono o responder segun la estrategia de compatibilidad definida

Decision importante:

- no se debe diseñar el sistema alrededor del modo sync
- no conviene optimizar la arquitectura para sostener indefinidamente clientes bloqueantes si eso degrada el diseño global
- si un cliente puede migrar a async con beneficio real, deberia promoverse esa migracion
- en la practica, el sistema podra operar en ambos modos, pero las nuevas implementaciones deberian orientarse a async por defecto

### 6.2 Trazabilidad y retencion

Decision tomada:

- guardar inicialmente todos los requests y responses para facilitar depuracion
- conservar metricas y errores en forma persistente
- conservar payloads completos solo temporalmente

Politica inicial recomendada:

- retencion de requests/responses completos: `30 dias`
- retencion de metricas y errores: permanente o largo plazo
- politica de purge automatica diaria

Esta retencion debe quedar configurable por ambiente.

## 7. Modelo de datos recomendado

Tablas o entidades iniciales sugeridas:

- `api_tenants`
- `api_keys`
- `optimization_jobs`
- `optimization_requests`
- `optimization_responses`
- `optimization_artifacts`
- `error_events`
- `usage_metrics_daily`

### 7.1 `api_tenants`

Deberia representar a cada empresa cliente autenticada por API key.

Campos sugeridos:

- `id`
- `name`
- `status`
- `notes`
- `created_at`
- `updated_at`

### 7.2 `api_keys`

La key no deberia guardarse en texto plano.

Campos sugeridos:

- `id`
- `client_id`
- `key_prefix`
- `key_hash`
- `status`
- `expires_at`
- `last_used_at`
- `created_at`
- `revoked_at`

### 7.3 `optimization_jobs`

Deberia modelar el ciclo completo del proceso.

Campos sugeridos:

- `id`
- `client_id`
- `mode` (`sync` o `async`)
- `status`
- `request_received_at`
- `processing_started_at`
- `processing_finished_at`
- `error_code`
- `error_message`
- `contract_version`
- `correlation_id`

### 7.4 `optimization_requests` y `optimization_responses`

Deberian almacenar payloads completos mientras dure la ventana de retencion.

Campos sugeridos:

- `job_id`
- `payload_json`
- `created_at`
- `expires_at`

## 8. Seguridad y autenticacion

### 8.1 Modelo de autenticacion

Se usara autenticacion por API key.

Decision operativa:

- mantener la forma de autenticacion actual mientras no impida la evolucion del sistema
- encapsular internamente la autenticacion en middleware propio

### 8.2 Recomendaciones de implementacion

- almacenar solo hash de la API key
- loguear `key_prefix` o identificador derivado, no la key completa
- asociar cada key a una empresa cliente
- permitir desactivacion o rotacion por empresa cliente

## 9. Observabilidad minima necesaria

Desde la primera fase, la API deberia registrar:

- `correlation_id`
- `job_id`
- empresa cliente autenticada
- modo de ejecucion (`sync` o `async`)
- tiempos de validacion, procesamiento y serializacion
- resultado final (`success`, `validation_error`, `processing_error`)
- artefactos generados

## 10. Decisiones que se posponen

Quedan deliberadamente pospuestas:

- arquitectura multi-servicio
- colas externas dedicadas
- rate limiting sofisticado por empresa cliente
- auth mas compleja que API key
- estrategia final de versionado publico del contrato
- mecanismo final de interpretacion de `planilla_vid`

## 11. Riesgos y observaciones

- si las 4 empresas cliente o sus sistemas integradores dependen fuertemente del comportamiento sincrono, la transicion a async visible externamente puede requerir una etapa intermedia
- la compatibilidad de auth debe validarse contra cada empresa cliente y su sistema integrador real
- la retencion de payloads completos requiere definir politica de limpieza desde el inicio
- la generacion de PDFs y archivos CNC puede mover el cuello de botella fuera del optimizador puro
- sostener sync como modo principal puede degradar la arquitectura futura; por eso debe evitarse como decision estructural

## 12. Proximo paso sugerido

El siguiente paso util despues de este documento es:

- definir fixtures generales representativos del uso comun
- diseñar el contrato asincrono objetivo
- decidir si el modo sync sera temporal, optativo o limitado por SLA
- bajar la especificacion de errores y estados de job






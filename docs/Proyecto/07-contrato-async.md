# Contrato Async

## 1. Objetivo del documento

Definir el contrato operativo asincrono de la API sin duplicar el contrato funcional ya existente.

Principio rector:

- el contrato async reutiliza el contrato sync
- y solo agrega los campos operativos necesarios para crear, monitorear y recuperar jobs

## 2. Principio de compatibilidad

El modo async no define un modelo funcional distinto.

Esto significa:

- el payload funcional del pedido sigue siendo el mismo
- el resultado funcional final sigue siendo el mismo
- async solo agrega una capa operativa para procesamiento desacoplado

En consecuencia:

- `sync` = contrato funcional completo resuelto en una sola respuesta
- `async` = mismo contrato funcional, pero dividido en aceptacion del trabajo y recuperacion posterior del resultado

## 2.1 Terminologia operativa

En este documento:

- empresa cliente = empresa vendedora de materiales que posee la API key
- sistema integrador = software de gestion que llama a la API
- cliente final = valor de pedidos.cliente dentro del payload funcional
## 3. Estrategias posibles de exposicion

Se consideran dos alternativas compatibles con esta regla:

### Opcion A. Mismo endpoint con metadato de modo

Ejemplo conceptual:

- `POST /optimizaciones`
- body funcional actual + metadato de operacion

Ventaja:

- minimiza divergencia entre sync y async

### Opcion B. Endpoints separados para sync y async

Ejemplo conceptual:

- `POST /optimizaciones/sync`
- `POST /optimizaciones/async`

Ventaja:

- deja mas explicita la intencion operativa
- simplifica validaciones de cada flujo

Recomendacion inicial:

- usar endpoints separados a nivel operativo
- pero conservar el mismo payload funcional base en ambos modos

## 4. Payload funcional base

El request async debe incluir exactamente el mismo payload funcional que el modo sync.

Referencia:

- ver `02-contrato-api.md`

Conceptualmente:

```json
{
  "pedidos": {
    "...": "mismo contrato funcional que sync"
  }
}
```

## 5. Campos operativos adicionales del modo async

Sobre el payload funcional base, el modo async puede agregar un bloque operativo minimo.

Propuesta:

```json
{
  "pedidos": {
    "...": "contrato funcional actual"
  },
  "async_options": {
    "client_reference": "string opcional",
    "callback_url": "string opcional",
    "callback_headers": {
      "Header-Name": "value"
    },
    "ttl_hours": 24
  }
}
```

### 5.1 Campos propuestos en `async_options`

| Campo | Tipo | Obligatorio | Objetivo |
| --- | --- | --- | --- |
| `client_reference` | string | no | Identificador externo del sistema integrador o de la empresa cliente. |
| `callback_url` | string | no | URL para notificacion cuando el job finaliza. |
| `callback_headers` | object | no | Headers adicionales para callback. |
| `ttl_hours` | number | no | Tiempo de retencion operativa del job o del resultado. |

Regla:

- estos campos no alteran la logica funcional del optimizador
- solo afectan el ciclo operativo del trabajo asincrono

## 6. Endpoint de creacion async

### 6.1 Propuesta

`POST /optimizaciones/async`

### 6.2 Request

- body: contrato funcional actual + `async_options` opcional
- auth: API key

### 6.3 Response inicial

La respuesta inicial async no devuelve el resultado funcional final. Devuelve aceptacion del trabajo.

Propuesta:

```json
{
  "job": {
    "id": "job_123",
    "status": "queued",
    "mode": "async",
    "created_at": "2026-03-24T14:00:00Z",
    "client_reference": "abc-123",
    "status_url": "/optimizaciones/jobs/job_123",
    "result_url": "/optimizaciones/jobs/job_123/result"
  }
}
```

### 6.4 Campos de la respuesta inicial

| Campo | Tipo | Objetivo |
| --- | --- | --- |
| `job.id` | string | Identificador unico del job. |
| `job.status` | string | Estado inicial del proceso. |
| `job.mode` | string | Modo de operacion. |
| `job.created_at` | datetime string | Fecha de creacion. |
| `job.client_reference` | string | Eco del identificador externo si fue enviado. |
| `job.status_url` | string | URL para polling. |
| `job.result_url` | string | URL para recuperar resultado final. |

## 7. Endpoint de estado

### 7.1 Propuesta

`GET /optimizaciones/jobs/{job_id}`

### 7.2 Objetivo

Permitir a un sistema integrador conocer el estado del job sin descargar aun el resultado completo.

### 7.3 Response propuesta

```json
{
  "job": {
    "id": "job_123",
    "status": "processing",
    "mode": "async",
    "created_at": "2026-03-24T14:00:00Z",
    "started_at": "2026-03-24T14:00:02Z",
    "finished_at": null,
    "client_reference": "abc-123",
    "has_result": false,
    "result_url": "/optimizaciones/jobs/job_123/result",
    "error": null
  }
}
```

### 7.4 Estados sugeridos

Estados minimos recomendados:

- `queued`
- `processing`
- `completed`
- `failed`
- `expired`
- `cancelled` si en el futuro se habilita cancelacion

## 8. Endpoint de resultado final

### 8.1 Propuesta

`GET /optimizaciones/jobs/{job_id}/result`

### 8.2 Objetivo

Devolver el resultado funcional final del optimizador.

### 8.3 Regla principal

Cuando el job este completado, este endpoint debe devolver el mismo contrato funcional final que el modo sync, agregando como maximo un envoltorio operativo minimo si fuera necesario.

Propuesta preferida:

- devolver directamente el mismo response funcional actual

Ejemplo conceptual:

```json
{
  "planilla": {
    "...": "mismo contrato funcional que sync"
  },
  "planilla_vid": "..."
}
```

Alternativa permitida si se necesita metadata operativa:

```json
{
  "job": {
    "id": "job_123",
    "status": "completed"
  },
  "result": {
    "planilla": {
      "...": "mismo contrato funcional que sync"
    },
    "planilla_vid": "..."
  }
}
```

Recomendacion inicial:

- si es posible, devolver el mismo response funcional puro para maximizar reutilizacion del sistema integrador

## 9. Relacion entre sync y async

### 9.1 Sync como adaptador del mismo core

El modo sync debe usar el mismo pipeline interno del modo async.

Propuesta:

1. `sync` crea internamente un job
2. espera dentro de una ventana corta configurable
3. si el job termina, devuelve el contrato funcional final
4. si no termina, la estrategia dependera de compatibilidad e implementacion

### 9.2 Regla de diseño

No deben existir dos motores distintos:

- uno para sync
- otro para async

Debe existir un solo flujo interno de negocio basado en jobs.

## 10. Errores async

### 10.1 Errores de aceptacion

Aplican al `POST /optimizaciones/async`:

- autenticacion invalida
- payload invalido
- contrato incompatible
- empresa cliente deshabilitada o API key invalida

### 10.2 Errores de procesamiento

Aplican cuando el job fue aceptado pero falla durante la ejecucion.

Propuesta para `GET /optimizaciones/jobs/{job_id}`:

```json
{
  "job": {
    "id": "job_123",
    "status": "failed",
    "error": {
      "code": "OPTIMIZATION_ERROR",
      "message": "descripcion breve del error"
    }
  }
}
```

## 11. Callback opcional

Si se habilita `callback_url`, el sistema puede notificar al sistema integrador cuando el job termina.

Propuesta inicial:

- mantener callback como opcional
- no hacerlo obligatorio para el MVP
- no depender del callback para que el flujo async funcione

Payload sugerido del callback:

```json
{
  "job": {
    "id": "job_123",
    "status": "completed",
    "result_url": "/optimizaciones/jobs/job_123/result"
  }
}
```

## 12. Compatibilidad con empresas cliente y sistemas integradores actuales

Estrategia recomendada:

- las empresas cliente y sus sistemas integradores actuales pueden comenzar usando sync si hoy dependen de ese comportamiento
- nuevas integraciones o migraciones deberian orientarse a async
- si un sistema integrador puede migrar a async sin alto impacto, conviene promoverlo

## 13. Puntos a definir despues

Queda por definir con mayor precision:

- nombres finales de endpoints
- si el resultado async devolvera response puro o response con envoltorio operativo
- tiempo maximo de espera del modo sync
- politica de expiracion del resultado async
- politica de retry de callbacks
- codigos HTTP definitivos por estado y error

## 14. Proximo paso sugerido

El siguiente documento util deberia ser uno de estos dos:

- `09-errores-y-estados.md`
- `08-fixtures-generales.md`

Si hay que priorizar uno, conviene empezar por `08-fixtures-generales.md` para validar el comportamiento comun esperado y luego incorporar variantes reales si aparecen necesidades concretas.




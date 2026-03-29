# Handoff Para Nueva Sesion

## Objetivo

Este documento sirve como punto de arranque para continuar el trabajo en una nueva sesion sin perder el contexto del proyecto.

## Prompt recomendado

```text
Estamos trabajando en `C:\REPOS\OptimizadorCorte`.

Quiero continuar la documentacion y diseño de una API de optimizacion de corte que debe reemplazar una API existente, manteniendo compatibilidad funcional y estructural.

Antes de proponer nada, revisa estos documentos del repo:

- `docs/Proyecto/README.md`
- `docs/Proyecto/00-glosario.md`
- `docs/Proyecto/02-contrato-api.md`
- `docs/Proyecto/03-reglas-de-optimizacion.md`
- `docs/Proyecto/06-decisiones-tecnicas.md`
- `docs/Proyecto/08-fixtures-generales.md`
- `docs/Proyecto/10-motor-optimizacion.md`
- `docs/Proyecto/11-impacto-nueva-documentacion-2026-03-29.md`
- `docs/Proyecto/99-handoff-nueva-sesion.md`

Contexto importante:

- hoy hay 1 unico sistema integrador y 4 empresas cliente
- se capturo evidencia real nueva del tenant `mocona`
- ya se probaron requests reales contra `http://apidev.optimizadoronline.com/optimizar2`
- se confirmo que el response real devuelve `planilla`, `planilla_vid` y URLs de artefactos
- `planilla_vid` ya no debe tratarse como campo sin evidencia real
- `pedidos.cliente` se observa como identificador del tenant o empresa cliente, no como cliente final comercial
- hay incertidumbre abierta sobre semantica real de tapacantos, unidades y descuentos de dimensiones

Despues de leer eso, quiero que continúes con: [ACA PONES LA TAREA]
```

## Decisiones tomadas que siguen vigentes

- el algoritmo de optimizacion es guillotina implementado en casa, sin libreria externa
- los resultados deben ser funcionalmente optimos, no bit-a-bit identicos al sistema anterior
- la arquitectura objetivo sigue siendo `async-first`
- sync debe tratarse como adaptador de compatibilidad, no como base del diseño
- infra objetivo: Google Cloud Run + Supabase Postgres + Supabase Storage + GitHub Actions + Docker

## Evidencia nueva incorporada en esta sesion

Se analizo nueva documentacion en:

- `docs/LEPTON API/Nueva Doc 29-3/api_test.md`
- `docs/LEPTON API/Nueva Doc 29-3/OPTIMIZACIONES.txt`
- `docs/LEPTON API/Nueva Doc 29-3/OPTIMIZACIONES (1).txt`

Se confirmo evidencia operacional:

- endpoint developer: `http://apidev.optimizadoronline.com/optimizar2`
- endpoint production: `https://apilepton.optimizadoronline.com/optimizar2`
- header observado: `X-User-Id: mocona`

Se capturaron responses reales del endpoint developer con al menos 3 casos.

## Hallazgos clave que cambian el estado del proyecto

### 1. `planilla_vid` ya tiene evidencia real

Se obtuvieron responses reales con `planilla_vid` no vacio.

Implicancia:

- ya conviene atacar ingenieria inversa del formato
- deja de ser correcto asumirlo solo como placeholder vacio de referencia

### 2. `pedidos.cliente` no debe seguir leyendose como cliente final por defecto

En la evidencia real observada:

- `pedidos.cliente = mocona`

Implicancia:

- se comporta como identificador del tenant o empresa cliente
- el glosario y documentos de contrato ya fueron corregidos en ese sentido

### 3. El response real refuerza varias hipotesis y corrige otras

Se confirmo:

- estructura raiz `planilla` + `planilla_vid`
- URLs reales para `url_planos`, `url_labels`, `url_cnc`, `url_aux_cnc`
- `planilla.tapacantos` con estructura propia, no espejo directo del request

Se detecto tension abierta en:

- semantica de `tapacantos[].espesor`
- unidad de `desp_tapacantos`
- criterio exacto de descuento de dimensiones por canto
- criterio exacto de `m2_utilizados`, que parece usar placa nominal

## Documentos actualizados en esta sesion

- `docs/Proyecto/00-glosario.md`
- `docs/Proyecto/02-contrato-api.md`
- `docs/Proyecto/03-reglas-de-optimizacion.md`
- `docs/Proyecto/06-decisiones-tecnicas.md`
- `docs/Proyecto/08-fixtures-generales.md`
- `docs/Proyecto/10-motor-optimizacion.md`
- `docs/Proyecto/11-impacto-nueva-documentacion-2026-03-29.md`
- `docs/Proyecto/99-handoff-nueva-sesion.md`

## Siguiente foco recomendado

El siguiente paso mas valioso ya no es infraestructura.

El siguiente foco recomendado es cerrar la ingenieria inversa funcional con evidencia real.

Orden recomendado:

1. materializar fixtures reales request/response en el repo
2. crear `docs/Proyecto/12-ingenieria-inversa-planilla-vid.md`
3. crear `docs/Proyecto/13-semantica-tapacantos.md`
4. revisar `05-casos-de-prueba.md` a la luz de evidencia live
5. luego recien bajar esa informacion al skeleton `.NET`

## Tareas concretas que pueden hacerse en la proxima sesion

- guardar como fixtures los 3 casos ya probados contra developer
- correr mas requests reales del set `mocona` y capturar responses
- comparar respuestas entre casos con y sin tapacantos
- empezar a descomponer `planilla_vid` en tokens y correlacionarlo con `planos_corte`
- documentar hipotesis verificables sobre `m2_utilizados`, `ml_bordes`, `cant_desp_sierra` y `tapacantos`
- si se prefiere avanzar en implementacion, proponer estructura inicial del proyecto `.NET` basada ya en evidencia real de entrada y salida

## Nota operativa

La estrategia recomendada para continuar ahora es:

- leer primero la documentacion actualizada
- usar evidencia real antes que inferencias viejas cuando entren en conflicto
- preservar el contrato observado aunque internamente se modele mejor
- separar siempre: comportamiento observado, hipotesis de trabajo y decisiones de implementacion propias

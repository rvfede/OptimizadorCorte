# Handoff Para Nueva Sesion
codex resume 019d1c41-4bd9-7661-8351-09fe3a8c542e
## Objetivo

Este documento sirve como punto de arranque para continuar el trabajo en una nueva sesion sin perder el contexto del proyecto.

## Prompt recomendado

```text
Estamos trabajando en `C:\REPOS\OptimizadorCorte`.

Quiero continuar la documentación y diseño de una API de optimización de corte que debe reemplazar una API existente, manteniendo compatibilidad funcional y estructural.

Antes de proponer nada, revisa estos documentos del repo:

- `docs/Proyecto/README.md`
- `docs/Proyecto/00-glosario.md`
- `docs/Proyecto/01-requerimientos-funcionales.md`
- `docs/Proyecto/02-contrato-api.md`
- `docs/Proyecto/03-reglas-de-optimizacion.md`
- `docs/Proyecto/04-roadmap-tecnico.md`
- `docs/Proyecto/05-casos-de-prueba.md`
- `docs/Proyecto/06-decisiones-tecnicas.md`
- `docs/Proyecto/07-contrato-async.md`
- `docs/Proyecto/08-fixtures-generales.md`

Contexto importante:

- hoy hay 1 unico sistema integrador y 4 empresas cliente
- cada empresa cliente usa su propia API key
- la API es una sola para todos
- el sistema debe soportar sync y async
- la arquitectura debe ser async-first
- sync no debe condicionar el diseño central
- por ahora asumimos uso general comun, sin customizaciones por empresa cliente salvo evidencia futura

Despues de leer eso, quiero que continúes con: [ACA PONES LA TAREA]
```

## Tareas sugeridas para continuar

- redactar `docs/Proyecto/09-errores-y-estados.md`
- materializar fixtures reales en el repo a partir del ejemplo actual
- proponer la estructura inicial del proyecto `.NET`
- comenzar la implementacion de la API
- definir el esquema inicial de base de datos para tenants, api keys y jobs

## Recomendacion de uso

En una nueva sesion:

1. pegar el prompt anterior
2. reemplazar `[ACA PONES LA TAREA]` por una tarea concreta
3. pedir que primero lea los documentos antes de avanzar

## Nota

La estrategia recomendada para continuar es siempre:

- leer documentacion existente
- respetar decisiones ya tomadas
- no reinventar contrato ni terminologia
- avanzar sobre el siguiente documento o entregable concreto

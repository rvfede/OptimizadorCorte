# Proyecto

## Objetivo

Desarrollar una API propia de optimizacion de corte que reemplace a la API actualmente utilizada por el sistema, manteniendo compatibilidad funcional y estructural con el contrato existente.

## Referencia inicial analizada

- `docs/LEPTON API/Ejemplo_Lepton/Ejemplo_request.json`
- `docs/LEPTON API/Ejemplo_Lepton/Resultado_response.json`
- `docs/LEPTON API/API - JSON.md`
- `docs/LEPTON API/API - JSON (Generico).md`

## Criterio principal

El objetivo de esta etapa no es redefinir el contrato sino preservarlo.

Esto implica:

- mantener la estructura general del input
- mantener la estructura general del output
- mantener nombres de propiedades, jerarquias y tipos esperados
- evitar cambios incompatibles para el sistema ya integrado

## Alcance funcional inicial

El sistema debe recibir un pedido de corte sobre placas y devolver un plan de corte optimizado, respetando la interfaz actual.

## Restricciones iniciales

En la primera etapa del proyecto:

- no habra frontend
- el servicio sera consumido como API por sistemas existentes
- la autenticacion sera por API key
- el objetivo inmediato es reemplazar la API actual en 4 empresas cliente con sus sistemas integradores
- hoy existe 1 unico sistema integrador para esas 4 empresas cliente
- la arquitectura sera async-first, con soporte sync y async pero orientando nuevas implementaciones a async
- se guardaran requests/responses temporalmente y metricas/errores a largo plazo

## Terminologia recomendada

- empresa cliente: empresa vendedora de materiales autenticada por API key
- sistema integrador: software de gestion que consume la API
- cliente final: valor de `pedidos.cliente` dentro del pedido

Ver tambien:

- `docs/Proyecto/00-glosario.md`

## Documentos del proyecto

- `docs/Proyecto/00-glosario.md`
- `docs/Proyecto/01-requerimientos-funcionales.md`
- `docs/Proyecto/02-contrato-api.md`
- `docs/Proyecto/03-reglas-de-optimizacion.md`
- `docs/Proyecto/04-roadmap-tecnico.md`
- `docs/Proyecto/05-casos-de-prueba.md`
- `docs/Proyecto/06-decisiones-tecnicas.md`
- `docs/Proyecto/07-contrato-async.md`
- `docs/Proyecto/08-fixtures-generales.md`
- `docs/Proyecto/10-motor-optimizacion.md`
- `docs/Proyecto/99-handoff-nueva-sesion.md`

## Proximo foco de trabajo

Los siguientes pasos recomendados son:

1. revisar `02-contrato-api.md` con foco en diferencias entre variante generica y variante concreta
2. usar `03-reglas-de-optimizacion.md` para cerrar reglas pendientes
3. convertir `05-casos-de-prueba.md` en fixtures reales dentro del repo
4. definir fixtures generales y cerrar el contrato operativo async

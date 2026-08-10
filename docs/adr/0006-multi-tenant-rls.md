# ADR-0006: Aislamiento multi-tenant — Row Level Security por `organization_id`

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

`docs/producto.md` es explícito: el aislamiento por organización se construye desde la Fase 1, no como migración futura (sección 9, principio 5), y es una de las reglas de producto no negociables. Hace falta fijar el mecanismo técnico concreto.

## Decisión

Todas las tablas de negocio incluyen una columna `organization_id` no nula, y cada una tiene policies de Row Level Security en Postgres que restringen `select`/`insert`/`update`/`delete` a las filas cuya `organization_id` coincide con la organización del usuario autenticado (resuelta vía `auth.uid()` contra la tabla `users`). La policy se escribe en la misma migración que crea la tabla — nunca como paso separado posterior.

## Alternativas consideradas

- **Aislamiento solo a nivel de aplicación** (filtrar por `organization_id` en cada query desde el código): es el enfoque más común en proyectos chicos, pero traslada la responsabilidad de seguridad a que cada Server Action, sin excepción, recuerde agregar el filtro. Un solo olvido — plausible en un proyecto donde un agente de IA nuevo escribe una query sin todo el contexto — filtra datos entre organizaciones. Se descarta como único mecanismo por ser demasiado frágil para el estándar que pide `docs/producto.md`.
- **Una base de datos (o schema) separada por organización:** aísla al nivel más fuerte posible, pero multiplica el trabajo operativo (migraciones, backups, conexiones) por cada organización nueva — inviable para un modelo de negocio de decenas o cientos de clientes pequeños pagando USD 12-75/mes.
- **RLS como mecanismo único y no negociable:** la elegida. Es el punto medio correcto: aislamiento real a nivel de base de datos, sin multiplicar infraestructura por cliente.

## Consecuencias

- Cada migración que crea una tabla nueva debe incluir su policy de RLS en el mismo archivo — se trata como parte inseparable de crear la tabla, no como una tarea de seguridad posterior.
- El código de aplicación puede confiar en que Supabase ya filtra por organización — no hace falta (ni se debe) replicar ese filtro manualmente en cada query, lo cual simplifica las Server Actions.
- Antes de sumar el primer tenant externo (Fase 2), conviene una prueba explícita de que el aislamiento funciona (dos organizaciones de prueba, verificar que ninguna ve datos de la otra) — parte de la revisión de seguridad pendiente en `docs/producto.md` sección 17.

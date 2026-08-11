# ADR-0013: Datos semilla y organización demo en producción

**Estado:** Aceptada
**Fecha:** 2026-08-11

## Contexto

Antes de la Fase 2 (canales y beta) no hay agentes externos usando el sistema con datos reales, pero igual hace falta poder mostrarle el producto a un agente potencial de forma convincente — con datos que parezcan una operación real, no una pantalla vacía. El modelo multi-tenant con RLS ([ADR-0006](0006-multi-tenant-rls.md)) ya aísla los datos por organización, lo que permite resolver esto sin construir un modo "demo" especial en el código.

## Decisión

Se crea un script de seed que genera datos ficticios pero realistas (contactos, propiedades, conversaciones, tareas y visitas con nombres y casos representativos, no `lorem ipsum`) dentro de una organización dedicada, marcada con un flag `organizations.is_demo = true`. Ese script corre en local, staging **y producción** — la organización demo existe en producción desde la V0, específicamente para mostrar el producto a agentes potenciales sin depender de tener ya un cliente real cargado.

La organización demo es una organización más a nivel de datos y RLS — ningún camino de código la trata como caso especial. Lo único distinto es una señal visual en la UI (banner/badge "Demo") cuando un usuario está dentro de esa organización, para que nadie la confunda con datos reales propios o de un cliente.

## Alternativas consideradas

- **Demo solo en staging, mostrar el producto desde ahí:** evita el problema de mezclar demo con producción, pero un deploy preview de staging no es una URL estable ni pensada para mostrarle algo a un tercero — depende de qué PR esté abierto en ese momento. No sirve como material de venta.
- **Datos ficticios hardcodeados en el frontend, sin pasar por Supabase:** más simple de construir, pero no demuestra el producto real (no prueba que RLS, la bandeja de aprobaciones o el flujo de datos funcionan) — sería una maqueta, no una demo del sistema.
- **Clonar datos reales de la organización del equipo fundador, anonimizados:** más trabajo de mantenimiento (hay que re-anonimizar cada vez que cambian los datos reales) y más riesgo de filtrar algo sensible por un error de anonimización. Datos sintéticos desde el origen son más simples y no tienen ese riesgo.

## Consecuencias

- El schema de `organizations` incluye la columna `is_demo` desde la primera migración (ver `docs/arquitectura.md` sección 4).
- Hace falta un mecanismo de reseteo: si alguien interactúa con la demo en producción (crea, edita o borra algo durante una demostración en vivo), el estado puede quedar inconsistente. El script de seed debe ser re-ejecutable de forma idempotente (borra y recrea los datos demo) y se corre manualmente o con un job programado periódico.
- Los reportes y métricas de uso pensados para el negocio real (Fase 3/4) deben poder excluir a la organización demo — cualquier query de métricas agregadas filtra por `is_demo = false` salvo que se pida explícitamente lo contrario.
- La organización demo nunca contiene datos personales reales de terceros — solo datos sintéticos —, así que no activa las obligaciones de la Ley 25.326 que sí aplican desde el primer tenant externo real (`docs/producto.md` sección 17).

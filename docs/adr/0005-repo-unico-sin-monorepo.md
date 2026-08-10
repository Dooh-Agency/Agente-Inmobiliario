# ADR-0005: Estructura del repositorio — aplicación única, sin monorepo

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

Con Next.js manejando tanto UI como capa de API ([ADR-0004](0004-server-actions-sin-backend-separado.md)), hay que decidir si el repositorio se organiza como una única aplicación o como un monorepo con `apps/` y `packages/` separados, pensando en una eventual expansión futura (panel admin separado, app mobile).

## Decisión

Un único proyecto Next.js en la raíz del repositorio (junto a `docs/`), sin estructura de monorepo. Se confirma explícitamente esta elección con el usuario antes de escribirla: dado un desarrollador solo trabajando con agentes de IA, se prioriza la estructura más simple posible sobre la que anticipa una escala que todavía no existe.

## Alternativas consideradas

- **Monorepo (Turborepo/Nx con `apps/web`, `packages/ui`, `packages/db`, etc.):** el patrón correcto cuando hay múltiples aplicaciones reales que comparten código, o varios desarrolladores trabajando en paralelo sobre distintas partes. Hoy ninguna de las dos condiciones se cumple. Adoptarlo ahora agrega configuración (workspaces, build orchestration) que no paga su costo y que un agente de IA nuevo tendría que aprender a navegar sin necesidad real.

## Consecuencias

- Toda la lógica compartida (tipos de Supabase, validaciones Zod, componentes UI) vive en carpetas dentro de la misma app (`lib/`, `components/`), no en paquetes separados.
- Si en la Fase 4 (o antes) aparece una necesidad real de una segunda aplicación (por ejemplo, un panel de administración interno de DOOH separado del producto de cara al cliente), este ADR queda reemplazado por uno nuevo que evalúe la migración a monorepo en ese momento, con el contexto real de esa necesidad — no antes.

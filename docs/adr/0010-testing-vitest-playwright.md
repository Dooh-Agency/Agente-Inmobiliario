# ADR-0010: Testing — Vitest + Playwright

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

Se eligió arrancar la V0 con rigor de ingeniería "profesional desde el día 1" (CI, staging, tests) en vez de mínimo viable, dado que el producto va a manejar datos reales de terceros antes de la Fase 2. Hace falta definir con qué herramientas se testea.

## Decisión

Vitest para tests unitarios y de integración liviana (validaciones de Zod, lógica de negocio pura, utilidades). Playwright para tests end-to-end de los flujos críticos: login, aislamiento multi-tenant (una organización no debe ver datos de otra), y el flujo de aprobar/editar/descartar de la bandeja de aprobaciones.

## Alternativas consideradas

- **Jest en vez de Vitest:** Jest es la opción más establecida, pero Vitest es más rápido, tiene configuración más simple sobre Vite/Next.js moderno, y comparte sintaxis compatible con Jest (baja fricción de adopción).
- **Cypress en vez de Playwright:** ambas son opciones válidas para e2e. Se elige Playwright por mejor soporte multi-browser out of the box y por integrarse de forma directa con GitHub Actions sin servicios adicionales.
- **No escribir tests e2e en la V0:** contradice la elección explícita de rigor profesional — los flujos de aislamiento multi-tenant y aprobación humana son exactamente los que no se pueden permitir romper silenciosamente, dado que son las dos reglas de producto no negociables de `docs/producto.md` sección 7.

## Consecuencias

- No se persigue cobertura total de tests desde el día 1 — se prioriza cubrir los flujos donde un bug sería grave (fuga de datos entre organizaciones, mensaje enviado sin aprobación) por sobre cobertura exhaustiva de UI.
- Los tests e2e de Playwright corren contra el deploy preview de staging (ver [ADR-0012](0012-entornos-dev-staging-prod.md)), no contra producción.
- El pipeline de CI ([ADR-0011](0011-cicd-github-actions-netlify.md)) debe fallar el build si algún test de estos dos flujos críticos falla — no son tests opcionales.

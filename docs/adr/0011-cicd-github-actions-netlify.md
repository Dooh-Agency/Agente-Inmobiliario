# ADR-0011: CI/CD — GitHub Actions + Netlify

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

`docs/producto.md` fija Netlify como hosting. Con la elección de rigor "profesional desde el día 1", hace falta un pipeline de integración continua que corra antes de cada deploy, y una estrategia de despliegue automatizado.

## Decisión

GitHub Actions ejecuta, en cada Pull Request: instalación de dependencias, lint (ESLint), typecheck (`tsc --noEmit`), tests unitarios (Vitest) y build. Netlify se conecta directamente al repositorio de GitHub: genera un deploy preview por cada PR (usado como entorno de staging, ver [ADR-0012](0012-entornos-dev-staging-prod.md)) y despliega a producción automáticamente al mergear a `main`. Los tests e2e de Playwright corren contra el deploy preview antes de habilitar el merge en ramas que tocan flujos críticos.

## Alternativas consideradas

- **Netlify Build Plugins como único mecanismo de CI (sin GitHub Actions):** Netlify puede correr comandos de build propios, pero GitHub Actions da más control sobre qué bloquea un merge (checks requeridos en la rama `main`) y es donde vive naturalmente el paso de Playwright contra el preview.
- **Vercel en vez de Netlify:** técnicamente comparable para un proyecto Next.js, incluso con mejor integración nativa. Se descarta porque `docs/producto.md` ya fija Netlify como decisión de producto (costos ya estimados en la sección 13 del documento), y este ADR no reabre esa decisión.
- **Despliegue manual:** descartado explícitamente al elegir el nivel de rigor "profesional desde el día 1" — un deploy manual es una fuente de errores humanos y no deja registro reproducible de qué versión está en producción.

## Consecuencias

- Ningún cambio llega a `main` sin pasar lint, typecheck, tests y build en verde.
- El costo de este pipeline es tiempo de espera en cada PR — aceptado como trade-off correcto dado que el producto va a manejar datos reales de terceros pronto.
- Requiere mantener actualizados los secrets de GitHub Actions (credenciales de Supabase de staging, API key de OpenAI de test) por separado de los de Netlify.

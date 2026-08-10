# Registro de decisiones de arquitectura (ADRs)

Este directorio guarda las decisiones de arquitectura del proyecto, en formato ADR liviano (basado en MADR). Cada archivo documenta **una** decisión: el contexto que la motivó, qué se decidió, qué alternativas se evaluaron y por qué se descartaron, y las consecuencias de esa elección.

## Por qué existen

El proyecto se construye por un desarrollador solo trabajando con agentes de IA (ver `docs/arquitectura.md`). Sin un registro explícito de decisiones, cada nueva sesión de IA tendría que re-derivar el "por qué" de cada elección técnica a partir del código, o peor, asumir uno distinto cada vez. Los ADRs son la memoria de las decisiones que no son obvias mirando el código.

## Cuándo escribir un ADR nuevo

Cuando se toma una decisión de arquitectura que sería costosa de revertir, que tiene alternativas razonables descartadas, o que un agente de IA podría "corregir" por accidente si no conoce el motivo original (por ejemplo, agregar un backend separado, o mover una tabla fuera del esquema multi-tenant). No hace falta un ADR para decisiones de implementación reversibles (nombre de una función, estructura de una carpeta de componentes).

## Formato

```markdown
# ADR-00XX: Título de la decisión

**Estado:** Aceptada | Reemplazada por ADR-00YY | En revisión
**Fecha:** AAAA-MM-DD

## Contexto
Qué problema había que resolver y qué restricciones aplicaban.

## Decisión
Qué se decidió, en una o dos frases directas.

## Alternativas consideradas
Qué otras opciones se evaluaron y por qué se descartaron.

## Consecuencias
Qué se gana, qué se resigna, y qué queda pendiente de revisar a futuro.
```

## Índice

| ADR | Decisión |
|---|---|
| [0001](0001-framework-nextjs-typescript.md) | Framework y lenguaje: Next.js (App Router) + TypeScript |
| [0002](0002-supabase-backend.md) | Base de datos y backend: Supabase (PostgreSQL) |
| [0003](0003-supabase-auth.md) | Autenticación: Supabase Auth |
| [0004](0004-server-actions-sin-backend-separado.md) | Capa de API: Server Actions / Route Handlers, sin backend separado |
| [0005](0005-repo-unico-sin-monorepo.md) | Estructura del repositorio: aplicación única, sin monorepo |
| [0006](0006-multi-tenant-rls.md) | Aislamiento multi-tenant: Row Level Security por `organization_id` |
| [0007](0007-ui-tailwind-shadcn.md) | UI y estilos: Tailwind CSS + shadcn/ui |
| [0008](0008-validacion-zod-rhf.md) | Validación de datos: Zod + React Hook Form |
| [0009](0009-orquestacion-ia-sin-n8n-v0.md) | Orquestación de IA: llamadas directas a OpenAI; n8n diferido |
| [0010](0010-testing-vitest-playwright.md) | Testing: Vitest + Playwright |
| [0011](0011-cicd-github-actions-netlify.md) | CI/CD: GitHub Actions + Netlify |
| [0012](0012-entornos-dev-staging-prod.md) | Estrategia de entornos: local / staging / producción |

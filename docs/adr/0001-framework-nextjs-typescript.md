# ADR-0001: Framework y lenguaje — Next.js (App Router) + TypeScript

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

El producto es una aplicación web privada, responsive, pensada como PWA (`docs/producto.md` sección 10), operada por un desarrollador solo con agentes de IA como pair-programmers. `docs/producto.md` sección 9 ya fija Next.js comoframework de aplicación web; este ADR fija las decisiones que ese documento deja abiertas: versión del router, lenguaje y nivel de tipado.

## Decisión

Next.js con **App Router** (no Pages Router) y **TypeScript en modo estricto** (`strict: true`) en todo el proyecto, sin excepciones de archivo.

## Alternativas consideradas

- **Pages Router:** es el modelo anterior de Next.js. Se descarta porque el App Router es donde Next.js concentra el desarrollo activo, tiene mejor soporte de Server Components (relevante para no exponer lógica ni claves de OpenAI/Supabase al cliente) y es el camino recomendado por el propio framework para proyectos nuevos.
- **JavaScript sin tipos:** más rápido de escribir al principio, pero en un proyecto donde un agente de IA nuevo entra sin contexto acumulado, el tipado es la forma más barata de que ese agente detecte errores de integración (campos de `organizations`/`contacts` mal referenciados, por ejemplo) sin tener que ejecutar la app.
- **Remix / SvelteKit / otro framework:** no evaluados en profundidad — `docs/producto.md` ya fija Next.js como decisión de producto, este ADR no la reabre.

## Consecuencias

- Cada Server Component y Server Action puede acceder a Supabase y a la API de OpenAI sin exponer credenciales al bundle del cliente — base para [ADR-0004](0004-server-actions-sin-backend-separado.md).
- El tipado estricto obliga a generar y mantener actualizados los tipos de Supabase (`supabase gen types typescript`) como parte del flujo de trabajo, no como un paso opcional.
- Queda descartado usar `any` como escape hatch habitual; si aparece, debe ser una excepción puntual y justificada, no un patrón.

# ADR-0004: Capa de API — Server Actions / Route Handlers, sin backend separado

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

Hay que decidir cómo se comunica la interfaz con Supabase y con la API de OpenAI: si a través de un backend dedicado (API REST/GraphQL separada) o directamente desde la capa de servidor que ya ofrece Next.js.

## Decisión

No se construye un backend separado. Toda la lógica de servidor (leer/escribir en Supabase, llamar a la API de OpenAI, validar input) vive en Server Actions y Route Handlers de Next.js, dentro de la misma aplicación.

## Alternativas consideradas

- **Backend dedicado (Node/Express, NestJS, o funciones serverless separadas):** se ajusta mejor a equipos con backend y frontend separados, o cuando varios clientes (web, mobile, integraciones de terceros) necesitan el mismo API. Ninguno de esos casos aplica hoy: hay un solo cliente (la web app) y un solo desarrollador. Sumar un backend separado implica un segundo deploy, un segundo repositorio o carpeta a mantener sincronizada, y más superficie para que un agente de IA nuevo se pierda entre dos bases de código.
- **GraphQL (vía Supabase o una capa propia):** resuelve un problema (fetching flexible desde múltiples clientes) que este proyecto no tiene todavía. Se puede reevaluar si en el futuro aparece una app mobile nativa que se beneficie de GraphQL.

## Consecuencias

- Menor complejidad operativa: un solo proyecto, un solo pipeline de CI/CD, un solo deploy.
- Si en el futuro se necesita exponer una API pública (por ejemplo, para integraciones de terceros con el producto ya validado comercialmente), esa es API nueva a diseñar explícitamente, no una extensión implícita de las Server Actions internas.
- Las Server Actions y Route Handlers son el único lugar donde se valida input con Zod (ver [ADR-0008](0008-validacion-zod-rhf.md)) y donde se aplican reglas de negocio — ningún Client Component debe llamar a Supabase directamente para escrituras sensibles.

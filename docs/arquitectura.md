# Arquitectura técnica — Copiloto Inmobiliario

**Estado:** vigente desde el inicio de la Fase 0 técnica.
**Fecha:** 10 de agosto de 2026.
**Relación con otros documentos:** este documento traduce a decisiones técnicas concretas lo que [`docs/producto.md`](producto.md) define a nivel de producto (secciones 5 a 9, 12). Las decisiones puntuales con alternativas evaluadas están registradas como ADRs en [`docs/adr/`](adr/README.md) — este documento da la vista integrada; los ADRs dan el detalle y el razonamiento de cada decisión individual.

Contexto de equipo que condiciona cada elección de acá: un solo desarrollador (el equipo fundador) trabajando con agentes de IA (Claude Code, Codex) como pair-programmers, sin otros desarrolladores humanos sumándose en el corto plazo. Eso empuja hacia convenciones explícitas y poco ambiguas (para que un agente de IA nuevo pueda retomar el proyecto sin arqueología), y en contra de abstracciones tempranas que solo se justifican con equipos grandes.

## 1. Vista general

```
┌─────────────────────────────────────────────────────────────────┐
│                         Next.js (Netlify)                        │
│  ┌───────────────┐   ┌────────────────────┐   ┌───────────────┐ │
│  │ App Router UI │──▶│ Server Actions /    │──▶│  Supabase JS  │ │
│  │ (React RSC)   │   │ Route Handlers      │   │  client (SSR) │ │
│  └───────────────┘   └────────┬───────────┘   └───────┬───────┘ │
└────────────────────────────────┼───────────────────────┼─────────┘
                                  │                       │
                                  ▼                       ▼
                        ┌──────────────────┐    ┌──────────────────┐
                        │   API de OpenAI   │    │     Supabase      │
                        │ (clasificación,   │    │  Postgres + Auth   │
                        │  borradores)      │    │  + Storage + RLS   │
                        └──────────────────┘    └──────────────────┘

Fuera de la V0 (se activan en fases posteriores, ver docs/roadmap-tecnico.md):
  Meta Cloud API (WhatsApp/Instagram) · Google Calendar · n8n · pasarela de pago
```

No hay backend separado. Next.js, corriendo en Netlify, concentra la interfaz y la capa de API (Server Actions y Route Handlers). Supabase es la única pieza de infraestructura con estado (base de datos, autenticación, almacenamiento de archivos). Todo lo demás (IA, canales, automatizaciones, cobros) son servicios externos que la aplicación consume, no sistemas que haya que operar.

## 2. Principios que guían la arquitectura

Heredados de `docs/producto.md` sección 9, con su traducción técnica:

1. **Base de datos propia y estructurada** → Postgres administrado por Supabase, con schema versionado en el repo (migraciones SQL), no una base ad-hoc ni dependencia de herramientas cerradas.
2. **Arquitectura modular** → cada integración de canal (WhatsApp, Instagram, Calendar) se implementa como un adaptador aislado que escribe sobre las mismas tablas (`conversations`, `messages`, `appointments`), para poder sumar o sacar canales sin tocar el núcleo.
3. **Seguridad y trazabilidad desde el inicio** → RLS activo desde la primera migración, tabla `approvals` para auditoría de decisiones humanas, timestamps y autoría en todo registro sensible.
4. **Interfaz simple, núcleo preparado para crecer** → UI construida sobre un sistema de componentes consistente (shadcn/ui) en vez de piezas one-off, para que agregar pantallas no implique reinventar patrones.
5. **Multi-tenant desde el inicio** → toda tabla de negocio lleva `organization_id`, y el aislamiento se aplica en la base de datos (RLS), no solo en la capa de aplicación — para que un bug en el código de la app no filtre datos entre organizaciones.

## 3. Stack técnico

| Capa | Tecnología | ADR |
|---|---|---|
| Framework y lenguaje | Next.js (App Router) + TypeScript en modo estricto | [ADR-0001](adr/0001-framework-nextjs-typescript.md) |
| Base de datos y backend | Supabase (PostgreSQL + Row Level Security) | [ADR-0002](adr/0002-supabase-backend.md) |
| Autenticación | Supabase Auth | [ADR-0003](adr/0003-supabase-auth.md) |
| Capa de API | Server Actions y Route Handlers de Next.js | [ADR-0004](adr/0004-server-actions-sin-backend-separado.md) |
| Estructura del repositorio | Aplicación única (sin monorepo) | [ADR-0005](adr/0005-repo-unico-sin-monorepo.md) |
| Aislamiento multi-tenant | RLS por `organization_id` | [ADR-0006](adr/0006-multi-tenant-rls.md) |
| UI y estilos | Tailwind CSS + shadcn/ui | [ADR-0007](adr/0007-ui-tailwind-shadcn.md) |
| Validación de datos | Zod + React Hook Form | [ADR-0008](adr/0008-validacion-zod-rhf.md) |
| Orquestación de IA | Llamadas directas a la API de OpenAI desde el servidor; sin n8n en la V0 | [ADR-0009](adr/0009-orquestacion-ia-sin-n8n-v0.md) |
| Testing | Vitest (unitario) + Playwright (end-to-end) | [ADR-0010](adr/0010-testing-vitest-playwright.md) |
| CI/CD | GitHub Actions + despliegue en Netlify | [ADR-0011](adr/0011-cicd-github-actions-netlify.md) |
| Entornos | Local / staging / producción, con proyectos Supabase separados | [ADR-0012](adr/0012-entornos-dev-staging-prod.md) |
| Datos demo | Organización `is_demo` con datos sintéticos, también en producción | [ADR-0013](adr/0013-datos-semilla-organizacion-demo.md) |

Componentes definidos en `docs/producto.md` pero **fuera de alcance de la V0** (se detallan en `docs/roadmap-tecnico.md`): Meta Cloud API (WhatsApp/Instagram), Google Calendar, n8n, pasarela de cobro.

## 4. Modelo de datos

Las entidades mínimas ya están definidas en `docs/producto.md` sección 9: `organizations`, `users`, `subscriptions`, `contacts`, `contact_requirements`, `properties`, `conversations`, `messages`, `activities`, `tasks`, `appointments`, `matches`, `approvals`.

Convenciones técnicas para el schema (a implementar como migraciones SQL versionadas en el repo, gestionadas con el CLI de Supabase):

- Toda tabla de negocio (todas excepto tablas puramente de catálogo/lookup) tiene una columna `organization_id uuid not null references organizations(id)`.
- Toda tabla tiene `created_at` y `updated_at` con default `now()` y trigger de actualización.
- Las claves primarias son `uuid` generadas con `gen_random_uuid()`, no seriales incrementales — evita filtrar el volumen de registros entre organizaciones y facilita sincronización futura si hiciera falta.
- Los `enum` de Postgres se usan para campos de estado cerrados (`contacts.status`, `tasks.status`), reflejando los estados de `docs/producto.md` sección 5 (`Nuevo → Calificado → Visita → Negociación → Cerrado`, más los complementarios).
- Cada política RLS se escribe junto con la migración que crea la tabla, nunca como paso separado posterior — ver [ADR-0006](adr/0006-multi-tenant-rls.md).
- `organizations` incluye una columna `is_demo boolean not null default false`, que marca la organización de demostración sembrada con datos sintéticos (ver [ADR-0013](adr/0013-datos-semilla-organizacion-demo.md)). Cualquier query de métricas/reportes agregados filtra `is_demo = false` salvo que se pida explícitamente lo contrario.

## 5. Capas de la aplicación

- **UI (React Server Components):** páginas y componentes bajo `app/`. Se favorece Server Components por defecto; Client Components solo donde hay interactividad real (formularios, tarjetas de aprobación con acciones).
- **Server Actions / Route Handlers:** única puerta de entrada a datos y lógica de negocio. Reciben input validado con Zod, llaman a Supabase (con el cliente server-side, respetando RLS vía el JWT del usuario) y devuelven datos tipados.
- **Supabase:** Postgres + Auth + Storage. El acceso desde el cliente del navegador es limitado a lo que las policies de RLS permiten; las operaciones sensibles pasan siempre por Server Actions.
- **API de OpenAI:** invocada solo desde el servidor (nunca expuesta al cliente), con function calling / structured outputs para las clasificaciones y borradores de la sección 7 de `docs/producto.md`. Ver [ADR-0009](adr/0009-orquestacion-ia-sin-n8n-v0.md) sobre por qué no hay una capa de orquestación (n8n) en la V0.

## 6. Entornos y pipeline

Tres entornos desde el arranque, dado el nivel de rigor elegido (ver [ADR-0012](adr/0012-entornos-dev-staging-prod.md)):

| Entorno | Next.js | Supabase | Propósito |
|---|---|---|---|
| Local | `next dev` en la máquina del desarrollador | Proyecto Supabase de desarrollo (o Supabase local vía CLI/Docker) | Desarrollo día a día |
| Staging | Deploy preview de Netlify por PR | Proyecto Supabase de staging | Validar cambios antes de producción, probar migraciones |
| Producción | Deploy de Netlify sobre `main` | Proyecto Supabase de producción | Datos reales del equipo fundador y, más adelante, de organizaciones beta |

Pipeline en cada PR (GitHub Actions, ver [ADR-0011](adr/0011-cicd-github-actions-netlify.md)): instalar dependencias → lint → typecheck → tests unitarios → build. El merge a `main` dispara el deploy de producción en Netlify. Los tests end-to-end (Playwright) corren contra el deploy preview de staging antes de habilitar el merge a `main` en ramas que tocan flujos críticos (aprobación de mensajes, aislamiento multi-tenant).

## 7. Seguridad y operación mínima

- **RLS como límite real, no solo la UI.** Ninguna query a Supabase desde el navegador debe depender de que el frontend "no muestre" datos de otra organización — la policy de RLS es la que decide qué filas son visibles.
- **Secrets:** variables de entorno gestionadas en Netlify (por entorno) y nunca committeadas; `.env.example` documenta qué variables hacen falta sin exponer valores reales.
- **Manejo de errores en producción:** se integra un servicio de error tracking (recomendado: Sentry, plan gratuito alcanza en esta etapa) desde la V0.1, para tener visibilidad real antes de sumar el primer agente beta con datos de terceros — no es parte del stack "core" pero es barato de sumar y evita operar a ciegas.
- **Respaldos:** los respaldos automáticos de Supabase (incluidos en su plan) se consideran suficientes para la etapa de piloto cerrado; antes de sumar el primer tenant externo, verificar y documentar el procedimiento de restauración (ver `docs/producto.md` sección 17, punto de seguridad pendiente).
- Amenazas formales, RTO/RPO y una revisión de seguridad más rigurosa siguen pendientes como indica `docs/producto.md` sección 17 — esta arquitectura no los resuelve, los deja con un piso razonable (RLS + backups + error tracking) hasta que se haga esa revisión.

## 8. Qué queda deliberadamente fuera de la V0

Para que ningún agente de IA asuma que hay que construir esto ya:

- **n8n** — no se despliega en la V0. Se retoma cuando la Fase 2/3 lo requiera (integraciones de canal, automatizaciones de seguimiento). Ver [ADR-0009](adr/0009-orquestacion-ia-sin-n8n-v0.md).
- **Meta Cloud API (WhatsApp/Instagram)** y **Google Calendar** — integraciones de Fase 2, no de la V0.
- **Pasarela de cobro** — no hay lógica de facturación real hasta la Fase 4. La tabla `subscriptions` existe desde la V0.1 a nivel de modelo de datos, pero sin integración de pago activa.
- **Multi-app / monorepo** — se descartó para la V0 (ver [ADR-0005](adr/0005-repo-unico-sin-monorepo.md)); si en el futuro hace falta separar un panel admin o una app mobile, ese es el momento de reevaluar la estructura del repositorio, no antes.

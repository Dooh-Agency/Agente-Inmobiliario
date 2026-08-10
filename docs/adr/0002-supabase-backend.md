# ADR-0002: Base de datos y backend — Supabase (PostgreSQL)

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

`docs/producto.md` sección 9 ya define Supabase/PostgreSQL como base de datos y autenticación, con Row Level Security para el aislamiento por organización. Este ADR confirma esa decisión desde el criterio técnico y deja registradas las alternativas evaluadas, dado que es la decisión de infraestructura de mayor impacto del proyecto (todo el modelo multi-tenant depende de ella).

## Decisión

Supabase como backend-as-a-service: PostgreSQL administrado, autenticación (ver [ADR-0003](0003-supabase-auth.md)), Storage para archivos, y Row Level Security como mecanismo de aislamiento multi-tenant (ver [ADR-0006](0006-multi-tenant-rls.md)).

## Alternativas consideradas

- **Firebase (Firestore):** base NoSQL, peor ajuste para un modelo de datos altamente relacional (contactos ↔ propiedades ↔ matches ↔ approvals, con integridad referencial real). Además, RLS de Postgres es más expresivo que las reglas de seguridad de Firestore para el caso de aislamiento por organización.
- **Backend propio (Node/Express o similar) + Postgres autoalojado:** da más control, pero implica operar infraestructura (servidor, backups, parches de seguridad) que un desarrollador solo no debería cargar en la etapa de validación. Supabase resuelve auth, backups y la capa de datos con una superficie operativa mínima.
- **PlanetScale / Neon (Postgres/MySQL serverless sin auth ni RLS integrados):** hubiera requerido resolver autenticación y aislamiento multi-tenant por separado, sumando piezas móviles sin necesidad.

## Consecuencias

- El aislamiento multi-tenant se resuelve a nivel de base de datos (RLS), no solo en el código de la aplicación — ver [ADR-0006](0006-multi-tenant-rls.md).
- El plan gratuito de Supabase alcanza para desarrollo y validación, pero no se debe depender de él una vez que haya datos reales de un tenant externo (ya señalado en `docs/producto.md` secciones 13 y 17) — pasar a plan pago es una tarea de la V0.2, no antes.
- Atarse a Supabase implica cierto vendor lock-in; se acepta como trade-off razonable dado que es Postgres estándar por debajo — una migración futura a Postgres autoalojado seguiría siendo técnicamente posible si hiciera falta.

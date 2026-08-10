# ADR-0012: Estrategia de entornos — local / staging / producción

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

Con el nivel de rigor "profesional desde el día 1" elegido para la V0, hace falta decidir cuántos entornos existen y cómo se relacionan con Supabase (que no comparte una sola instancia entre entornos de forma segura, dado que cada uno necesita su propio schema y datos de prueba).

## Decisión

Tres entornos desde el arranque de la V0:

- **Local:** cada desarrollador (hoy, el equipo fundador) corre `next dev` contra un proyecto Supabase de desarrollo (o Supabase local vía CLI/Docker si el volumen de trabajo lo justifica).
- **Staging:** un proyecto Supabase separado, contra el que corren los deploy previews de Netlify (uno por PR) y los tests e2e de Playwright.
- **Producción:** proyecto Supabase de producción, con los datos reales del equipo fundador desde la V0.1 y, más adelante, de las organizaciones beta.

Las migraciones SQL se versionan en el repositorio y se aplican en el mismo orden a los tres entornos (local → staging → producción), nunca editadas a mano en un entorno sin pasar por el repo.

## Alternativas consideradas

- **Un solo entorno (producción desde el día 1, sin staging):** es lo que hubiera correspondido bajo el nivel de rigor "mínimo viable", que no fue el elegido. Sin staging, cada cambio de schema o de lógica de negocio se probaría directamente contra los datos reales del equipo fundador — riesgo innecesario dado que ya se optó por más rigor.
- **Staging compartiendo el mismo proyecto Supabase que producción, con un schema separado:** técnicamente posible, pero aumenta el riesgo de que una migración o un bug de RLS mal probado en "staging" afecte datos de producción al compartir la misma instancia. Proyectos separados cuestan lo mismo (ambos gratuitos en esta etapa) y eliminan ese riesgo.

## Consecuencias

- Hay que mantener credenciales y variables de entorno distintas para tres contextos (local, staging, producción en Netlify + GitHub Actions).
- Los seeds de datos de prueba para staging deben mantenerse mínimamente realistas (organizaciones, contactos y propiedades ficticias) para que los tests e2e tengan sentido.
- Cuando se sume el primer tenant externo (V0.2), staging pasa a ser también el lugar donde se prueban las políticas de RLS con múltiples organizaciones antes de confiar en ellas en producción.

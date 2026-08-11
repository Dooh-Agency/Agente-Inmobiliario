# Roadmap técnico por versiones — Copiloto Inmobiliario

**Relación con otros documentos:** `docs/producto.md` sección 12 define las **Fases de producto** (0 a 4: qué se construye y por qué). Este documento define las **versiones técnicas** que implementan cada fase: entregables concretos, criterio de "hecho" y qué queda explícitamente fuera de cada versión. Cada versión técnica se corresponde con una fase de producto, pero divide el trabajo en pasos verificables por un solo desarrollador operando con agentes de IA.

No arrancar una versión sin haber cerrado el criterio de salida de la anterior, salvo que el usuario lo pida explícitamente (misma regla que ya rige en `AGENTS.md` para las fases de producto).

## V0 — Fundacional (implementa la Fase 0 de producto)

**Objetivo:** dejar la base técnica lista para construir funcionalidad, sin construir todavía pantallas de negocio.

Entregables:

- Repositorio Next.js inicializado (TypeScript estricto, ESLint, Prettier, Husky + lint-staged).
- Proyecto Supabase creado (producción) + segundo proyecto Supabase de staging.
- Primera migración SQL: las 12 tablas mínimas de `docs/producto.md` sección 9, con `organization_id`, timestamps y policies de RLS desde la primera versión de cada tabla (no agregadas después).
- Autenticación con Supabase Auth funcionando (login, logout, sesión persistida).
- Pipeline de CI en GitHub Actions: lint, typecheck, test, build en cada PR.
- Deploy conectado a Netlify: producción sobre `main`, deploy preview por PR.
- `.env.example` documentado y variables reales cargadas en Netlify por entorno.
- Script de seed idempotente que crea una organización `is_demo = true` con contactos, propiedades, conversaciones y tareas ficticias pero realistas, ejecutado en local, staging **y producción** — para poder mostrar el producto a un agente potencial desde el día 1 (ver [ADR-0013](adr/0013-datos-semilla-organizacion-demo.md)).
- ADRs de esta etapa aprobados (`docs/adr/`).

**Criterio de salida:** se puede crear una organización, loguearse, y ver una pantalla vacía protegida por sesión, con CI verde y deploy automático funcionando en los tres entornos, y la organización demo visible y navegable en producción.

**Fuera de alcance:** cualquier pantalla de negocio (contactos, propiedades, agenda), IA, canales.

## V0.1 — Núcleo funcional (implementa la Fase 1 de producto)

**Objetivo:** el equipo fundador puede operar su día a día real dentro del sistema, sin IA ni canales conectados.

Entregables:

- Alta y gestión de organización y usuarios (el equipo fundador como primer tenant).
- CRUD de contactos, con `contact_requirements` embebido en la ficha.
- CRUD de propiedades.
- Tareas y agenda (`tasks`, `appointments`) con responsable y fecha obligatorios.
- Estados de contacto (`Nuevo → Calificado → Visita → Negociación → Cerrado` + complementarios) reflejados en la UI.
- Pantalla de Inicio con resumen diario (mensajes nuevos, pendientes, visitas del día, sin seguimiento).
- Bandeja de aprobaciones funcional pero **sin origen automático de mensajes todavía** — se cargan manualmente o por nota de llamada, y el flujo de aprobar/editar/descartar ya queda operativo y auditado en `approvals`.
- Flujo de alta de una nueva organización ya operativo (aunque todavía no se invite a nadie), para no tener que rehacer arquitectura cuando llegue la Fase 2.
- Error tracking (Sentry u equivalente) integrado.
- Banner/badge visible de "Demo" en la UI cuando el usuario está dentro de la organización `is_demo`, y el script de seed actualizado para poblar todas las pantallas nuevas de esta versión (antes solo tenía que sostener una app vacía).

**Criterio de salida:** el equipo fundador reemplaza su operación manual actual por el sistema durante el uso diario real, sin depender de planillas ni de WhatsApp Web como fuente de verdad del seguimiento.

**Fuera de alcance:** WhatsApp/Instagram, IA de clasificación o redacción, matching automático, cobros.

## V0.2 — Canales y beta cerrada (implementa la Fase 2 de producto)

**Objetivo:** los mensajes entran solos al sistema, y los primeros agentes externos empiezan a usarlo.

Entregables:

- Integración oficial de WhatsApp Business Cloud API: mensajes entrantes se registran como `conversations`/`messages` vinculados a un contacto (existente o nuevo).
- Integración de Instagram profesional vía Meta, sobre la misma bandeja.
- Gestión de "en gestión por…" para evitar respuestas duplicadas.
- Sincronización con Google Calendar para visitas y recordatorios compartidos.
- Flujo de invitación de una organización beta (self-serve o asistido, a definir según cuántas se sumen).
- n8n se despliega recién acá si hace falta como capa de automatización entre Meta/Calendar y la aplicación — no antes (ver [ADR-0009](adr/0009-orquestacion-ia-sin-n8n-v0.md)).
- **Bloqueante antes de invitar a la primera organización externa:** Términos de Servicio, Política de Privacidad y resolución de la consulta legal de protección de datos (`docs/producto.md` sección 17).

**Criterio de salida:** al menos una organización externa a la del equipo fundador está operando en el sistema con datos reales, con mensajes de WhatsApp/Instagram entrando automáticamente.

## V1.0 — IA activa (implementa la Fase 3 de producto)

**Objetivo:** la IA hace el trabajo pesado de clasificación y redacción; las personas siguen aprobando todo.

Entregables:

- Asistente de Recepción: clasificación automática de intención, zona, presupuesto y urgencia sobre mensajes entrantes.
- Asistente de Redacción: borrador de respuesta sugerido en la bandeja de aprobaciones.
- Asistente de Seguimiento: detección de contactos sin respuesta y propuesta de próxima acción/fecha.
- Matching básico entre `contact_requirements` y `properties`.
- Resumen diario/semanal automatizado y reportes iniciales (velocidad de respuesta, visitas, oportunidades activas).
- Todas las sugerencias de IA trazables a la actividad que las originó, vía `approvals` (regla no negociable de `docs/producto.md` sección 7 — no es negociable en esta versión tampoco).

**Criterio de salida:** un porcentaje relevante de las respuestas y próximas acciones del día a día se originan como sugerencia de IA aprobada por una persona, no escritas desde cero.

## V1.x — Monetización y escalamiento (implementa la Fase 4 de producto)

**Objetivo:** el producto cobra de verdad y puede sumar organizaciones sin trabajo manual de onboarding.

Entregables:

- Integración de cobro recurrente (pasarela a definir — `docs/producto.md` sección 13/17, pendiente de definición financiera/legal).
- Activación real de la tabla `subscriptions` (planes, límites de uso, estado de pago).
- Onboarding self-service o asistido para nuevas organizaciones.
- Dominio propio de DOOH.
- Métricas de uso por organización expuestas a DOOH (no solo al cliente).

**Criterio de entrada a esta versión:** los criterios de validación comercial de `docs/producto.md` sección 14 (al menos 3 de 5 agentes beta usando el sistema semanalmente y dispuestos a pagar) — no se arranca V1.x solo porque V1.0 esté técnicamente lista.

## Resumen

| Versión técnica | Fase de producto | Foco |
|---|---|---|
| V0 | Fase 0 | Base técnica: repo, datos, entornos, CI/CD |
| V0.1 | Fase 1 | Operación manual completa, sin IA ni canales |
| V0.2 | Fase 2 | Canales conectados + primeras organizaciones beta |
| V1.0 | Fase 3 | IA de clasificación, redacción, seguimiento y matching |
| V1.x | Fase 4 | Cobro real y escalamiento |

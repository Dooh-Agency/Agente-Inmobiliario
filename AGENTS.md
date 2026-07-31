# Agente Inmobiliario — instrucciones para agentes de IA

Este archivo es la fuente canónica de contexto para cualquier asistente de IA que trabaje en este repositorio (Claude Code, Codex CLI, Cursor, u otro). Si vas a tocar código o documentación acá, leé esto primero.

## Qué es este proyecto

Un copiloto operativo SaaS para agentes inmobiliarios, desarrollado y operado por **DOOH** (agencia). Unifica contactos, conversaciones, seguimientos, visitas y oportunidades en un solo lugar. La IA clasifica, prioriza y sugiere; las personas aprueban y deciden — nunca reemplaza al agente ni actúa de forma autónoma frente al cliente.

El documento de producto completo, con alcance funcional, arquitectura, roadmap y modelo de negocio, vive en [`docs/producto.md`](docs/producto.md). **Es la fuente de verdad** — cualquier decisión de diseño o alcance debe ser consistente con ese documento. Si algo que se pide contradice lo que dice ese documento, señalalo antes de implementarlo en vez de asumir que el documento quedó desactualizado.

También existe [`docs/producto.html`](docs/producto.html): una versión renderizada del mismo contenido (con identidad visual DOOH y tabla de contenidos navegable) pensada para compartir o leer fuera de un editor de markdown. Es una copia de lectura, no la fuente de verdad — cuando `docs/producto.md` cambie, `docs/producto.html` debe regenerarse para no quedar desactualizado.

## Estado actual

Etapa de planificación / pre-código. Todavía no hay aplicación construida. El roadmap (sección 12 de `docs/producto.md`) define las fases: Fase 0 (descubrimiento y diseño) → Fase 1 (centro de control funcional) → Fase 2 (canales y beta) → Fase 3 (IA y automatizaciones) → Fase 4 (monetización). No saltear fases ni construir funcionalidad de una fase posterior sin que la anterior esté resuelta, salvo que el usuario lo pida explícitamente.

## Reglas de producto no negociables

Estas reglas (sección 7 de `docs/producto.md`) aplican a cualquier código o flujo que se construya, no son solo aspiracionales:

- La IA **nunca** envía mensajes de forma autónoma al cliente final. Todo mensaje saliente pasa por aprobación humana.
- La IA **nunca** confirma precio final, disponibilidad definitiva, condiciones legales, reserva o documentación.
- Toda sugerencia de IA debe quedar ligada al contacto y a la actividad que la originó (trazabilidad).
- Cada mensaje enviado y cada respuesta recibida se registra en el historial.
- Las acciones de aprobación, edición y descarte quedan auditadas.
- Aislamiento por organización (`organizations`) desde la Fase 1 — este producto es multi-tenant desde el primer commit, no una app de un solo usuario a la que se le agrega multi-tenancy después.

## Decisiones de arquitectura vigentes

- **Frontend:** Next.js.
- **Hosting:** Netlify (plan free durante desarrollo/piloto).
- **Base de datos y auth:** Supabase / PostgreSQL, con Row Level Security para el aislamiento por organización.
- **Automatizaciones:** n8n autoalojado.
- **IA:** API de OpenAI.
- **Canales:** Meta Cloud API (WhatsApp Business, Instagram profesional) — integraciones oficiales únicamente, nunca scraping o APIs no oficiales.
- **Agenda:** Google Calendar.
- **Facturación:** a definir (candidatos: MercadoPago, Paddle, Lemon Squeezy — Stripe no opera de forma directa en Argentina).

Entidades mínimas del modelo de datos: `organizations`, `users`, `subscriptions`, `contacts`, `contact_requirements`, `properties`, `conversations`, `messages`, `activities`, `tasks`, `appointments`, `matches`, `approvals`. Ver sección 9 de `docs/producto.md` para el detalle.

## Pendientes que todavía no están resueltos

No asumas que estos puntos ya están decididos — confirmá con el usuario antes de construir sobre ellos:

- Nombre definitivo del producto (hoy usa el nombre de trabajo "Copiloto Inmobiliario").
- Términos de Servicio y Política de Privacidad — **bloqueante antes de sumar el primer agente beta con contactos reales de terceros** (ver sección 17 de `docs/producto.md`, Ley 25.326 de Protección de Datos Personales).
- Mecanismo de cobro/facturación concreto.
- Estructura legal/impositiva de DOOH para facturar el SaaS.

## Convenciones de trabajo

- Los documentos de producto y negocio se escriben en español (Argentina). El código y sus comentarios, en inglés, salvo que el usuario indique lo contrario.
- Este repo pertenece a `Dooh-Agency/Agente-Inmobiliario` en GitHub, rama principal `main`.
- No se necesita dominio propio ni cuentas comerciales activadas mientras el proyecto esté en etapa de piloto cerrado (equipo fundador, luego agentes beta conocidos).
- Antes de tomar decisiones de alcance, arquitectura o negocio no cubiertas explícitamente en `docs/producto.md`, preguntar en vez de asumir — es un producto real con validación comercial en curso, no un proyecto de práctica.

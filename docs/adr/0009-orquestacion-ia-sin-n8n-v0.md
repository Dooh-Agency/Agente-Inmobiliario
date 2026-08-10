# ADR-0009: Orquestación de IA — llamadas directas a OpenAI; n8n diferido

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

`docs/producto.md` sección 9 incluye n8n autoalojado en el stack, pensado como capa de automatización entre canales (WhatsApp/Instagram/Calendar), la aplicación y la IA. En la conversación de arranque del proyecto se decidió explícitamente posponer n8n hasta que haga falta, en vez de desplegarlo en la V0: en la Fase 1 (`docs/roadmap-tecnico.md`, V0.1) no hay canales conectados todavía, así que no hay nada que n8n tenga que orquestar.

## Decisión

En la V0 y V0.1, la aplicación llama directamente a la API de OpenAI desde Server Actions/Route Handlers (function calling / structured outputs para clasificación y borradores), sin ninguna capa de orquestación intermedia. n8n se despliega recién en la V0.2 (Fase 2 de producto), cuando haya integraciones de canal reales que automatizar.

## Alternativas consideradas

- **Desplegar n8n desde la V0 "por las dudas":** suma un servicio más para operar (hosting, actualizaciones, monitoreo) sin ningún flujo real que justifique su existencia todavía. Contradice el criterio de no construir infraestructura antes de necesitarla.
- **Reemplazar n8n por completo con lógica en Next.js:** válido a mediano plazo si los flujos de automatización terminan siendo simples, pero prematuro decidirlo ahora — se reevalúa cuando la Fase 2 defina qué automatizaciones hacen falta en concreto.

## Consecuencias

- La V0/V0.1 tiene una superficie de infraestructura más chica: solo Next.js/Netlify + Supabase + API de OpenAI.
- Cuando llegue la V0.2, hay que decidir dónde vive n8n (managed como Railway/Render, o VPS propio) — decisión explícitamente pendiente, a resolver en ese momento con el contexto real de qué automatizaciones hacen falta.
- Este ADR queda reemplazado por uno nuevo en el momento en que n8n efectivamente se despliegue, documentando esa decisión de hosting.

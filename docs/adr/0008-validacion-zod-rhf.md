# ADR-0008: Validación de datos — Zod + React Hook Form

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

Con Server Actions como única puerta de entrada a los datos ([ADR-0004](0004-server-actions-sin-backend-separado.md)), hace falta una forma consistente de validar tanto el input de formularios en el cliente como el input real que llega al servidor (que nunca debe asumirse confiable solo porque el formulario ya validó).

## Decisión

Zod para definir schemas de validación, reutilizados tanto en el cliente (formularios) como en el servidor (primera línea de cada Server Action). React Hook Form para el manejo de estado de formularios en el cliente, integrado con los mismos schemas de Zod vía `@hookform/resolvers`.

## Alternativas consideradas

- **Validar solo en el cliente:** inaceptable — cualquier Server Action es alcanzable directamente sin pasar por el formulario, y `docs/producto.md` exige trazabilidad y datos consistentes; no se puede confiar en que el input ya viene validado.
- **Yup en vez de Zod:** alternativa razonable y comparable en madurez. Se elige Zod por su integración más natural con TypeScript (infiere tipos estáticos directamente del schema, sin declarar el tipo por separado), lo cual importa dado el tipado estricto de [ADR-0001](0001-framework-nextjs-typescript.md).
- **Formik en vez de React Hook Form:** React Hook Form tiene mejor rendimiento (menos re-renders) y una integración más directa con Zod vía resolvers oficiales.

## Consecuencias

- Un mismo schema de Zod define la forma de un dato en el formulario y en la Server Action correspondiente — evita que cliente y servidor validen cosas distintas sin darse cuenta.
- Los tipos de TypeScript de cada formulario se derivan del schema de Zod (`z.infer<typeof schema>`), reduciendo duplicación entre validación y tipado.
- Cambios en las reglas de negocio de un formulario (por ejemplo, qué campos son obligatorios al crear un contacto) se hacen en un solo lugar: el schema.

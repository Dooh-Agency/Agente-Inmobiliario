# ADR-0007: UI y estilos — Tailwind CSS + shadcn/ui

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

`docs/producto.md` sección 10 pide una interfaz "muy simple", parecida a una bandeja de oportunidades, responsive, con identidad visual DOOH en las pantallas de cara al cliente. Hace falta un sistema de componentes que permita construir pantallas CRUD-pesadas (contactos, propiedades, tareas) rápido y de forma consistente, dado un solo desarrollador con agentes de IA.

## Decisión

Tailwind CSS como sistema de estilos, y shadcn/ui como base de componentes (botones, formularios, tablas, diálogos, tarjetas) copiados al repositorio y adaptados a la identidad DOOH (paleta lima/negro/off-white, tipografía Nunito Sans — ver skill de marca DOOH), en vez de construidos desde cero.

## Alternativas consideradas

- **CSS Modules / CSS plano:** más control, pero mucho más lento para construir la cantidad de pantallas CRUD que pide el MVP (sección 5), y más fácil que la consistencia visual se degrade entre pantallas escritas en sesiones distintas.
- **Librería de componentes cerrada (MUI, Ant Design, Chakra):** vienen con su propio lenguaje visual fuerte, que hay que pelear para adaptar a la identidad DOOH. shadcn/ui, al copiar el código del componente al repo en vez de importarlo como dependencia opaca, es más fácil de reestilizar a fondo sin luchar contra la librería.
- **Diseñar un design system propio desde cero:** el nivel de inversión correcto para un producto con equipo de diseño dedicado; no para un MVP construido por un desarrollador solo.

## Consecuencias

- Los componentes de shadcn/ui viven en el repositorio (no son una dependencia de node_modules que actualizar), lo que da control total para aplicar la identidad DOOH pero también responsabilidad de mantenerlos.
- Tailwind exige disciplina para no terminar con clases repetidas por todos lados; se resuelve extrayendo patrones repetidos a componentes, no a nivel de configuración de Tailwind.
- La paleta y tipografía de Tailwind se configuran una sola vez (`tailwind.config`) siguiendo el sistema de marca DOOH, para que ningún componente nuevo introduzca colores o fuentes fuera de sistema.

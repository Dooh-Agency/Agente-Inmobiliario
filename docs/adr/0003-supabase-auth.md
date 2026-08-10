# ADR-0003: Autenticación — Supabase Auth

**Estado:** Aceptada
**Fecha:** 2026-08-10

## Contexto

`docs/producto.md` sección 11 exige que cada persona acceda con su propio usuario (sin contraseñas compartidas), autenticación en dos pasos documentada, y permisos por usuario. Hace falta decidir con qué sistema se resuelve esto, atado a la elección de base de datos de [ADR-0002](0002-supabase-backend.md).

## Decisión

Supabase Auth, con login por email/contraseña como método principal y magic link como alternativa. La sesión se valida en el servidor (Server Actions/Route Handlers) usando el JWT de Supabase, y las policies de RLS usan `auth.uid()` para resolver a qué organización pertenece cada usuario.

## Alternativas consideradas

- **Auth0 / Clerk:** proveedores de auth dedicados, con más funcionalidad de out-of-the-box (SSO empresarial, gestión de organizaciones nativa). Se descartan por ahora: suman un servicio externo más y un costo mensual adicional que no se justifica en la etapa de piloto cerrado, cuando Supabase Auth ya cubre lo que pide `docs/producto.md`. Si el producto escala a equipos grandes con requerimientos de SSO corporativo, vale la pena reevaluar.
- **Auth propio (tablas de usuarios + hashing manual):** más trabajo de mantenimiento y superficie de riesgo de seguridad (manejo de contraseñas, tokens de recuperación) sin ningún beneficio frente a usar Supabase Auth, que ya está integrado con RLS.

## Consecuencias

- La tabla `users` de `docs/producto.md` se relaciona 1 a 1 con `auth.users` de Supabase (perfil extendido: rol, organización, nombre visible), no la reemplaza.
- La autenticación en dos pasos (2FA) que pide `docs/producto.md` sección 11 se implementa con el soporte nativo de Supabase Auth cuando se habilite — pendiente de activar antes de sumar el primer usuario externo al piloto, no bloqueante para la V0.
- Cambiar de proveedor de auth más adelante implicaría migrar usuarios y revisar todas las policies de RLS que dependen de `auth.uid()` — costo de reversión medio-alto, asumido conscientemente.

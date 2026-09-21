# Web Sicilia

Web profesional para **Sicilia**, productor musical, con tres pilares:

1. **Producciones** — embed oficial de Spotify + botón "Abrir en Spotify".
2. **Catálogo y venta de beats** — beat store con licencias (Basic/Premium/Exclusive) y compra vía Mercado Pago (preferencia dinámica por orden).
3. **Servicios y presentación profesional** — captación de clientes para producción/mezcla/mastering.

La especificación completa del producto y las reglas de negocio están en [`CLAUDE.md`](./CLAUDE.md) — es la única fuente de verdad del proyecto. El detalle de producto original está en [`Prompt_Senior___Desarrollo_Web_de_Sicilia.md`](./Prompt_Senior___Desarrollo_Web_de_Sicilia.md).

## Stack

Next.js 15 (App Router, TypeScript strict) · Payload CMS (embebido) · PostgreSQL · Cloudflare R2 · Zustand + Howler.js · React Hook Form + Zod · Tailwind CSS + shadcn/ui · Mercado Pago (Checkout Pro) · Vercel.

Ver `CLAUDE.md` sección 2 para el detalle y la justificación de cada elección.

## Estado del proyecto

El scaffold de código (Next.js + Payload, Fase 0 del plan de fases en `CLAUDE.md` sección 14) todavía no arrancó. Este repo por ahora solo tiene la especificación, el pipeline de CI y la estructura de control de versiones lista.

## Cómo levantar el proyecto

Se documenta acá una vez completada la Fase 0. Por ahora, copiar `.env.example` a `.env.local` y completar las variables reales cuando el scaffold exista.

## CI/CD

- **GitHub Actions** (`.github/workflows/ci.yml`) corre lint, typecheck y build en cada push/PR contra `main`. Es un gate de calidad, no deployea nada.
- **Deploy** vía integración nativa de Vercel con GitHub: previews automáticos por PR, producción en cada push a `main`.

## Decisiones de arquitectura

Ver [`DECISIONS.md`](./DECISIONS.md) para el registro de decisiones tomadas ante ambigüedades no cubiertas explícitamente por `CLAUDE.md` (sección 0.4).

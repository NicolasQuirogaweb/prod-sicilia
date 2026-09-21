# Web Sicilia

Web profesional para **Sicilia**, productor musical, con tres pilares:

1. **Producciones** — embed oficial de Spotify + botón "Abrir en Spotify".
2. **Catálogo y venta de beats** — beat store con licencias (Basic/Premium/Exclusive) y compra vía Mercado Pago (preferencia dinámica por orden).
3. **Servicios y presentación profesional** — captación de clientes para producción/mezcla/mastering.

La especificación completa del producto y las reglas de negocio están en [`CLAUDE.md`](./CLAUDE.md) — es la única fuente de verdad del proyecto. El detalle de producto original está en [`Prompt_Senior___Desarrollo_Web_de_Sicilia.md`](./Prompt_Senior___Desarrollo_Web_de_Sicilia.md).

## Stack

Next.js 16 (App Router, TypeScript strict) · Payload CMS 3 (embebido) · PostgreSQL · Cloudflare R2 · Zustand + Howler.js · React Hook Form + Zod · Tailwind CSS + shadcn/ui · Mercado Pago (Checkout Pro) · Vercel.

Ver `CLAUDE.md` sección 2 para el detalle de cada elección, y [`DECISIONS.md`](./DECISIONS.md) para por qué el proyecto corre en Next.js 16 en vez de Next.js 15 (la sección 2 fija Next 15, pero Payload solo soporta un parche de Next 15 con una vulnerabilidad crítica sin fix disponible en esa línea).

## Estado del proyecto

**Fase 0 completa**: scaffold de Next.js + Payload CMS embebido, collections mínimas `Users` (auth) y `Media` (backed por R2), estructura de carpetas `(site)`/`(payload)`. Falta conectar una base de datos y un bucket R2 reales (ver más abajo) y arrancar la Fase 1 (modelo de datos de negocio: `Beat`, `License`, `Order`).

## Cómo levantar el proyecto

1. `npm install`
2. Copiar `.env.example` a `.env.local` (o completar el que ya existe) con:
   - `DATABASE_URL` de un proyecto en [Neon](https://neon.tech)
   - `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET_NAME`, `R2_ENDPOINT` de un bucket de Cloudflare R2, y `R2_PUBLIC_URL` (subdominio `R2.dev` o dominio custom habilitado en el bucket — R2 es privado por default, ver `DECISIONS.md`)
   - `PAYLOAD_SECRET` (cualquier string aleatorio largo)
3. `npm run dev` y abrir `http://localhost:3000/admin` para crear el primer usuario.

## CI/CD

- **GitHub Actions** (`.github/workflows/ci.yml`) corre lint, typecheck y build en cada push/PR contra `main`. Es un gate de calidad, no deployea nada.
- **Deploy** vía integración nativa de Vercel con GitHub: previews automáticos por PR, producción en cada push a `main`.

## Decisiones de arquitectura

Ver [`DECISIONS.md`](./DECISIONS.md) para el registro de decisiones tomadas ante ambigüedades no cubiertas explícitamente por `CLAUDE.md` (sección 0.4).

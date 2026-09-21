# DECISIONS.md

Registro de decisiones tomadas ante ambigüedades no cubiertas explícitamente por `CLAUDE.md`, según exige su sección 0.4.

---

## 2026-09-20 — CI/CD como gate de calidad, no como mecanismo de deploy

**Decisión:** GitHub Actions (`.github/workflows/ci.yml`) solo corre lint, typecheck y build en cada push/PR contra `main`. No hace deploy.

**Por qué:** el deploy se resuelve con la integración nativa de Vercel↔GitHub (previews por PR, producción en push a `main`), que ya cubre lo que pide `CLAUDE.md` sección 2 ("Deploy: Vercel — Integración nativa con Next.js"). Duplicar esa lógica en Actions agregaría complejidad (gestionar `VERCEL_TOKEN`/`ORG_ID`/`PROJECT_ID` como secrets) sin ningún beneficio real.

---

## 2026-09-20 — Creación del repo remoto en GitHub es manual

**Decisión:** el repositorio en GitHub se crea manualmente desde github.com, no vía comando.

**Por qué:** el entorno de desarrollo no tiene `gh` CLI instalado ni acceso autenticado al MCP de GitHub. El repo local queda listo con el primer commit; el usuario solo necesita crear el repo vacío remoto y correr `git remote add` + `git push`.

---

## 2026-09-20 — Pipeline de CI tolera la ausencia de `package.json`

**Decisión:** el workflow de CI detecta si existe `package.json` antes de correr `npm ci`/lint/typecheck/build. Si no existe, termina en verde con un mensaje informativo en vez de fallar.

**Por qué:** el repo se inicializa antes de la Fase 0 (scaffold de Next.js/Payload) para tener el flujo de Git y CI listo desde el primer commit. Sin este chequeo, cada push previo a la Fase 0 mostraría un check en rojo falso, sin ningún valor diagnóstico real.

---

## 2026-09-21 — Stack actualizado de Next.js 15 a Next.js 16.3.5 (reemplaza la decisión anterior del 2026-09-20)

**Decisión:** `next` queda fijo en `16.3.5` (última publicada de la línea 16.3.x), no en Next.js 15 como fija literalmente la sección 2 del `CLAUDE.md`. `eslint-config-next` se pinea a la misma versión.

**Por qué:** al scaffoldear el proyecto se pineó primero `next@15.4.11`, el único parche de Next 15 compatible con el peer dependency de Payload 3.90.1 (rangos soportados: `>=15.2.9 <15.3.0`, `>=15.3.9 <15.4.0`, `>=15.4.11 <15.5.0`, o `>=16.3.3 <17.0.0`). `npm audit` detectó que esa versión exacta carga una vulnerabilidad **crítica** heredada de Next.js (incluye RCE no autenticado en servidores hosteados en Windows y RCE no autenticado en la API de optimización de imágenes con AVIF) más varias altas. La corrección real de Next está publicada en `15.5.25`, pero esa versión cae fuera del rango que acepta Payload (`<15.5.0`) — no existe ningún parche de Next 15 que sea simultáneamente seguro y compatible con Payload en este momento. Next 16.3.3+ sí es compatible con Payload y queda fuera del rango vulnerable reportado por el advisory (que termina en `16.3.0-preview.10`, un prerelease anterior al 16.3.0 estable). Decisión confirmada con el usuario dado que el proyecto procesa datos de clientes (nombre, email, Instagram) y pagos vía Mercado Pago — no se justifica quedarse en una versión con RCE conocido y sin parche disponible en esa línea. Verificado contra el registro de npm en vivo (`npm view`, `npm audit --json`), no contra documentación desactualizada.

---

## 2026-09-20 — Storage adapter: `@payloadcms/storage-s3` apuntando a R2, no `@payloadcms/storage-r2`

**Decisión:** el plugin de storage usado es `@payloadcms/storage-s3` configurado contra el endpoint S3-compatible de Cloudflare R2 (`region: 'auto'`, `forcePathStyle: true`).

**Por qué:** `@payloadcms/storage-r2` es exclusivo para Cloudflare Workers con binding nativo de bucket — no funciona en Vercel/Node, que es donde corre este proyecto. La documentación oficial de Payload recomienda explícitamente `storage-s3` contra el endpoint de R2 para cualquier entorno Node.js/Vercel/Netlify.

---

## 2026-09-20 — Nueva variable de entorno `R2_PUBLIC_URL`

**Decisión:** se agrega `R2_PUBLIC_URL` a `.env.example` y `.env.local`, además de las variables ya listadas en la sección 13 del `CLAUDE.md`.

**Por qué:** los buckets de R2 son privados por default. El `R2_ENDPOINT` (API S3) sirve solo para subir archivos, no para servirlos públicamente — hace falta una URL pública separada (subdominio `R2.dev` o dominio custom conectado en el dashboard de Cloudflare) para que `generateFileURL` arme URLs de covers/previews accesibles desde el navegador. El `CLAUDE.md` no anticipaba esta distinción técnica.

---

## 2026-09-20 — Collections `Users` y `Media` se crean en Fase 0, no en Fase 1

**Decisión:** se agrega una collection `Users` (no está en la sección 4 del `CLAUDE.md`) con `auth: true` y un campo `role` (`admin`/`editor`), y se adelanta la creación de `Media` (sí está en la sección 4) a esta fase.

**Por qué:** Payload exige al menos una collection con `auth: true` para poder loguearse al panel admin — es infraestructura mínima no negociable, y `Users` con roles cumple además el checklist de seguridad de la sección 9 ("solo Sicilia y vos tienen acceso de escritura"). `Media` se adelanta porque Fase 0 incluye explícitamente "configuración de R2", y sin una collection de upload no hay forma de verificar que el storage funciona. `Beat`, `License` y `Order` (modelo de negocio real) quedan intactos para Fase 1, tal como dice la sección 14.

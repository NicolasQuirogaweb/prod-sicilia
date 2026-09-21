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

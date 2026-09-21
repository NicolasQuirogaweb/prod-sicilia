# CLAUDE.md — Web Sicilia
### Especificación técnica completa para desarrollo agéntico (Claude Code / VS Code)

> Este documento es la **única fuente de verdad** para construir el proyecto. Está pensado para que un agente autónomo (Claude Code) trabaje sin necesidad de contexto adicional. Leelo completo antes de escribir la primera línea de código.

---

## 0. Rol y modo de trabajo del agente

Actuás como un **ingeniero de software senior full-stack**, responsable de construir este producto de punta a punta con estándares de producción.

Reglas de trabajo:

1. **Trabajá por fases** (ver sección 14). Cada fase debe terminar en un estado funcional, compilable y verificable — nunca dejes el repo en un estado roto entre fases.
2. **Mantené un TODO checklist activo** durante toda la sesión (usando la herramienta de tareas si está disponible) y marcá el progreso en tiempo real.
3. **No expandas el alcance.** Todo lo que no esté explícitamente en la sección 3 (Alcance de V1) NO se implementa, aunque el modelo de datos esté preparado para soportarlo después (sección 15).
4. **No asumas silenciosamente.** Si hay una ambigüedad real no cubierta en este documento, tomá la decisión más simple y consistente con el resto de la arquitectura, y dejala registrada en un archivo `DECISIONS.md` en la raíz del repo con una línea explicando el porqué.
5. **Ninguna feature se da por "hecha" sin:** validación de inputs, manejo de errores, estados de carga, y revisión del checklist de seguridad correspondiente (sección 9).
6. **Commits atómicos**, siguiendo Conventional Commits (`feat:`, `fix:`, `chore:`, `refactor:`, etc.), uno por unidad de trabajo coherente.
7. Al finalizar cada fase, corré `build`, `lint` y `typecheck` y no avances a la siguiente fase si alguno falla.

---

## 1. Contexto y objetivo del proyecto

Web profesional para **Sicilia**, productor musical, con tres pilares:

1. **Producciones** — mostrar su música vía embed de Spotify.
2. **Catálogo y venta de beats** — beat store con licencias y compra vía Mercado Pago.
3. **Servicios y presentación profesional** — captar clientes para producción/mezcla/mastering.

El detalle funcional completo, palabra por palabra como fue definido, está en el Anexo A (sección 17) de este documento — **es la especificación de producto y tiene prioridad sobre cualquier interpretación libre.**

Referencia visual **solo de experiencia de navegación** del catálogo (no copiar diseño/código/identidad): `https://niv0web.vercel.app`

---

## 2. Stack tecnológico (definitivo — no cambiar sin justificación registrada en DECISIONS.md)

| Capa | Tecnología | Motivo |
|---|---|---|
| Framework | **Next.js 15 (App Router)** + **TypeScript strict** | SSR/SEO real, server components, routing por beat |
| CMS del catálogo | **Payload CMS** (self-hosted, corre embebido en el mismo proyecto Next.js) | Sicilia edita beats/licencias/precios sin tocar código, con auth y roles incluidos |
| Base de datos | **PostgreSQL** (Neon o Supabase, vía adaptador de Payload) | Relacional, ideal para Beat/License/Order |
| Storage de archivos | **Cloudflare R2** (S3-compatible) | Sin costo de egress — crítico porque se sirven previews de audio constantemente |
| Estado global del reproductor | **Zustand** | Store simple para track activo, progreso, volumen |
| Motor de audio | **Howler.js** | Maneja compatibilidad cross-browser mejor que `<audio>` pelado |
| Formularios | **React Hook Form + Zod** | Validación tipada end-to-end |
| Estilos | **Tailwind CSS + shadcn/ui** | Velocidad de desarrollo con buena calidad visual base |
| Pagos | **Mercado Pago API (Checkout Pro)** — preferencia dinámica por orden, NO links estáticos | Ver sección 7 |
| Deploy | **Vercel** | Integración nativa con Next.js, ISR, image optimization |
| Emails (preparado, no activo en v1) | **Resend + React Email** | Para cuando se automatice la confirmación de compra |

---

## 3. Alcance de V1 (obligatorio — no expandir)

### Incluido

- Home: hero (SICILIA / Music Producer / descripción + 3 CTAs), sección Producciones (embed Spotify + botón "Abrir en Spotify"), sección Servicios (5 tarjetas + CTA), sección Sobre mí, sección Contacto (redes + CTA).
- Catálogo de beats: listado con artwork, nombre, género, BPM, tonalidad; página individual por beat (`/beats/[slug]`).
- Reproductor global de previews: un solo beat sonando a la vez, play/pause, progreso, duración, volumen, indicador visual del activo, persiste entre navegaciones.
- Flujo de compra: selección de licencia (sin preselección) → formulario con beat/licencia auto-completados (read-only) → guardado de orden en DB con estado `PENDING_PAYMENT` → generación de preferencia de pago dinámica → redirección a Mercado Pago.
- Webhook de Mercado Pago que actualiza automáticamente el estado de la orden a `PAID` (o `CANCELLED`/`REJECTED` según corresponda) — **sin intervención manual**.
- Panel de Payload CMS funcionando: Sicilia puede crear/editar beats (cover, preview de audio, género, BPM, key, mood, status) y editar precios/features de licencias, sin tocar código.
- SEO básico: metadata por página, sitemap.xml, robots.txt, Open Graph, JSON-LD tipo `Product` por beat.
- Responsive completo (desktop/tablet/mobile), con atención especial al catálogo, reproductor, selección de licencia y formulario.

### Explícitamente EXCLUIDO de v1

- Descarga automática del archivo final al cliente.
- Emails automáticos de confirmación.
- Cuentas de clientes / login de usuarios finales.
- Cupones, packs de beats, promociones.
- Dashboard de analytics.
- Generación automática de licencias en PDF.
- Cualquier feature no listada arriba, sin importar cuán simple parezca agregarla.

---

## 4. Modelo de datos (Collections de Payload)

```ts
Beat {
  id: string (uuid)
  title: string
  slug: string (unique, generado desde title)
  cover: relation -> Media
  previewAudio: relation -> Media       // archivo de preview, liviano, público
  finalFile: relation -> Media          // archivo final, NUNCA expuesto por API pública
  genre: string
  bpm: number
  key: string
  mood: string[]
  status: enum('draft', 'published', 'sold_exclusive')
  createdAt, updatedAt
}

License {
  id: string (uuid)
  type: enum('BASIC', 'PREMIUM', 'EXCLUSIVE')
  name: string
  price: number
  currency: string (default 'ARS' o la que corresponda)
  features: string[]
  description: string
}

Order {
  id: string (uuid)
  beat: relation -> Beat
  license: relation -> License
  customerName: string (required)
  email: string (required, validado)
  instagram: string (optional)
  status: enum('PENDING_PAYMENT', 'PAID', 'CANCELLED', 'REJECTED')
  mpPreferenceId: string        // id de la preferencia creada en Mercado Pago
  mpPaymentId: string | null    // se completa cuando llega el webhook
  externalReference: string     // = Order.id, usado para cruzar con el webhook
  createdAt, updatedAt
}

Media {
  // collection nativa de Payload, backed por Cloudflare R2
  // usada tanto para covers, previews como archivos finales
}
```

Reglas de integridad no negociables:

- `Beat.finalFile` **jamás** se resuelve en ningún endpoint o response accesible desde el cliente público, bajo ninguna circunstancia, ni siquiera si la orden está `PAID` (queda preparado para descarga automática en fase futura, pero v1 no la implementa — ver sección 15).
- Toda mutación de `Order.status` a `PAID` ocurre **únicamente** desde el handler del webhook de Mercado Pago, validando la firma (sección 7 y 9). Ningún endpoint público debe permitir setear ese campo directamente.

---

## 5. Arquitectura de carpetas (referencia)

```
/src
  /app
    /(site)
      page.tsx                    -> Home
      /beats
        page.tsx                  -> Catálogo
        /[slug]/page.tsx          -> Beat individual
      /checkout
        /[orderId]/page.tsx       -> Confirmación / estado post-redirección
    /api
      /checkout/create-preference/route.ts
      /webhooks/mercadopago/route.ts
      /orders/route.ts            -> creación de orden (POST, server action o route handler)
    /(payload)
      /admin                      -> panel de Payload CMS
  /collections
    Beats.ts
    Licenses.ts
    Orders.ts
    Media.ts
  /components
    /player                       -> GlobalPlayer, PlayerBar, TrackProgress
    /beats                        -> BeatCard, BeatGrid, LicenseSelector
    /checkout                     -> PurchaseForm
    /sections                     -> Hero, Producciones, Servicios, SobreMi, Contacto
    /ui                           -> shadcn components
  /store
    playerStore.ts                -> Zustand store del reproductor global
  /lib
    mercadopago.ts                -> cliente y helpers de MP
    validations.ts                -> schemas Zod (formulario, orden)
    r2.ts                         -> helpers de storage
  /payload.config.ts
DECISIONS.md
CLAUDE.md                          (este archivo)
README.md
```

---

## 6. Flujo de compra (paso a paso, exacto)

```
CATÁLOGO
  → SELECCIONAR BEAT
  → ELEGIR LICENCIA (sin preselección — el usuario debe elegir explícitamente)
  → FORMULARIO
      - Nombre completo (obligatorio)
      - Email (obligatorio, validado)
      - Instagram (opcional)
      - Beat (automático, read-only)
      - Licencia (automática, read-only)
  → "CONTINUAR AL PAGO"
      1. Se crea el Order en DB con status PENDING_PAYMENT
      2. Se genera una preferencia en Mercado Pago vía API con
         external_reference = order.id y el precio de la licencia elegida
      3. Se redirige al usuario al init_point devuelto por Mercado Pago
  → MERCADO PAGO (pago del usuario)
  → WEBHOOK
      1. Mercado Pago notifica a /api/webhooks/mercadopago
      2. Se valida la firma del request (obligatorio, ver sección 9)
      3. Se busca la orden por external_reference
      4. Se actualiza status según el resultado (PAID / REJECTED / CANCELLED)
  → Página de confirmación (/checkout/[orderId]) muestra el estado actual de la orden
```

---

## 7. Integración de Mercado Pago (detallado)

**Importante:** el prompt de producto original pedía 3 links de pago estáticos. Esa decisión fue revisada y reemplazada por lo siguiente, ya acordado — **usar esta versión, no la de links estáticos:**

- `POST /api/checkout/create-preference`
  - Recibe `orderId`.
  - Llama a la API de Mercado Pago (Checkout Pro / Preferences) generando una preferencia con: título del beat + licencia, precio de la licencia (fijo, definido en la collection `License`), y `external_reference: order.id`.
  - Guarda `mpPreferenceId` en la orden.
  - Devuelve el `init_point` para redirigir al usuario.

- `POST /api/webhooks/mercadopago`
  - **Verifica la firma del webhook** (header `x-signature` + `x-request-id`, según la documentación oficial de Mercado Pago) antes de procesar nada. Si la firma no es válida, responder 401 y no tocar la DB.
  - Es **idempotente**: si llega la misma notificación más de una vez (Mercado Pago puede reintentar), no debe duplicar efectos ni romper si la orden ya está en el estado final.
  - Busca la orden por `external_reference`, consulta el estado real del pago contra la API de Mercado Pago (no confiar ciegamente en el payload del webhook), y actualiza `Order.status` y `Order.mpPaymentId` acorde.
  - Responde 200 rápido (Mercado Pago espera confirmación breve; cualquier procesamiento pesado se hace async).

- Todas las credenciales (`MP_ACCESS_TOKEN`, `MP_WEBHOOK_SECRET`) van por variables de entorno, nunca hardcodeadas.

---

## 8. Reproductor de audio global

- Un único elemento de audio (Howler.js) vive en el **root layout**, controlado por un store de Zustand (`playerStore.ts`): `currentTrack`, `isPlaying`, `progress`, `volume`, `duration`.
- Cualquier `BeatCard` o página de beat individual dispara acciones sobre el store (`play(beatId)`, `pause()`, `seek()`), nunca crea su propio elemento de audio.
- Al reproducir un beat nuevo, el store pausa automáticamente el anterior.
- El preview (`previewAudio`) se sirve en MP3 ~96–128kbps, optimizado para streaming, completamente separado del `finalFile`.
- Indicador visual: el `BeatCard` correspondiente al `currentTrack` del store debe reflejar el estado activo (ícono de play/pause, animación o resaltado).

---

## 9. Seguridad — checklist obligatorio (revisar en cada fase relevante)

- [ ] Validación y sanitización de **todos** los inputs con Zod, tanto en cliente como server-side (nunca confiar solo en la validación del cliente).
- [ ] **Verificación de firma del webhook de Mercado Pago** — sin excepción, sin bypass "temporal".
- [ ] `Beat.finalFile` nunca se incluye en ninguna respuesta de API pública ni en el bundle de cliente.
- [ ] Rate limiting en endpoints públicos sensibles (creación de orden, creación de preferencia) para evitar abuso/spam.
- [ ] Autenticación y roles en el panel de Payload — solo Sicilia (y vos) tienen acceso de escritura.
- [ ] Variables de entorno para **todo** secreto (DB, R2, Mercado Pago) — nunca en el repo, `.env.example` sí se versiona con placeholders.
- [ ] Content-Security-Policy y headers de seguridad básicos configurados en Next.js (`next.config.js` / middleware).
- [ ] HTTPS obligatorio en producción (Vercel lo provee por defecto — no desactivar).
- [ ] CORS restringido a los orígenes necesarios (el propio dominio + Mercado Pago para el webhook).
- [ ] Datos personales del cliente (nombre, email, Instagram) tratados con cuidado: no loggear en texto plano en logs persistentes, y dejar una política de privacidad mínima visible en el sitio.
- [ ] Protección contra doble submit del formulario de compra (evitar crear órdenes duplicadas).

---

## 10. Performance

- `next/image` para todo asset visual, con tamaños responsivos.
- Lazy loading de secciones pesadas (catálogo completo, reproductor) fuera del viewport inicial.
- ISR (`revalidate`) en páginas de beats para no regenerar en cada request pero mantenerse actualizadas.
- Previews de audio comprimidos, nunca los archivos finales servidos por error.
- Code splitting natural de Next.js — evitar imports pesados en el bundle inicial (ej. cargar Howler solo en el cliente).

---

## 11. SEO

- Next.js Metadata API por página (`title`, `description`, Open Graph).
- `sitemap.ts` y `robots.ts` dinámicos, incluyendo cada `/beats/[slug]`.
- JSON-LD tipo `Product` (o `MusicRecording` donde aplique) en cada página de beat.
- Favicon, URLs amigables (slugs), estructura semántica correcta (h1/h2 por sección).

---

## 12. Calidad de código / Definition of Done

Una feature se considera terminada solo si:

- TypeScript strict, sin `any` sin justificar.
- ESLint + Prettier sin errores ni warnings nuevos.
- Estados de carga y error manejados explícitamente en toda interacción async (fetch, submit, redirect).
- Convención de commits respetada.
- README actualizado si se agregan pasos de setup o variables de entorno nuevas.
- Tests unitarios (Vitest) para lógica crítica no trivial: cálculo/transiciones de estado de la orden, validaciones de Zod, helpers de Mercado Pago (mockeado). No se exige e2e completo en v1, pero dejar Playwright configurado y con al menos un test smoke del flujo de compra.

---

## 13. Variables de entorno necesarias

```
DATABASE_URL=
PAYLOAD_SECRET=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET_NAME=
R2_ENDPOINT=
MP_ACCESS_TOKEN=
MP_WEBHOOK_SECRET=
NEXT_PUBLIC_SITE_URL=
NEXT_PUBLIC_SPOTIFY_PLAYLIST_ID=
```

Todas documentadas en `.env.example` con placeholders, nunca con valores reales en el repo.

---

## 14. Plan de fases (ejecución agéntica)

Cada fase termina con build + lint + typecheck en verde antes de avanzar.

1. **Fase 0 — Setup**: proyecto Next.js + TypeScript, Payload CMS embebido, conexión a Postgres, configuración de R2, estructura de carpetas base, `.env.example`.
2. **Fase 1 — Modelo de datos**: collections `Beat`, `License`, `Order`, `Media` en Payload, con validaciones y relaciones. Seed de datos de prueba (2-3 beats, 3 licencias).
3. **Fase 2 — Home y secciones estáticas**: Hero, Producciones (embed Spotify), Servicios, Sobre mí, Contacto. Responsive completo.
4. **Fase 3 — Catálogo y reproductor**: listado de beats, página individual por slug, reproductor global (Zustand + Howler) con indicador visual activo.
5. **Fase 4 — Flujo de compra (sin pago real aún)**: selector de licencia, formulario con RHF+Zod, creación de `Order` en DB con status `PENDING_PAYMENT`.
6. **Fase 5 — Integración Mercado Pago**: creación de preferencia dinámica, redirección, webhook con verificación de firma, actualización automática de estado, página de confirmación.
7. **Fase 6 — SEO, performance y pasada de seguridad final**: metadata, sitemap, robots, JSON-LD, revisión completa del checklist de la sección 9.
8. **Fase 7 — QA end-to-end y deploy**: recorrido manual completo del flujo (catálogo → compra → pago sandbox → webhook → confirmación), deploy a Vercel, verificación en producción con credenciales de test de Mercado Pago.

---

## 15. Roadmap post-v1 (NO implementar ahora — solo dejar la arquitectura lista)

- Descarga automática del archivo final al confirmarse el pago.
- Emails automáticos (Resend + React Email).
- Cuentas de clientes / historial de compras.
- Dashboard administrativo con analytics.
- Cupones, packs de beats, promociones.
- Generación automática de licencias en PDF.

---

## 16. Reglas de negocio inquebrantables (Anexo — tomadas del prompt original)

1. No existe descarga gratuita desde el catálogo.
2. Los previews solamente sirven para escuchar.
3. La licencia siempre debe ser elegida explícitamente por el usuario.
4. No se preselecciona ninguna licencia (ni Premium ni ninguna otra).
5. El beat se selecciona antes del formulario.
6. La licencia se selecciona antes del formulario.
7. Beat y licencia aparecen automáticamente (read-only) en el formulario.
8. Nombre completo es obligatorio.
9. Email es obligatorio.
10. Instagram es opcional.
11. Todos los beats tienen el mismo precio según el tipo de licencia.
12. El pago se gestiona vía Mercado Pago con preferencia dinámica por orden (actualización acordada sobre la versión original de 3 links estáticos).
13/14/15. Basic, Premium y Exclusive usan siempre el precio definido en su `License` correspondiente.
16. Los datos del formulario se guardan en DB **antes** de redirigir a Mercado Pago.

---

## 17. Anexo A — Especificación de producto original

*(Contenido íntegro del prompt de producto entregado por el cliente — máxima autoridad sobre el comportamiento esperado de cada sección. Ver archivo `Prompt_Senior___Desarrollo_Web_de_Sicilia.md` adjunto en el repo/contexto del proyecto para el detalle completo sección por sección: Home/Hero, Producciones, Catálogo de beats, Reproductor, Modal de compra, Formulario, Mercado Pago, Registro de datos, Servicios, Sobre mí, Contacto, Arquitectura de beats, Escalabilidad, Responsive, Performance, SEO, Reglas del sistema de venta y Flujo final de compra.)*

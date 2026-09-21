# PROMPT SENIOR — WEB SICILIA

Quiero desarrollar una web profesional para **Sicilia**, enfocada principalmente en:

- Producciones musicales
- Catálogo y venta de beats
- Servicios profesionales de producción y audio
- Presentación personal

La referencia visual para el catálogo de beats es:

https://niv0web.vercel.app

Usar esa web únicamente como referencia de experiencia e interacción del catálogo. **No copiar diseño, código ni identidad visual.**

---

# 1. HOME / HERO

Crear una portada limpia y profesional que presente a Sicilia como productor musical.

Debe incluir:

**SICILIA**

**Music Producer**

Una breve descripción de la actividad.

CTAs principales:

**ESCUCHAR PRODUCCIONES**

**EXPLORAR BEATS**

**TRABAJAR CONMIGO**

---

# 2. PRODUCCIONES

Esta sección debe estar dedicada exclusivamente a las producciones musicales de Sicilia.

Integrar una **playlist de Spotify mediante el embed oficial de Spotify**.

El usuario debe poder:

- Reproducir la playlist directamente desde la web.
- Pausar.
- Cambiar de canción.
- Navegar por la playlist.

Debajo del reproductor incluir:

**ABRIR EN SPOTIFY ↗**

Este botón debe abrir directamente la playlist en Spotify.

No utilizar reproductor propio para esta sección.

---

# 3. BEATS — CATÁLOGO

Crear un catálogo de beats inspirado en la experiencia de navegación de:

https://niv0web.vercel.app

El objetivo es que se sienta como una **beat library / beat store**, pero con identidad propia.

Cada beat debe mostrar:

- Artwork / portada
- Nombre
- Género
- BPM
- Tonalidad
- Preview
- Botón de compra

Ejemplo:

**MIDNIGHT**

Dark Trap · 140 BPM · F# Minor

▶ PLAY

**COMPRAR**

No incluir ninguna opción de descarga dentro del catálogo.

El usuario solamente puede:

**Escuchar → Comprar**

---

# 4. REPRODUCTOR DE BEATS

Implementar un reproductor de previews.

Características:

- Play / Pause
- Barra de progreso
- Duración
- Volumen
- Un solo beat reproduciéndose simultáneamente
- Si se reproduce otro beat, el anterior se pausa
- Indicador visual del beat actualmente activo

Los archivos utilizados como previews deben estar separados de los archivos finales de entrega.

No exponer WAV, stems ni archivos finales desde el catálogo.

---

# 5. PÁGINA / MODAL DE COMPRA

Cuando el usuario selecciona un beat para comprar, primero debe elegir una licencia.

Mostrar:

### BASIC

Precio definido por Sicilia.

**ELEGIR**

### PREMIUM

Precio definido por Sicilia.

**ELEGIR**

### EXCLUSIVE

Precio definido por Sicilia.

**ELEGIR**

La licencia **NO debe seleccionarse automáticamente**.

El usuario debe elegir explícitamente una de las tres.

---

# 6. FORMULARIO DE COMPRA

Después de seleccionar:

**Beat + Licencia**

mostrar el formulario.

Título:

**DATOS DE COMPRA**

Campos:

### Nombre completo
**OBLIGATORIO**

### Email
**OBLIGATORIO**

### Instagram
**OPCIONAL**

### Beat elegido
**OBLIGATORIO — PRESELECCIONADO AUTOMÁTICAMENTE**

### Licencia elegida
**OBLIGATORIO — PRESELECCIONADO AUTOMÁTICAMENTE**

El usuario NO debe escribir manualmente el beat ni la licencia.

Estos datos deben venir automáticamente de las selecciones realizadas anteriormente.

Los campos pueden mostrarse como `read-only` para evitar modificaciones accidentales.

CTA:

**CONTINUAR AL PAGO**

---

# 7. MERCADO PAGO

La primera versión utilizará **links de pago de Mercado Pago**, sin implementar todavía una integración completa mediante API.

Como todos los beats tendrán el mismo precio según la licencia, solamente habrá:

**1 link de Mercado Pago para Basic**

**1 link de Mercado Pago para Premium**

**1 link de Mercado Pago para Exclusive**

Total:

**3 links de pago.**

El sistema debe detectar la licencia elegida y redirigir al link correspondiente.

Ejemplo:

```text
MIDNIGHT
↓
PREMIUM
↓
FORMULARIO
↓
CONTINUAR AL PAGO
↓
MERCADO PAGO — LINK PREMIUM
```

---

# 8. REGISTRO DE DATOS

Antes de redirigir al usuario a Mercado Pago, guardar la información enviada.

Registrar:

- Nombre
- Email
- Instagram
- Beat
- Licencia
- Fecha
- Estado

Estados iniciales:

**PENDING PAYMENT**

**PAID**

**CANCELLED**

Inicialmente el estado de pago puede ser actualizado manualmente después de verificar Mercado Pago.

La arquitectura debe permitir automatizar este proceso posteriormente.

---

# 9. SERVICIOS

Crear una sección para presentar los servicios profesionales de Sicilia.

Servicios:

### MUSIC PRODUCTION

Producción musical personalizada.

### BEAT PRODUCTION

Producción de beats personalizados.

### AUDIO EDITING

Edición y limpieza profesional de audio.

### SOUND DESIGN

Diseño sonoro.

### MIXING / AUDIO POST-PRODUCTION

Servicios de mezcla y postproducción según el proyecto.

Cada servicio debe incluir:

- Nombre
- Descripción breve
- CTA

CTA:

**TRABAJAR CONMIGO**

---

# 10. SOBRE MÍ

Crear una sección breve de presentación profesional.

Incluir:

- Foto
- Nombre artístico
- Descripción profesional
- Especialidades
- DAW
- Herramientas principales

La sección debe ser concisa y orientada a generar confianza.

---

# 11. CONTACTO

Crear una sección clara para contacto profesional.

Incluir:

**Email**

**Instagram**

**WhatsApp**, si se decide utilizar.

**LinkedIn**

**Spotify**

**YouTube**

CTA principal:

**INICIAR PROYECTO**

---

# 12. ARQUITECTURA DE LOS BEATS

No hardcodear individualmente cada beat.

Cada beat debe manejarse como un objeto/dato independiente.

Estructura mínima:

```text
id
title
slug
cover
preview
genre
bpm
key
mood
status
```

Las licencias deben ser independientes:

```text
license
name
price
mercadoPagoLink
description
features
```

La compra debe relacionar:

```text
Beat + License + Customer
```

Esto permitirá administrar todo el catálogo de manera escalable.

---

# 13. ESCALABILIDAD

La arquitectura debe permitir agregar posteriormente:

- Mercado Pago API
- Webhooks
- Confirmación automática de pagos
- Descarga automática
- Generación automática de licencias
- Emails automáticos
- Cuentas de clientes
- Historial de compras
- Dashboard administrativo
- Analytics
- Cupones
- Packs de beats
- Promociones

No implementar estas funciones ahora salvo que sean necesarias.

La primera versión debe mantenerse simple.

---

# 14. RESPONSIVE

La web debe funcionar correctamente en:

- Desktop
- Tablet
- Mobile

Especial atención a:

- Catálogo
- Reproductor
- Selección de licencia
- Formulario
- Spotify Embed
- Botones de compra

La experiencia de compra debe ser cómoda desde teléfono.

---

# 15. PERFORMANCE

Priorizar:

- Carga rápida
- Imágenes optimizadas
- Audio previews optimizados
- Lazy loading
- Código modular
- Buen rendimiento mobile

Los previews de audio deben estar optimizados para streaming y no utilizar archivos innecesariamente pesados.

---

# 16. SEO

Configurar:

- Title
- Meta description
- Open Graph
- Favicon
- URLs amigables
- Sitemap
- Robots.txt

Cada beat debe poder tener una URL individual.

Ejemplo:

`/beats/midnight`

---

# 17. REGLAS DEL SISTEMA DE VENTA

Estas reglas son obligatorias:

1. No existe descarga gratuita desde el catálogo.
2. Los previews solamente sirven para escuchar.
3. La licencia siempre debe ser elegida por el usuario.
4. No seleccionar Premium automáticamente.
5. El Beat se selecciona antes del formulario.
6. La Licencia se selecciona antes del formulario.
7. Beat y Licencia aparecen automáticamente en el formulario.
8. Nombre completo es obligatorio.
9. Email es obligatorio.
10. Instagram es opcional.
11. Todos los beats tienen el mismo precio según el tipo de licencia.
12. Existen solamente 3 links de Mercado Pago.
13. Basic utiliza el link Basic.
14. Premium utiliza el link Premium.
15. Exclusive utiliza el link Exclusive.
16. Los datos del formulario deben guardarse antes de redirigir a Mercado Pago.

---

# 18. FLUJO FINAL DE COMPRA

El flujo debe ser exactamente:

```text
CATÁLOGO
    ↓
SELECCIONAR BEAT
    ↓
ELEGIR LICENCIA
    ↓
FORMULARIO
    ↓
NOMBRE
EMAIL
INSTAGRAM (OPCIONAL)
BEAT (AUTOMÁTICO)
LICENCIA (AUTOMÁTICO)
    ↓
CONTINUAR AL PAGO
    ↓
MERCADO PAGO
    ↓
PAGO
```

La experiencia debe ser rápida, clara y sin pasos innecesarios.

---

# 19. OBJETIVO DE LA PRIMERA VERSIÓN

La primera versión debe resolver perfectamente tres cosas:

### PRODUCCIONES

Spotify Embed + botón **Abrir en Spotify**.

### BEATS

Catálogo visual + previews + selección de licencia.

### VENTA

Formulario → Mercado Pago.

No sobrecargar la primera versión con funciones que todavía no son necesarias.

La arquitectura debe estar preparada para crecer posteriormente, pero el producto inicial debe ser **simple, rápido y funcional**.
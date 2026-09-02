# Plan — Youngers in Action: CMS para la coordinadora + rediseño "Awwwards" con foco en performance

## Contexto

`youngers-in-action` es un sitio **estático de Astro 4.16** en **Netlify**
(repo `git@github.com:Casapaico/youngers-in-action.git`, git user `casapaico`).

**Problema:** hoy **todo el contenido está hardcodeado** en el frontmatter de los `.astro`
(arrays JS: `timeline`, `valores`, `odsData`, `beneficios`, `tiposVoluntariado`, `equipo`,
`faqs`, `secciones` legales…), las imágenes viven sueltas en `public/images/` sin
optimización, y **no existe** blog/novedades ni calendario de reuniones. La coordinadora
no puede publicar nada sin un desarrollador.

**Objetivo:**
1. Un **panel `/admin` súper amigable** donde la coordinadora edite fotos, textos, novedades
   y fechas de reuniones/programaciones, sin tocar código y sin costo.
2. Subir el nivel visual: **el minijuego arranca al cargar** (como ahora) y el resto del
   sitio pasa a una **presentación cinemática con scroll dinámico** inspirada en webs de
   ONGs profesionales — **manteniendo el amarillo YIA `#FFB800`** y **priorizando
   performance** (público de Lima, mucho móvil de gama media/baja).
3. Una **"carpeta de actividades"** (guía + checklist) para que la coordinadora reúna
   fotos, textos y permisos, y mantenga el sitio vivo con el tiempo.

### Decisión de arquitectura — NO usar AWS/S3/backend propio

El usuario planteó backend en AWS + panel en S3 + subdominio. **Recomiendo no hacerlo.**
Para un sitio informativo estático, un backend propio agrega costo (el free tier de AWS
caduca a los 12 meses y es fácil generar cargos), mantenimiento y superficie de
seguridad, **sin beneficio**. El estándar moderno para este caso es un **CMS basado en
Git (JAMstack)**, todo gratuito:

```
Coordinadora ──edita en──▶  /admin  (Sveltia CMS, 1 archivo HTML desde CDN)
                                │  login con su cuenta de GitHub
                                ▼
                    Cloudflare Worker de auth (gratis)  ──▶  GitHub OAuth App
                                │
                                ▼  "Guardar" = commit a src/content/**
                         GitHub (Casapaico/youngers-in-action)
                                │  webhook
                                ▼
                          Netlify  ──build──▶  sitio publicado (~1–2 min)
```

- **Sin servidor, sin base de datos, $0.** Historial y reversión gratis vía Git.
- Si algún día hace falta algo dinámico (registro a eventos, guardar formularios), se
  añade una **Netlify Function** puntual (gratis) — no un backend entero.

Fuentes: [Sveltia CMS docs](https://sveltiacms.app/en/docs/start) ·
[sveltia-cms-auth](https://github.com/sveltia/sveltia-cms-auth) ·
[Sveltia + Astro](https://ergaster.org/til/sveltia-cms-astro/) ·
[Setup Sveltia para blog Astro](https://bryanhogan.com/blog/sveltia-cms-astro-blog/)

### Evaluación de las skills/repos aportados (verificadas, sin malware)

| Repo | Qué es | Veredicto de seguridad | Uso en este plan |
|---|---|---|---|
| **zanwei/design-dna** *(ya instalado)* | Skill: extrae/aplica "ADN de diseño" (tokens + estilo + efectos). Scripts Node solo miden colores (k-means), dep `sharp`. | **Limpia.** Sin red, sin hooks de instalación, sin env vars. MIT. | Fuente de verdad del diseño: se genera `design-dna.json` del sitio + 3–4 referencias de ONGs. |
| **Leonxlnx/taste-skill** | Skills de "buen gusto" frontend (anti-slop): tipografía, ritmo, motion, layout. Solo Markdown; los `.mjs` del repo son mantenimiento de su README, no se ejecutan al usar la skill. | **Bajo riesgo** (instrucciones Markdown). MIT. | Instalar `taste-skill` + `redesign-skill`. Dan dirección y barra de calidad. |
| **nateherkai/scroll-craft** | Plugin Claude Code para webs "scroll-driven premium": motor JS/CSS, 8 "gramáticas" de página, verificación headless (Playwright + ffmpeg). Incluye `kie.mjs` que sube imágenes a `api.kie.ai`/`redpandaai.co` para generar assets IA (requiere `KIE_AI_API_KEY` de pago). | **No malicioso**, pero pesado. `kie.mjs` solo corre si se opta explícitamente por IA con API key. | Instalar el plugin; usar **su gramática "chaptered editorial" + técnicas ligeras** (CSS `animation-timeline`, IntersectionObserver, `prefers-reduced-motion`). **NO usar `kie.mjs`** (hay fotos de menores). |
| **anthropics/skills** | Skills oficiales de Anthropic. | **Segura** (primera parte). | Traer `webapp-testing` (verificación headless tras cada cambio) y `theme-factory` (tokens/tema cohesivo). |

Tensión a gestionar: scroll-craft empuja a efectos pesados; el usuario prioriza performance.
El plan usa scroll cinético **solo en la home**, con presupuesto de performance estricto,
y páginas internas rápidas y simples.

---

## Estado real del código (hallazgos del inventario)

- **8 páginas**, **5 componentes vivos** (`MinimalHeader`, `FullscreenMenu`, `DotNavigation`,
  `ImpactGame`, `ui/OdsCard`). **21 componentes son código muerto** con datos de demo
  (`sections/*`, casi todo `ui/*`, `layout/Header|Footer|Navigation|MobileMenu`, `PageLayout`).
- **React está instalado pero no se usa** (0 islands) → se puede quitar.
- **Sin `src/content/`, sin Content Collections, sin `netlify.toml`, sin `@astrojs/sitemap`,
  sin `astro:assets`.** Imágenes en `public/` servidas tal cual (PNGs pesados).
- **Formularios**: contacto y voluntariado tienen `action="https://formspree.io/f/placeholder"`
  (no funciona); en realidad un script hace `preventDefault()` y abre WhatsApp
  `wa.me/51989942045`. El `NewsletterForm` es falso y no está montado.
- **Contenido duplicado** que necesita fuente única: nombres+colores de los 17 ODS (4 copias),
  cifras de impacto ("35 escolares / 12 iniciativas / 85% / 10+ alianzas / 67.7%") repartidas
  en 4 archivos a mano, `socialLinks` copiado en 3 sitios (2 versiones divergentes más).
- **Datos huérfanos ya escritos** (definidos, nunca renderizados — la coordinadora quizá los
  quiera de vuelta): `contacto.astro` `contactInfo`; `proyectos/empoderarlas.astro` `equipo`,
  `resultados`, `impactoPropuestas`.
- **Strings que caducan**: `'15 de enero de 2025'` en `privacidad`/`terminos`;
  "Proyecto Activo 2026"; timeline 2023–2025; `og:locale` `es_ES` (debería `es_PE`).
- **Imágenes faltantes** referenciadas: `/images/og-default.jpg`, `/apple-touch-icon.png`.
- **~18 imágenes huérfanas** en `public/images/` + carpetas `team/` y `logos/` sin uso.
- **`ImpactGame.astro`**: 4.013 líneas / ~167 KB en un archivo (markup + ~1.870 líneas CSS
  `is:global` + ~1.480 líneas TS con motor de audio Web Audio, confetti, focus-trap). Todo
  **inline en la home**. Su copy editable (`PROFILES`, textos de decisiones, "2.847 jóvenes")
  está hardcodeado en el `<script>`. Bugs: `#btn-join` hace scroll a `#hero` en vez de ir a
  `/voluntariado`; comparte URL con casing errado `youngersInAction.org`; imágenes
  `game/lima-hero.png` y `game/yia-team.png` solo referenciadas en comentarios.

---

## Fase 0 — Preparación y dirección de diseño

**0.1 Skills / plugins** (una vez):
- `npx skills add Leonxlnx/taste-skill -a claude-code -g -y` (revisar antes de usar)
- `/plugin marketplace add nateherkai/scroll-craft` + `/plugin install nateherk-design`
- Copiar `webapp-testing` y `theme-factory` desde `anthropics/skills` a `~/.claude/skills/`
- **No** configurar `KIE_AI_API_KEY`.

**0.2 Upgrade de Astro 4.16 → última (7.2.x)** — *checkpoint con aprobación intermedia*
- Rama dedicada, `npx @astrojs/upgrade`, luego arreglar lo que rompa.
- Riesgo real: en Astro 5 los `<script>` de `.astro` ya no se hoistean ni se bundlean juntos
  — este sitio tiene muchos `<script>` inline (home, ImpactGame, layouts). Hay que revisarlos
  uno a uno. Es un sitio de 8 páginas: manejable.
- Beneficio: **Content Layer API** (loader `glob`), mejor `astro:assets`, imágenes responsive
  en build.
- **Fallback** si el upgrade se complica: quedarse en Astro 4 y usar Content Collections
  "legacy" (`src/content/config.ts`). El resto del plan no cambia.
- Quitar `@astrojs/react`, `react`, `react-dom` (0 islands). Añadir `@astrojs/sitemap`.

**0.3 ADN de diseño (design-dna Fase 2 → 3)**
- Analizar el sitio actual + **3–4 referencias de ONGs profesionales** con foco en scroll y
  arquitectura de página. Candidatas: charity: water, Malala Fund, Room to Read, Girl Effect,
  Acumen, Ashoka. (El usuario puede sustituir por las que prefiera.)
- Producir `design/design-dna.json`: paleta (fija el amarillo `#FFB800` + neutros + acentos
  ODS), escala tipográfica (Poppins display / Inter texto), spacing, sombras, radios, y un
  bloque `visual_effects` con scroll-driven suave.
- Con `theme-factory` + `taste-skill`, derivar **tokens CSS custom properties** en
  `src/styles/tokens.css` (hoy todo es hex a mano) y un mini design-system documentado.

**0.4 Presupuesto de performance** (se verifica en cada fase con `webapp-testing` + Lighthouse):
- LCP < 2.5 s en 4G simulado · CLS < 0.1 · TBT < 200 ms
- JS total < 150 KB gzip en la home · < 40 KB en páginas internas
- Lighthouse móvil ≥ 90 en Performance y Accesibilidad
- Todas las imágenes vía `<Image>`/`<Picture>` (`astro:assets`), formato AVIF/WebP,
  `width`/`height` explícitos, `loading="lazy"` salvo la del LCP.

---

## Fase 1 — Content Collections (migrar TODO el contenido editable)

Crear `src/content.config.ts` (o `src/content/config.ts` en Astro 4) con colecciones
tipadas (Zod). Imágenes editables vía `image()` en el schema, guardadas en
`src/assets/uploads/` para que Astro las optimice.

### Colecciones "singleton" (1 entrada, formulario simple en el CMS)

| Colección | Reemplaza | Campos clave |
|---|---|---|
| `config/site` | `socialLinks` ×3, email, WhatsApp, handles | `orgName`, `tagline`, `email`, `whatsapp`, `instagram`, `linkedin`, `facebook`, `direccion`, `ogImage`, `googleMaps?` |
| `config/impact` | cifras repartidas en 4 archivos | `escolares`, `iniciativas`, `participacionActiva`, `alianzas`, `propuestasPersonales`, `odsPriorizados[]` |
| `config/home` | textos de `index.astro` | hero (eyebrow, título, 2 CTAs), copy de secciones programa / ODS / únete / aliados |
| `config/juego` | copy hardcodeado en `ImpactGame` `<script>` | `PROFILES[]` (título, icono, color, desc, ods, quote, cta), textos de decisiones, "jóvenes que jugaron" |
| `pages/nosotros` | misión/visión/quote/hero de `nosotros.astro` | `heroTitle`, `heroImg`, `misionTitulo`, `misionCuerpo`, `visionTitulo`, `visionCuerpo`, `quote` |
| `pages/empoderarlas` | textos + `areasAfectadas`, `pilares` de `proyectos/empoderarlas.astro` | hero, "por qué nace", `areasAfectadas[]`, `pilares[]`, `odsRelacionados[]`, `activo`, `anio` |

### Colecciones "lista" (varias entradas, tabla en el CMS)

| Colección | Reemplaza | Campos |
|---|---|---|
| `ods` | `odsData` + 4 duplicados de nombres/colores | `number`, `title`, `color`, `descripcion`, `comoContribuimos[]`, `meta`, `impactoValor`, `impactoDesc`, `prioritario:bool`, `order` |
| `timeline` | `nosotros.astro` `timeline` | `anio`, `titulo`, `descripcion`, `imagen`, `order` |
| `valores` | `nosotros.astro` `valores` | `titulo`, `descripcion`, `icono`, `order` |
| `beneficios-voluntariado` | `voluntariado.astro` `beneficios` | `titulo`, `descripcion`, `icono`, `order` |
| `tipos-voluntariado` | `voluntariado.astro` `tiposVoluntariado` | `titulo`, `descripcion`, `compromiso`, `icono`, `requisitos[]`, `order` |
| `faqs` | `contacto.astro` `faqs` | `pregunta`, `respuesta`, `categoria` (contacto/voluntariado/general), `order` |
| `aliados` | tarjetas hardcodeadas en `index.astro` | `nombre`, `logo`, `tipo`, `descripcion`, `url?`, `order` |
| `equipo` | `equipo` huérfano en `empoderarlas.astro` | `nombre`, `rol`, `foto?`, `categoria` (fundadora/mentor/autoridad/respaldo), `order` |
| `legal` | `secciones` HTML de `privacidad` + `terminos` | `slug`, `titulo`, `fechaActualizacion`, body **Markdown** (migrar el HTML a MD) |

### Nuevas (Fase 3, ver abajo): `novedades`, `programacion`.

### Refactor de páginas
Cada `.astro` pasa de `const x = [...]` a
`const x = await getCollection('x')` / `getEntry(...)`, ordenando por `order`/`date`.
Extraer componentes de presentación reutilizables: `<OdsCard>` (ya existe, quitar sus
datos internos y alimentarlo desde `ods`), `<Timeline>`, `<ValorGrid>`, `<AliadoCard>`,
`<StatBar>` (consume `config/impact`), `<SocialLinks>` (consume `config/site`).
Borrar los 21 componentes muertos y las ~18 imágenes huérfanas (tras confirmar con captura).

**Verificación Fase 1:** `npm run build` sin errores; `webapp-testing` compara cada página
contra captura previa (contenido idéntico, layout intacto); `git diff` del HTML generado.

---

## Fase 2 — Panel `/admin` (Sveltia CMS)

**2.1 Archivos en el repo**
- `public/admin/index.html` — carga Sveltia desde CDN (`https://unpkg.com/@sveltia/cms` pinneado).
- `public/admin/config.yml` — `backend: { name: github, repo: Casapaico/youngers-in-action,
  branch: main, base_url: <WORKER_URL> }`; `media_folder: "src/assets/uploads"`,
  `public_folder: "/src/assets/uploads"`; `i18n` off; **todas las colecciones de Fase 1**
  mapeadas con widgets amigables (string, text, markdown, image, list, select, number,
  boolean, datetime), con `label`/`hint` en español y `preview` cuando ayude.
- `netlify.toml` — `[build] command="npm run build"`, `publish="dist"`; headers básicos.
- `robots.txt` con `Disallow: /admin`.

**2.2 Autenticación (fuera del repo, una vez)**
- Desplegar `sveltia/sveltia-cms-auth` como **Cloudflare Worker** (plan free).
- Crear **GitHub OAuth App**: Homepage = URL de Netlify; Authorization callback =
  `https://<worker>.workers.dev/callback`. Guardar `GITHUB_CLIENT_ID` / `SECRET` como
  secrets del Worker.
- Poner `base_url` del Worker en `config.yml`.
- Agregar a la coordinadora como **colaboradora** del repo GitHub (permiso *Write*).

**2.3 Onboarding**
- Documento corto "Cómo entrar y publicar" (parte de la carpeta de actividades, §Fase 5).
- Prueba end-to-end: la coordinadora cambia una foto y un texto desde `/admin` → commit →
  build de Netlify → cambio visible.

**Verificación Fase 2:** login OK con cuenta de prueba; editar cada tipo de colección desde
la UI genera el commit correcto; el build de Netlify pasa; `/admin` no aparece en el sitemap
ni en buscadores.

---

## Fase 3 — Novedades + Programación/Reuniones

**3.1 Colección `novedades`** (blog de noticias de la ONG)
- Schema: `titulo`, `fecha` (datetime), `portada` (image), `resumen`, `cuerpo` (Markdown),
  `etiquetas[]`, `autor?`, `destacada:bool`, `borrador:bool`.
- Páginas: `/novedades` (índice paginado, tarjetas con portada) y `/novedades/[slug]`
  (artículo, `<Image>` optimizada, SEO/OG por post).
- `@astrojs/rss` → `/rss.xml`. Bloque "Últimas novedades" en la home (3 más recientes).

**3.2 Colección `programacion`** (reuniones, talleres, convocatorias, fechas)
- Schema: `titulo`, `tipo` (reunión / taller / convocatoria / evento), `inicio` (datetime),
  `fin?`, `lugar` (texto o "Virtual"), `enlace?` (Meet/Zoom/registro), `descripcion`,
  `dirigidoA` (voluntarios / mentores / público / escolares), `recurrente?` (texto libre,
  ej. "cada sábado"), `publicado:bool`.
- Página `/agenda`: próximas fechas agrupadas por mes, con filtro por `dirigidoA`; las
  pasadas se ocultan automáticamente (lógica en build + `<time>` semántico).
- Bloque "Próximas fechas" en la home y en `/voluntariado`.
- Export `.ics` por evento (link "Agregar a mi calendario") — generado en build, sin backend.

**3.3 CMS**: ambas colecciones con formularios muy simples (la de programación:
básicamente título + fecha + lugar + a quién va dirigido). `editor.preview` activado.

**Verificación Fase 3:** crear una novedad y un evento de prueba desde `/admin`; aparecen en
`/novedades`, `/agenda` y en los bloques de la home; el `.ics` abre bien en Google Calendar;
un evento con fecha pasada desaparece de la agenda.

---

## Fase 4 — Rediseño "Awwwards" + performance

**Gramática de página (home), inspirada en scroll-craft "chaptered editorial":**
`Minijuego (carga inmediata, como hoy)` → `Capítulo 1: el problema` → `Capítulo 2:
EmpoderArLAS` → `Capítulo 3: impacto en números` → `Capítulo 4: los ODS` → `Capítulo 5:
súmate` → `Aliados + cierre`. Cada capítulo entra con scroll-driven reveal, imagen grande
optimizada y una cifra/dato ancla.

**4.1 Técnica de scroll (GPU-friendly, sin librería pesada)**
- CSS **`animation-timeline: scroll()` / `view()`** para reveals, parallax sutil y barras de
  progreso — cero JS.
- Mantener el `IntersectionObserver` existente de `[data-reveal]` como fallback.
- Conservar `scroll-snap` en la home actual solo si no pelea con el scroll cinético
  (probablemente se relaja a snap por capítulo, no por sección).
- **`@media (prefers-reduced-motion: reduce)`** desactiva todo el motion (ya hay base).
- Nada de video-scrubbing ni WebGL (presupuesto de performance).

**4.2 Tokens y consistencia visual**
- `src/styles/tokens.css` con las CSS custom properties del `design-dna.json`.
- Unificar amarillo en `#FFB800` (hoy el token Tailwind es `#f3b923`, distinto).
- Tipografía: `font-display: swap`, subset latino, `preload` de la fuente del LCP,
  self-host opcional (quita 1 request a Google Fonts).

**4.3 Imágenes**
- Migrar imágenes de contenido a `src/assets/`, servir con `<Image>`/`<Picture>`.
- Recomprimir las que suba el CMS (Sveltia respeta el pipeline de Astro).
- `og-default.jpg` + `apple-touch-icon.png` + favicon completo (faltan).

**4.4 Minijuego (`ImpactGame`)**
- Mover las ~1.870 líneas de CSS `is:global` a `src/styles/impact-game.css` (import normal).
- **Diferir** el motor de audio, el confetti y el focus-trap hasta la 1ª interacción del
  usuario (import dinámico) → baja el JS inicial de la home.
- Arreglar bugs: `#btn-join` → `/voluntariado`; URL `https://youngersinaction.org`;
  quitar comentarios de imágenes muertas; borrar `game/lima-hero.png` y `game/yia-team.png`
  si no se adoptan.
- Copy del juego desde `config/juego` (Fase 1) para que la coordinadora lo ajuste.

**4.5 Accesibilidad y SEO**
- Contraste AA en todo el texto (taste-skill/scroll-craft lo verifican).
- Landmarks, `alt` reales (desde el CMS), foco visible, navegación por teclado del menú.
- `@astrojs/sitemap`, `canonical` correcto, `og:locale` `es_PE`, datos estructurados
  `Organization` + `Article` (novedades) + `Event` (programación).
- Formularios contacto/voluntariado: cambiar el `action` placeholder de Formspree por
  algo real y gratuito — **Netlify Forms** (100 envíos/mes gratis) — manteniendo el botón
  de WhatsApp como opción secundaria. (Confirmar con el usuario en Fase 3/4.)

**Verificación Fase 4:** Lighthouse móvil ≥ 90 (Perf + A11y) en home, `/nosotros`,
`/novedades/[slug]`; `webapp-testing` en 3 viewports (móvil 360, tablet 768, desktop 1440)
sin scroll muerto ni fallos de contraste; `prefers-reduced-motion` respetado;
comparación visual antes/después con el usuario.

---

## Fase 5 — "Carpeta de actividades" para la coordinadora

Entregable: carpeta `docs/coordinadora/` en el repo + versión publicada como página privada
(artifact) para compartir. Contenido:

1. **`01-como-publicar.md`** — Entrar a `/admin`, iniciar sesión con GitHub, editar,
   "Guardar", esperar 1–2 min, verificar. Con capturas.
2. **`02-checklist-fotos.md`** — Qué fotos se necesitan y con qué specs:
   - Portadas de novedades: 1200×800, horizontal, < 300 KB.
   - Hero home / secciones: 1920×1080, punto focal centrado.
   - Equipo: 800×800 cuadrada.
   - Aliados: logo PNG con fondo transparente, 400 px de alto.
   - **Permisos de menores**: toda foto con rostros de escolares necesita autorización
     firmada de tutores; preferir fotos de espaldas / actividad / manos. Plantilla de
     consentimiento incluida.
   - Herramientas gratis para comprimir: Squoosh, TinyPNG.
3. **`03-inventario-contenido.md`** — Tabla de todo lo editable hoy (de la Fase 1) y quién
   es responsable de mantener cada pieza.
4. **`04-calendario-editorial.md`** — Ritmo sugerido: 1 novedad cada 2 semanas; actualizar
   `/agenda` los lunes; revisar cifras de impacto cada trimestre; refrescar fotos del hero
   cada semestre. Ideas de contenido recurrente (testimonios, "detrás de escena", logros).
5. **`05-textos-pendientes.md`** — Lista concreta de textos que hoy están genéricos o
   caducados y hay que reemplazar con contenido real:
   - `equipo` real (hoy nombres inventados: "Ana Lucía Vargas", "Dra. Patricia Huamán"…).
   - Fechas de `privacidad`/`terminos` (hoy "15 de enero de 2025").
   - Teléfono real (hoy placeholder `+51 999 888 777`).
   - Unificar handle de Instagram (hoy hay 2 versiones).
   - Confirmar cifras: "35 escolares", "12 iniciativas", "2.847 jóvenes", "67.7%", "10+ alianzas".
   - Foto/retrato de la fundadora y bio.
6. **`06-glosario-y-marca.md`** — Uso del amarillo YIA, logo, tono de voz, do/don't.

---

## Orden de ejecución y checkpoints

1. **Fase 0** (prep + upgrade Astro + ADN de diseño) → *checkpoint: aprobar upgrade y ADN.*
2. **Fase 1** (Content Collections) → *checkpoint: build + paridad visual.*
3. **Fase 2** (Sveltia `/admin` + auth) → *checkpoint: la coordinadora edita en una prueba real.*
4. **Fase 3** (Novedades + Programación).
5. **Fase 4** (Rediseño + performance) → *checkpoint: Lighthouse + revisión visual.*
6. **Fase 5** (Carpeta de actividades) — puede empezar en paralelo desde la Fase 1.

Cada fase = una rama y un PR con la atribución de commits indicada.

## Archivos/áreas que se tocan (resumen)

- **Nuevos:** `src/content.config.ts`, `src/content/**` (todas las colecciones),
  `src/assets/uploads/`, `public/admin/{index.html,config.yml}`, `netlify.toml`,
  `public/robots.txt`, `src/styles/tokens.css`, `src/styles/impact-game.css`,
  `src/pages/novedades/**`, `src/pages/agenda.astro`, `src/pages/rss.xml.ts`,
  `design/design-dna.json`, `docs/coordinadora/**`.
- **Refactor:** las 8 páginas `src/pages/**`, `src/layouts/*`, `ImpactGame.astro`,
  `astro.config.mjs`, `package.json`, `tailwind.config.mjs`, `src/styles/global.css`.
- **Borrar:** 21 componentes muertos, `@astrojs/react`+react+react-dom, ~18 imágenes
  huérfanas, carpetas `public/logos/` y `public/images/team/` (si no se adoptan),
  `public/ASSETS-GUIA.md` (reemplazada por `docs/coordinadora/`).

## Riesgos y mitigaciones

- **Upgrade Astro 4→7**: scripts inline dejan de bundlearse (v5). *Mitigación:* rama aislada,
  revisar cada `<script>`, `webapp-testing` en las 8 páginas; fallback a Astro 4 + colecciones legacy.
- **Peso del repo por imágenes del CMS**: si suben muchas fotos grandes, el repo crece.
  *Mitigación:* Sveltia comprime vía Astro; si escala, activar media library externa gratis
  (Cloudinary free) más adelante.
- **La coordinadora y GitHub**: fricción de tener cuenta. *Ya confirmado que sí puede.*
  El onboarding (§2.3) lo cubre con capturas.
- **Scroll cinético vs performance**: *Mitigación:* solo CSS scroll-driven, presupuesto
  estricto, `prefers-reduced-motion`, sin video/WebGL.
- **Fotos de menores**: *Mitigación:* la carpeta de actividades incluye política y plantilla
  de consentimiento; el CMS no publica sin que la coordinadora suba la imagen.

## Verificación global (al final)

- `npm run build` limpio; `astro check` sin errores de tipos.
- Lighthouse móvil ≥ 90 (Perf + A11y + SEO) en home y 2 páginas internas.
- Recorrido `webapp-testing` en móvil/tablet/desktop de las 8 páginas + `/novedades` + `/agenda`.
- Prueba real de la coordinadora: publicar 1 novedad, 1 evento, cambiar 1 foto y 1 cifra,
  todo desde `/admin`, y verlo en producción.
- Formularios entregan (Netlify Forms) o abren WhatsApp correctamente.
- `/admin` bloqueado en `robots.txt` y fuera del sitemap.

# Handoff — estado del trabajo (sesión 2026-09-01)

Punto de partida para continuar en la próxima sesión con Claude Code.

## Qué se decidió

- **CMS para la coordinadora**: Sveltia CMS en `/admin` + backend GitHub + Netlify.
  **Sin AWS, sin servidor, sin base de datos, $0.** Auth vía Cloudflare Worker
  (`sveltia-cms-auth`) + GitHub OAuth App. La coordinadora entra con su cuenta de GitHub
  (ya confirmó que puede tener una).
- **Dirección visual**: se mantiene el amarillo YIA `#FFB800`. El minijuego sigue
  cargando apenas abre la página; el resto pasa a presentación cinemática con scroll
  dinámico (referencias: webs de ONGs profesionales), **priorizando performance**.
- **Alcance de la sesión**: solo plan + guía. **No se tocó el código del sitio todavía.**

## Documentos de esta sesión (en este repo)

- `docs/plan-plataforma-cms-rediseno.md` — **el plan completo por fases** (0 a 5),
  arquitectura, inventario del código, riesgos y verificación. Leer esto primero.
- `docs/coordinadora/` — la "carpeta de actividades" para la coordinación:
  - `carpeta-actividades.html` — versión interactiva (checklist con autoguardado).
  - `carpeta-actividades-print.html` — versión maquetada para PDF.
  - `Carpeta-de-Actividades-YIA.pdf` — PDF de 10 páginas listo para enviar.
  - Artifact publicado: https://claude.ai/code/artifact/d43b70c4-b0bf-4661-a1d0-266efb2f5d86

## Herramientas instaladas (en `~/.claude/skills/`, fuera del repo)

- `design-dna` — ADN de diseño (tokens + estilo + efectos).
- `taste-skill` y subskills (`redesign-existing-projects`, `high-end-visual-design`,
  `minimalist-ui`, `image-to-code`, `brandkit`…).
- `webapp-testing` (Anthropic) — verificación headless con Playwright.
- `theme-factory` (Anthropic) — generación de tokens/tema.
- Los 3 repos aportados por el usuario se auditaron: **sin malware**. Nota: `scroll-craft`
  trae `kie.mjs` que sube imágenes a un servicio de IA de pago → **no usar** (hay fotos
  de menores).

## Pendiente para arrancar la Fase 0

1. **El usuario debe instalar el plugin scroll-craft** (comandos suyos, no míos):
   - `/plugin marketplace add nateherkai/scroll-craft`
   - `/plugin install nateherk-design`
   - **No** configurar `KIE_AI_API_KEY`.
2. **Confirmar referencias de diseño** para `design-dna` Fase 2. Propuesta:
   charity: water, Malala Fund, Room to Read, Girl Effect. El usuario puede cambiarlas.
3. **Luz verde para tocar código.** La Fase 0 empieza por: actualizar Astro 4.16 → última
   (checkpoint con aprobación intermedia) + generar `design/design-dna.json`.

## Estado del repositorio

- Rama: `main`.
- **Hay 6 archivos modificados sin commitear que YA estaban así al empezar la sesión**
  (trabajo previo del usuario, no lo tocó Claude):
  `src/components/ImpactGame.astro`, `src/components/layout/DotNavigation.astro`,
  `src/pages/index.astro`, `src/pages/ods.astro`,
  `src/pages/proyectos/empoderarlas.astro`, `src/styles/global.css`.
  Un apagado de PC **no** los borra — siguen en disco. Decidir en la próxima sesión si
  se commitean o se descartan antes de empezar la Fase 1.
- El servidor local corría en `http://localhost:4321/` (`npm run dev`).

## Orden de fases (resumen)

| Fase | Qué |
|---|---|
| 0 | Prep: skills, upgrade de Astro, ADN de diseño, presupuesto de performance |
| 1 | Content Collections: migrar TODO el contenido hardcodeado a `src/content/**` |
| 2 | Sveltia CMS en `/admin` + auth + onboarding de la coordinadora |
| 3 | Novedades (blog) + Programación/Agenda de reuniones |
| 4 | Rediseño "Awwwards" + performance (Lighthouse ≥ 90 móvil) |
| 5 | Carpeta de actividades, versión final con capturas del panel real |

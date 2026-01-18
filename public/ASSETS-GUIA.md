# Guía de Assets - Youngers in Action

## Estructura de carpetas

```
public/
├── favicon.svg              ← Logo de YIA en formato SVG (favicon del navegador)
├── apple-touch-icon.png     ← Logo 180x180px para iOS
├── logo-yia.svg             ← Logo principal de la organización
│
├── images/
│   ├── equipo-yia.jpg           ← Foto grupal del equipo (800x600px)
│   ├── metodologia-yia.jpg      ← Foto de metodología/talleres (800x600px)
│   ├── empoderarlas-intro.jpg   ← Imagen principal EmpoderArLAS (1200x800px)
│   ├── mentorias-action.jpg     ← Foto de mentorías en acción (800x600px)
│   ├── pattern-dots.svg         ← Patrón decorativo de puntos
│   ├── pattern-grid.svg         ← Patrón decorativo de cuadrícula
│   │
│   ├── dona/                    ← Imágenes para página de donaciones
│   │   ├── hero-jovenes-capacitacion.jpg      ← Hero principal (1920x1080px)
│   │   ├── jovenes-beneficiarios-capacitacion.jpg ← Jóvenes en capacitación
│   │   ├── grupo-voluntarios-reunion.jpg      ← Foto grupal voluntarios
│   │   ├── directora-ong-retrato.jpg          ← Retrato directora
│   │   ├── reunion-plaza-comunidad.jpg        ← Reunión en plaza
│   │   ├── taller-educativo-jovenes.jpg       ← Taller educativo
│   │   ├── entrega-materiales-escolares.jpg   ← Entrega de materiales
│   │   ├── fondo-jovenes-actividad.jpg        ← Fondo sección pasos
│   │   └── fondo-equipo-sonriendo.jpg         ← Fondo sección contacto
│   │
│   ├── ods/                     ← Imágenes para página ODS
│   │   ├── ods-wheel.png            ← Rueda de los 17 ODS (puede ser PNG o SVG)
│   │   ├── ods-4-educacion.jpg      ← Actividad relacionada ODS 4
│   │   ├── ods-5-genero.jpg         ← Actividad relacionada ODS 5
│   │   ├── ods-3-salud.jpg          ← Actividad relacionada ODS 3
│   │   ├── ods-8-trabajo.jpg        ← Actividad relacionada ODS 8
│   │   └── ods-17-alianzas.jpg      ← Actividad relacionada ODS 17
│   │
│   ├── alianzas/                ← Logos e imágenes de aliados
│   │   └── senaju-alianza.jpg       ← Foto de alianza con SENAJU
│   │
│   ├── equipo/                  ← Fotos individuales del equipo
│   │   ├── miembro-1.jpg            ← Foto miembro (400x400px, cuadrada)
│   │   ├── miembro-2.jpg
│   │   └── ...
│   │
│   └── proyectos/               ← Imágenes de proyectos
│       └── empoderarlas-hero.jpg    ← Imagen principal proyecto
│
├── logos/                       ← Logos de aliados institucionales
│   ├── senaju.svg                   ← Logo SENAJU
│   ├── ugel.svg                     ← Logo UGEL 05
│   ├── aliado3.svg                  ← Logo aliado educativo
│   └── aliado4.svg                  ← Logo aliado institucional
│
└── videos/
    └── dona/
        └── jovenes-en-accion.mp4    ← Video hero página donaciones (opcional)
```

---

## Especificaciones técnicas

### Formatos recomendados:
- **Fotos**: JPG (calidad 80-85%)
- **Logos**: SVG (vectorial) o PNG con transparencia
- **Iconos**: SVG
- **Videos**: MP4 (H.264, max 10MB para web)

### Tamaños sugeridos:
| Tipo | Dimensiones | Uso |
|------|-------------|-----|
| Hero/Banner | 1920x1080px | Fondos principales |
| Cards | 800x600px | Tarjetas de proyectos |
| Equipo | 400x400px | Fotos de miembros (cuadradas) |
| Logos | 200x200px max | Logos de aliados |
| Favicon | 32x32px (SVG escalable) | Ícono navegador |
| Apple Touch | 180x180px | Ícono iOS |

### Optimización:
- Comprimir imágenes antes de subir (TinyPNG, Squoosh)
- Peso máximo recomendado: 200KB por imagen
- Videos: máximo 10MB, considerar usar YouTube/Vimeo embed

---

## Cómo agregar assets

1. **Coloca el archivo** en la carpeta correspondiente
2. **Nombra el archivo** exactamente como se indica arriba
3. **El sitio lo detectará automáticamente** (los placeholders desaparecerán)

---

## Gráficos estadísticos (página ODS)

Para los gráficos de la página `/ods`, tienes 2 opciones:

### Opción A: Imágenes estáticas
- Crea los gráficos en Canva, Figma o Excel
- Expórtalos como PNG/JPG
- Colócalos en `/images/ods/`

### Opción B: Gráficos interactivos (requiere código)
- Usar Chart.js o similar
- Requiere modificar el código de la página

---

## Notas importantes

- Las imágenes de menores deben tener **autorización de tutores**
- Evitar mostrar rostros identificables sin consentimiento
- El logo de YIA debe ser consistente en todo el sitio

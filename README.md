# Akai Ito - Plantilla de Cumpleaños Romántica

Plantilla single-file `index.html` inspirada en la leyenda del **Hilo Rojo del Destino (Akai Ito)**, adaptada para cumpleaños con tema festivo.

## ✨ Características

- **Single-file** — Todo en un `index.html` (CSS + JS embebidos, sin build)
- **Temática cumpleaños** — SVG festivo animado: globos, pastel con velas, confeti, regalos, nombre prominente
- **Contador vivo** — Desde fecha configurable (ej. nacimiento: `2000-09-05`)
- **Flujo interactivo**:
  1. Hero → Click **DESCÚBRELO** → Confeti
  2. Abre **video modal** fullscreen (muted, música de fondo sigue sonando)
  3. Video termina → **Auto-cierra** → Aparece **carta teaser** (fade-in-up)
  4. Click sobre → **Carta modal** con typewriter effect (45ms/char texto, 40ms/char firma)
- **Música de fondo** — Autoplay con fade-in (2.5s) + botón vinilo fallback si el navegador bloquea autoplay
- **Video compuesto** — 10 imágenes con crossfades (configurable: 5s c/u, transiciones 1.5s)
- **WebGL Shader** — Fondo tipo papel texturizado + 3 hilos orgánicos + partículas doradas (reactivo a tema claro/oscuro)
- **Accesibilidad** — ARIA labels, focus-visible, semantic HTML, `prefers-reduced-motion`
- **Responsive** — Mobile-first, max-width 720px desktop

## 📁 Estructura esperada (assets locales)

```
/tu-carpeta/
├── index.html          ← Este archivo
├── musica.mp3          ← Tu música de fondo (loop)
├── video.mp4           ← Tu video compuesto (9:16, ~36s)
└── imagenes/
    ├── principal.jpg   ← Foto principal (polaroid + poster video)
    └── IMG-*.jpg       ← Fotos para generar el video (opcional)
```

> **Nota:** Los assets (`musica.mp3`, `video.mp4`, `imagenes/`) **NO se incluyen** en el repo. Cada usuario pone los suyos.

## ⚙️ Configuración rápida

Edita la sección `CONFIGURACIÓN CENTRALIZADA` en el `<script>` al final del `index.html`:

```javascript
// Fecha para el contador "Tiempo juntos" (formato: AAAA-MM-DD)
const ANNIVERSARY_DATE = '2000-09-05';

// Fecha de cumpleaños para mostrar en polaroid (formato: DD/MM/AAAA)
const BIRTHDAY_DATE = '05/09/2000';

// Nombres de la pareja
const COUPLE_NAMES = 'Lowel & Yazari';

// Texto de la carta (usa \n para saltos de línea)
const LETTER_TEXT = `Tu texto aquí...`;

// Firma de la carta
const LETTER_SIGNATURE = 'Con todo mi amor, Tu Nombre.';
```

## 🎬 Generar el video compuesto

Si tienes las fotos en `imagenes/`, usa este script Python (requiere `ffmpeg`):

```python
# create_video.py - Ejecuta desde la carpeta imagenes/
import subprocess

W, H = 1080, 1920
FR = 30
DUR = 5.0      # Segundos por imagen
TRANS = 1.5    # Crossfade duración

images = [
    "IMG-1.jpg", "IMG-2.jpg", ..., "principal.jpg"
]

# 1. Crear segmentos
for i, img in enumerate(images):
    subprocess.run(["ffmpeg", "-y", "-loop", "1", "-t", str(DUR), "-i", img,
        "-vf", f"scale={W}:{H}:force_original_aspect_ratio=decrease,pad={W}:{H}:(ow-iw)/2:(oh-ih)/2:color=black@0.1,format=yuv420p",
        "-c:v", "libx264", "-pix_fmt", "yuv420p", "-r", str(FR), f"seg_{i}.mp4"], check=True)

# 2. Crossfade chain
OFFSET = DUR - TRANS
filter_parts = []
prev = "0:v"
for i in range(1, len(images)):
    curr = f"{i}:v"
    out = f"v{i-1}{i}" if i < len(images)-1 else "outv"
    filter_parts.append(f"[{prev}][{curr}]xfade=transition=fade:duration={TRANS}:offset={i*OFFSET}[{out}]")
    prev = out

inputs = []
for i in range(len(images)): inputs.extend(["-i", f"seg_{i}.mp4"])

subprocess.run(["ffmpeg", "-y"] + inputs + [
    "-filter_complex", ";".join(filter_parts),
    "-map", "[outv]", "-c:v", "libx264", "-pix_fmt", "yuv420p", "-r", str(FR),
    "-crf", "20", "../video.mp4"
], check=True)
```

## 🚀 Deploy

**Netlify Drop:** Arrastra la carpeta completa (con tus assets) a [app.netlify.com/drop](https://app.netlify.com/drop)

**GitHub Pages / Vercel / Cloudflare Pages:** Sube la carpeta y configura el directorio raíz.

## 🎨 Personalización

- **Colores:** Edita las CSS variables en `:root` (líneas 24-66)
- **Fuentes:** Cambia `--font-display` / `--font-body` (Google Fonts en `<head>`)
- **Duración video:** Cambia `DUR` y `TRANS` en el script de generación
- **Textos:** Modifica `LETTER_TEXT`, `LEGEND_TITLE`, `VIDEO_TITLE`, etc.

## 📄 Licencia

MIT — Úsalo libremente para tus proyectos románticos 💕

---

**Inspirado en:** Google Stitch prototypes + Leyenda japonesa del Akai Ito  
**Desarrollado con:** Vanilla HTML/CSS/JS + WebGL + FFmpeg
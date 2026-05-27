# Interactive Map Dashboard

> Dashboard web interactivo para visualizar datos sobre un plano o mapa SVG personalizado.
> 100% estático, gratis, sin dependencias, fácil de compartir.

![status](https://img.shields.io/badge/status-MVP-2ec28a)
![license](https://img.shields.io/badge/license-MIT-4d9fff)
![deps](https://img.shields.io/badge/dependencies-0-10e0c4)

Pensado originalmente para monitoreo RF de antenas sobre el plano de un piso de
aeropuerto, pero el patrón sirve para **cualquier mapa SVG con regiones nombradas**:
pisos de oficina, almacenes, tiendas, plantas industriales, estadios, recintos
de eventos, mapas geográficos personalizados, etc.

---

## ✨ Características

- **Plano SVG interactivo** — tooltip al pasar el cursor, filtrado por clic en cada región.
- **4 modos de mapa de calor** — por estado/categoría, por dos métricas numéricas, y por conteo.
- **Panel de KPIs reactivo** — los indicadores se recalculan según la región seleccionada.
- **Tabla de detalle** con búsqueda en vivo y enlace bidireccional con el plano.
- **Modo offline** con datos embebidos + **modo en vivo** desde Google Sheets.
- **Un solo archivo HTML** — sin build, sin `npm`, sin servidor. Lo abres y funciona.
- **Compartible** — se publica gratis en GitHub Pages, Netlify Drop, Cloudflare Pages, etc.

---

## 🚀 Inicio rápido

### Opción A — Solo verlo localmente
Doble clic en `index.html`. Listo.

### Opción B — Publicarlo en GitHub Pages (gratis)
1. Sube este repo a GitHub.
2. Ve a **Settings ▸ Pages**.
3. En *Source* elige la rama `main` y la carpeta `/ (root)`.
4. En unos segundos tu dashboard estará en
   `https://<tu-usuario>.github.io/interactive-map-dashboard/`.

### Opción C — Arrastrar y soltar
Ve a [netlify.com/drop](https://app.netlify.com/drop), arrastra `index.html` y
te devuelve un link público al instante. Útil para previews.

---

## 📁 Estructura del proyecto

```
.
├── index.html              # Dashboard auto-contenido (HTML + CSS + JS)
├── assets/
│   └── floor-plan.svg      # Plano fuente (Inkscape) — referencia y reutilización
├── data/
│   └── sample-data.xlsx    # Datos de muestra (50 antenas, 9 zonas)
├── docs/
│   ├── google-sheets-setup.md   # Cómo conectar datos en vivo
│   └── customization.md         # Cómo cambiar el plano y los datos
├── .gitignore
├── LICENSE
└── README.md
```

> **Nota:** la geometría de las regiones y los datos de muestra están **embebidos**
> dentro de `index.html` para que el dashboard funcione sin conexión. Los archivos
> en `assets/` y `data/` son las **fuentes originales** que se usan para
> regenerar `index.html` cuando se cambia el plano o el dataset.

---

## 📊 Conectar datos en vivo (Google Sheets)

El dashboard puede leer datos en tiempo real desde una hoja de Google Sheets
publicada como CSV. Esto te da un "backend gratis" hasta que decidas migrar a
una base de datos real.

**Pasos resumidos** (la guía completa está en [`docs/google-sheets-setup.md`](docs/google-sheets-setup.md)):

1. Sube `data/sample-data.xlsx` a Google Drive y ábrelo con Google Sheets.
2. **Archivo ▸ Compartir ▸ Publicar en la web** — selecciona la hoja
   `Piso_1_Antenas`, formato `CSV`, y pulsa *Publicar*.
3. Copia la URL que genera y pégala en `index.html`:
   ```js
   const CONFIG = {
     SHEET_CSV_URL: "https://docs.google.com/spreadsheets/d/e/.../pub?gid=0&single=true&output=csv",
     AUTO_REFRESH_SECONDS: 60   // 0 = manual
   };
   ```
4. Recarga el dashboard. El indicador superior pasará a `En vivo` 🟢.

---

## 🔧 Personalización (usar tu propio plano / datos)

### Cambiar el plano
1. Dibuja tu plano en Inkscape, Figma, Illustrator o el editor que prefieras.
2. Asigna a cada región un `id` con el patrón `REG_01`, `REG_02`, …
   (o el nombre que uses como clave; debe **coincidir exactamente** con la
   columna `ZONA MAPA` de tus datos).
3. Exporta como SVG y reemplaza `assets/floor-plan.svg`.
4. Copia el atributo `d=` de cada región a `ZONA_PATHS` dentro de `index.html`
   y, si tu plano tiene un contorno de muros, pégalo en el `<path class="muro">`.
5. (Opcional) ajusta el `viewBox` y los colores en `:root` para tu marca.

> Guía detallada en [`docs/customization.md`](docs/customization.md).

### Cambiar los datos de muestra
Edita el bloque `EMBEDDED_DATA` dentro de `index.html`, o conecta directamente
una Google Sheet (recomendado).

---

## 🗺️ Roadmap

Este es el plan de evolución del proyecto. La idea es que el dashboard se
mantenga **simple y portable** mientras gana capacidades de backend.

### Fase 1 — MVP ✅ (estado actual)
- [x] Dashboard estático con plano SVG interactivo
- [x] Datos embebidos para demo offline
- [x] Heatmap por estado y por métricas RF
- [x] Lectura opcional de Google Sheets publicado como CSV

### Fase 2 — Captura de datos en campo 🚧
Que el personal de campo pueda **levantar y actualizar** información sin
acceso al Excel maestro.
- [ ] Formulario web de captura (Google Forms → alimenta la misma Sheet)
- [ ] Alternativa móvil: PWA simple con foto + GPS + timestamp
- [ ] Validación de zonas y campos en el formulario
- [ ] Historial de cambios (mediante revisiones de la Sheet o columna `updated_at`)

### Fase 3 — Base de datos real ⏳
Migrar de Google Sheets a un backend que escale.
- [ ] **Supabase** o **Firebase Firestore** (ambos con tier gratuito generoso)
- [ ] Endpoint REST que devuelva el mismo formato JSON que ya consume el dashboard
- [ ] Autenticación básica (correo + contraseña, o Google SSO)
- [ ] Roles: campo (solo escribe), supervisor (lee y edita), admin
- [ ] Aplicativo móvil ligero (PWA o Flutter) conectado al mismo backend

### Fase 4 — Multipiso / Multisitio ⏳
- [ ] Selector de piso/edificio/sitio en el dashboard
- [ ] Vista comparativa entre zonas equivalentes de distintos pisos
- [ ] Filtros globales (tecnología, banda, fecha de intervención)
- [ ] Exportar reportes (PDF/Excel) de la zona o piso seleccionado

### Fase 5 — Tiempo real e inteligencia ⏳
- [ ] WebSocket / Server-Sent Events para refresco instantáneo
- [ ] Alertas configurables (RSRP por debajo de umbral, antenas críticas > N)
- [ ] Histórico de mediciones por antena con mini-gráfico de tendencia
- [ ] (Opcional) sugerencias con modelo: detectar zonas en riesgo

---

## 🛠️ Stack actual

| Capa             | Tecnología                          |
| ---------------- | ----------------------------------- |
| Frontend         | HTML5, CSS3, JavaScript vanilla     |
| Gráficos         | SVG nativo del navegador            |
| Tipografía       | Chakra Petch, Archivo, JetBrains Mono (Google Fonts) |
| Datos (Fase 1)   | JSON embebido + CSV de Google Sheets |
| Datos (Fase 3)   | _por definir_ — Supabase / Firebase  |
| Hosting          | GitHub Pages / Netlify / Cloudflare Pages |
| Build            | _ninguno_                            |

Sin frameworks, sin bundler, sin pasos de compilación. Si funciona en el
navegador, funciona en producción.

---

## 🤝 Cómo subirlo a GitHub (desde WSL)

```bash
# 1. Descomprime el zip y entra a la carpeta
cd ~/proyectos
unzip ~/Downloads/interactive-map-dashboard.zip
cd interactive-map-dashboard

# 2. Inicializa git
git init
git add .
git commit -m "feat: MVP del dashboard interactivo con plano SVG"

# 3. Crea el repo vacío en github.com (sin README ni .gitignore — ya los tienes)
#    Luego conéctalo:
git branch -M main
git remote add origin https://github.com/<tu-usuario>/interactive-map-dashboard.git
git push -u origin main
```

> Si prefieres usar la CLI de GitHub: `gh repo create interactive-map-dashboard --public --source=. --push`

---

## 📄 Licencia

[MIT](LICENSE) — úsalo, modifícalo, distribúyelo libremente.

---

## 🙏 Créditos

- Plano original dibujado en [Inkscape](https://inkscape.org/).
- Tipografías de [Google Fonts](https://fonts.google.com/).
- Idea original: visualizar el estado RF de antenas piso por piso sin pagar
  licencia de Power BI premium.

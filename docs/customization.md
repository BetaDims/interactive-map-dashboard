# 🔧 Personalización: usar tu propio plano y datos

Este dashboard fue diseñado para ser **agnóstico al dominio**: la lógica de
interactividad, KPIs y filtrado funciona con cualquier plano SVG y cualquier
dataset, siempre que cumplan dos contratos simples.

## Los dos contratos

1. **Plano SVG** — cada región interactiva es un `<path>` con un `id` único.
2. **Datos** — cada registro tiene una columna `zona` cuyo valor coincide
   **exactamente** con uno de esos `id`.

Eso es todo. Si esos dos lados se hablan, el dashboard funciona.

---

## 1. Cambiar el plano

### Dibujar las regiones en Inkscape

1. Abre tu imagen base (foto del plano, blueprint, mapa, lo que sea) como
   capa de referencia.
2. Crea una capa nueva llamada por ejemplo `Regiones`.
3. Dibuja un rectángulo o polígono por cada zona que quieras que sea
   interactiva.
4. Selecciona cada forma → `Object Properties` (Ctrl+Shift+O) → asigna un
   **`Label`** y un **`ID`** consistente: `REG_01`, `REG_02`, …
   (puedes usar otros nombres como `ZONA_A`, `SALA_101`; lo que importa es
   que coincidan con tus datos).
5. Quita el relleno y deja solo un trazo fino para que se vea como guía;
   el dashboard les aplicará el color dinámicamente.
6. Exporta como **SVG plano (Plain SVG)** — Archivo → Guardar como.

### Llevarlo al dashboard

Abre el SVG con un editor de texto y localiza tus paths:

```xml
<path id="REG_01" d="m 32.94,146.05 h 50.76 v 18.15 H 32.94 Z" ... />
<path id="REG_02" d="m 83.85,126.11 h 31.36 v 38.99 H 83.85 Z" ... />
```

Copia cada atributo `d=` y pégalo dentro del objeto `ZONA_PATHS` en
`index.html`:

```js
const ZONA_PATHS = {
  REG_01: "m 32.94,146.05 h 50.76 v 18.15 H 32.94 Z",
  REG_02: "m 83.85,126.11 h 31.36 v 38.99 H 83.85 Z",
  // ...
};
```

### Contorno de muros (opcional)

Si tu plano tiene un contorno general (paredes, edificio), está dibujado en
el `<path class="muro">` del SVG dentro de `index.html`. Reemplaza ese `d=`
con el de tu propio contorno, o elimina el `<path class="muro">` completo
si no lo necesitas.

### Ajustar el `viewBox`

El dashboard reajusta automáticamente el `viewBox` al contenido — no
necesitas hacer nada. Pero si quieres más o menos margen, busca en el JS:

```js
const bb = document.getElementById("contenido").getBBox();
const pad = 8;   // ← márgen alrededor del contenido, en unidades SVG
```

---

## 2. Cambiar los datos

### Estructura mínima esperada

| Columna           | Tipo    | Notas                                                    |
| ----------------- | ------- | -------------------------------------------------------- |
| `antena` (clave)  | texto   | identificador único del registro (puede llamarse `id`, `equipo`, etc.) |
| `zona`            | texto   | **debe coincidir** con un `id` del SVG                   |
| `estado`          | texto   | una de tres categorías. El parser detecta `Crítico` / `Degradado` / `Óptimo` por prefijo (case insensitive, ignora tildes) |
| `rsrp` (métrica1) | número  | usado para el heatmap "RSRP promedio"                    |
| `sinr` (métrica2) | número  | usado para el heatmap "SINR promedio"                    |
| `tecnologia`      | texto   | filtro de detalle                                        |
| `banda`           | texto   | filtro de detalle                                        |
| `tipo_trabajo`    | texto   | columna informativa de la tabla                          |
| `intervencion`    | fecha   | columna informativa (formato libre)                      |

Si tu caso de uso es otro (mesas de un restaurante, máquinas de una planta,
salones de un centro comercial), simplemente **renombra mentalmente**:
- `antena` → `mesa` / `máquina` / `salón`
- `rsrp` → tu primera métrica numérica (ocupación, temperatura, ventas)
- `sinr` → tu segunda métrica numérica

Para que el dashboard hable tu idioma, hay que tocar etiquetas en el HTML
y el JS — busca y reemplaza textos como `Antenas`, `RSRP`, `SINR`,
`dBm`, `dB`.

### Adaptar los rangos de color de las métricas

Por defecto el código asume que **mayor = mejor** (verde) para las dos
métricas numéricas. Si una métrica tuya funciona al revés (donde menos es
mejor), invierte el cálculo dentro de la función `zonaColor`:

```js
// Cambiar:
return scaleColor(mx === mn ? .5 : (b[key] - mn) / (mx - mn));
// Por:
return scaleColor(mx === mn ? .5 : 1 - (b[key] - mn) / (mx - mn));
```

---

## 3. Cambiar la paleta y la marca

Todos los colores viven en variables CSS al inicio del `<style>`:

```css
:root {
  --bg: #0b0e13;          /* fondo principal */
  --accent: #10e0c4;      /* cian de acento */
  --accent-2: #4d9fff;    /* azul secundario */
  --optimo: #2ec28a;      /* verde estado bueno */
  --degradado: #f0b429;   /* ámbar estado intermedio */
  --critico: #f2585b;     /* rojo estado malo */
}
```

Cámbialos y el resto del UI se reacomoda automáticamente. El logo, título y
tipografías están en el `<header>` — fácil de identificar.

---

## 4. Versionar las personalizaciones

Si vas a hacer varios planos (varios pisos, varios sitios), una opción
práctica es:

- Mantener un `index.html` por plano: `piso-1.html`, `piso-2.html`, …
- O extraer `ZONA_PATHS` y `EMBEDDED_DATA` a archivos `.js` aparte y
  cargarlos con `<script src="planta-1.js"></script>`.

La versión actual prioriza tener **todo en un solo archivo** porque hace
que compartir y publicar sea trivial. Cuando llegues a la Fase 3 del roadmap
(base de datos real) tendrá sentido refactorizar a múltiples archivos.

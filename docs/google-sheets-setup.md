# 📊 Conectar datos en vivo con Google Sheets

Google Sheets funciona como un **backend gratuito** muy práctico para esta
primera etapa: alguien edita la hoja, el dashboard refleja los cambios al
recargar (o automáticamente, si activas el refresco).

## Requisitos

- Una cuenta de Google.
- El archivo `data/sample-data.xlsx` (o tu propio dataset con la **misma
  estructura de columnas**).

## Paso a paso

### 1. Subir el Excel a Google Sheets

1. Ve a [drive.google.com](https://drive.google.com).
2. **Nuevo ▸ Subida de archivos** → selecciona `sample-data.xlsx`.
3. Una vez subido, doble clic para abrirlo.
4. **Archivo ▸ Guardar como hojas de cálculo de Google** (o desde Drive,
   clic derecho → *Abrir con → Hojas de cálculo*).

> Recomendación: renómbralo a algo claro como `Antenas Piso 1 - Producción`.

### 2. Publicar la hoja como CSV

1. Con la hoja abierta, ve a **Archivo ▸ Compartir ▸ Publicar en la web**.
2. En el cuadro de diálogo:
   - **Vincular** (no Insertar).
   - En el primer desplegable selecciona la hoja específica:
     `Piso_1_Antenas` (no "Todo el documento").
   - En el segundo selecciona **`Valores separados por comas (.csv)`**.
3. Activa **"Volver a publicar automáticamente cuando se hagan cambios"**.
4. Pulsa **Publicar** y acepta la advertencia.
5. **Copia la URL** que aparece — se parece a:
   ```
   https://docs.google.com/spreadsheets/d/e/2PACX-1vTxxxxxxxxxxxx.../pub?gid=0&single=true&output=csv
   ```

### 3. Pegar la URL en el dashboard

Abre `index.html` con cualquier editor (VS Code, Notepad++, nano, …) y busca
el bloque `CONFIG` cerca del inicio del `<script>`:

```js
const CONFIG = {
  SHEET_CSV_URL: "",            // ← pega tu URL aquí entre las comillas
  AUTO_REFRESH_SECONDS: 0       // 0 = manual, 60 = cada minuto, etc.
};
```

Después de pegar:

```js
const CONFIG = {
  SHEET_CSV_URL: "https://docs.google.com/spreadsheets/d/e/2PACX-.../pub?gid=0&single=true&output=csv",
  AUTO_REFRESH_SECONDS: 60
};
```

Guarda el archivo, recarga el dashboard. El indicador superior pasará de
`Datos locales` 🟡 a `En vivo` 🟢.

## Cómo verificar que funciona

1. En la Sheet, cambia el `ESTADO` de `ANT-001` a `Óptimo`.
2. Espera 1–2 minutos (Google cachea el CSV publicado).
3. En el dashboard pulsa el botón **Actualizar** o recarga la página.
4. La región `REG_01` y la fila correspondiente deben cambiar de color.

## Cosas que debes saber

| Tema           | Detalle                                                          |
| -------------- | ---------------------------------------------------------------- |
| Latencia       | Google cachea el CSV publicado por unos minutos. No es instantáneo. |
| Privacidad     | La URL publicada es **pública pero secreta** — cualquiera con el link lee los datos. No publiques datos sensibles así. |
| Estructura     | Si renombras columnas, el parser intenta detectarlas por palabras clave (`ZONA`, `ESTADO`, `RSRP`, …). Mantén nombres reconocibles. |
| Filas vacías   | Se ignoran automáticamente.                                      |
| Edición móvil  | Se puede editar desde la app de Sheets en celular sin problema.  |

## Captura de datos por el equipo de campo

Para que el personal en campo actualice información **sin tocar la Sheet
directamente**, la opción más rápida es un **Google Form**:

1. En la Sheet, **Herramientas ▸ Crear un formulario nuevo**.
2. Diseña el formulario con los mismos campos.
3. Las respuestas caen en una hoja nueva — usa una fórmula `QUERY` o
   simplemente mueve la lógica al `gid` correcto.

Esto cubre la **Fase 2** del roadmap mientras se evalúa migrar a una BD real.

## Solución de problemas

**El dashboard dice "Sin conexión · datos locales":**
- Verifica que pegaste la URL completa entre comillas.
- Abre la URL en una pestaña nueva — debe descargar un CSV.
- Si copias-y-pegas desde el navegador, asegúrate de que NO sea la URL de la
  hoja normal (`/edit#gid=0`), sino la **publicada** (contiene `/pub?` y
  `output=csv`).

**Faltan registros / aparecen vacíos:**
- La hoja publicada puede estar cacheada. Espera 5 minutos y pulsa Actualizar.
- Confirma que la hoja correcta está seleccionada en el cuadro de Publicar.

**Los datos cambian pero el dashboard no:**
- Pulsa el botón **Actualizar** del dashboard.
- Si activaste `AUTO_REFRESH_SECONDS`, espera ese intervalo.

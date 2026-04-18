---
name: gis-surface-characterization
description: >
  Usar este skill para derivar y analizar variables morfométricas del terreno
  en QGIS vía MCP a partir de un Modelo Digital de Elevación (DEM). Activar
  cuando el usuario mencione: hillshade, sombreado de relieve, slope, pendiente,
  aspect, orientación de ladera, roughness, rugosidad del terreno, análisis
  morfométrico, caracterización del relieve, curvatura, TPI, TRI, derivadas del
  DEM, análisis geomorfológico de superficie, forma del terreno, visualización
  3D del relieve, o cuando pida entender, describir o mapear la forma del
  terreno a partir de un DEM o MDT. No omitir este skill si la tarea implica
  cualquier derivada topográfica calculada sobre un modelo de elevación.
---

# Caracterización de Superficie del Terreno
## Hillshade · Slope · Aspect · Roughness

---

## 1. Misión y Dominio

| Campo         | Valor                                                                                          |
|---------------|------------------------------------------------------------------------------------------------|
| **Objetivo**  | Derivar las variables morfométricas fundamentales del terreno (iluminación, pendiente, orientación y rugosidad) a partir de un DEM para comprender la forma y comportamiento del relieve. |
| **Dominio**   | Geomorfología / Análisis Terrain / Teledetección Topográfica                                   |
| **Escala**    | Local (LiDAR/UAV: < 1 m – 5 m) / Regional (SRTM/ALOS: 12–30 m) / Nacional (SRTM 90 m)       |
| **Nivel**     | Intermedio                                                                                     |

---

## 2. Insumos Requeridos

### 2.1 Capas necesarias

| # | Capa                    | Tipo   | Contenido                                      | Resolución sugerida  | Obligatoria      |
|---|-------------------------|--------|------------------------------------------------|----------------------|------------------|
| 1 | DEM / MDT               | Raster | Modelo Digital de Elevación (superficie o terreno) | ≤ 30 m           | ✅ Sí            |
| 2 | Límite área de estudio  | Vector | Polígono del área de interés (AOI)            | —                    | ⚠️ Recomendada   |
| 3 | Cuerpos de agua / lagos | Vector | Para enmascarar zonas planas de agua          | —                    | ⚠️ Opcional      |

> **Fuentes de DEM por escala de trabajo:**
>
> | Fuente          | Resolución | Cobertura     | Descarga                           |
> |-----------------|------------|---------------|------------------------------------|
> | LiDAR / UAV     | 0.1–2 m    | Local         | Vuelo propio                       |
> | ALOS PALSAR     | 12.5 m     | Global        | ASF Data Search (Alaska)           |
> | SRTM GL1        | 30 m       | Global ±60°N  | earthexplorer.usgs.gov             |
> | Copernicus DEM  | 30 m       | Global        | Copernicus Land Service            |
> | SRTM GL3        | 90 m       | Global ±60°N  | earthexplorer.usgs.gov             |
> | ASTER GDEM v3   | 30 m       | Global ±83°N  | earthdata.nasa.gov                 |

### 2.2 Validación Pre-proceso ⚠️ CRÍTICO

> Claude/Antigravity/Gemini **debe verificar todos estos puntos** antes de derivar cualquier
> variable. Las derivadas del DEM son extremadamente sensibles al CRS y a
> la presencia de artefactos. Un DEM en coordenadas geográficas produce
> pendientes y rugosidades completamente erróneas.

- [ ] **CRS métrico obligatorio:** El DEM debe estar en proyección métrica (UTM u otra).
  - Si está en WGS84 geográfico (EPSG:4326) → **reprojectar antes de cualquier derivada**.
  - CRS recomendado Perú: `EPSG:32718` (UTM Zona 18S) o según área de estudio.
  - ⚠️ Un DEM en grados decimales producirá slopes en grados/metro incompatibles.
- [ ] **Unidades de elevación:** Verificar que las elevaciones estén en metros.
  - Si están en pies → multiplicar por 0.3048 antes de procesar.
- [ ] **NoData definido:** Verificar que el valor NoData esté establecido (usualmente -9999 o -32768).
  - Los píxeles NoData sin definir se procesan como elevación 0 → genera artefactos severos.
- [ ] **Artefactos y huecos:** Revisar si el DEM tiene huecos (stripes de SRTM, zonas sin datos).
  - Si hay huecos → rellenar con `gdal:fillnodata` antes de calcular derivadas.
- [ ] **Resolución adecuada:** Verificar que la resolución del DEM sea apropiada para la escala.
  - Slope a 30 m pierde detalle local significativo. Para análisis de detalle usar DEM ≤ 10 m.
- [ ] **Recorte al AOI:** Si el DEM cubre un área mucho mayor → recortar antes de procesar.

---

## 3. Flujo de Trabajo

### 3.1 Diccionario de Algoritmos

| Paso | ID Técnico QGIS                        | Herramienta | Descripción                                            |
|------|----------------------------------------|-------------|--------------------------------------------------------|
| 0a   | `gdal:cliprasterbymasklayer`           | GDAL        | Recortar DEM al AOI                                    |
| 0b   | `gdal:warpreproject`                   | GDAL        | Reprojectar DEM a CRS métrico (si necesario)           |
| 0c   | `gdal:fillnodata`                      | GDAL        | Rellenar huecos y NoData en el DEM                     |
| 1    | `gdal:hillshade`                       | GDAL        | Sombreado de relieve (iluminación simulada)            |
| 2    | `gdal:slope`                           | GDAL        | Pendiente en grados o porcentaje                       |
| 3    | `gdal:aspect`                          | GDAL        | Orientación de ladera (azimut 0–360°)                  |
| 4    | `gdal:roughness`                       | GDAL        | Rugosidad del terreno (variabilidad local de elevación)|
| 5    | `gdal:tpitopographicpositionindex`     | GDAL        | Índice de posición topográfica (TPI)                   |
| 6    | `gdal:triterrainruggednessindex`       | GDAL        | Índice de rugosidad del terreno (TRI — Riley 1999)     |
| 7    | `native:rasterlayerstatistics`         | QGIS        | Estadísticas de cada derivada                          |
| 8    | `native:reclassifybytable`             | QGIS        | Clasificar slope y aspect en categorías temáticas      |

> **Nota sobre Roughness vs TRI vs TPI:**
>
> | Variable | Algoritmo GDAL          | Qué mide                                                  |
> |----------|-------------------------|-----------------------------------------------------------|
> | Roughness| `gdal:roughness`        | Diferencia entre píxel central y vecinos (máx − mín)     |
> | TRI      | `gdal:triterrainruggednessindex` | Diferencia media absoluta con los 8 vecinos (Riley 1999) |
> | TPI      | `gdal:tpitopographicpositionindex` | Diferencia entre píxel y media de vecinos (posición relativa) |

### 3.2 Parámetros de Hillshade

```
Azimut del sol (luz):   315° (noroeste — estándar cartográfico)
Altitud del sol:         45° (estándar) / 30° (relieve suave) / 60° (montaña)
Factor Z (exageración): 1.0 (sin exageración) / 2.0–5.0 (para relieve suave)
Multidireccional:        No (estándar) / Sí (para visualización más rica)
```

### 3.3 Parámetros de Slope

```
Unidades de salida: Grados (0–90°) → para análisis de estabilidad de laderas
                    Porcentaje (%) → para agricultura, ingeniería civil
Algoritmo:          Horn (1981) — estándar GDAL, recomendado para la mayoría de casos
```

### 3.4 Parámetros de Aspect

```
Rango de salida: 0–360° (azimut geográfico, Norte = 0/360°)
Zonas planas:    -9999 (sin orientación definida — asignar NoData)
Convención:      0° = Norte, 90° = Este, 180° = Sur, 270° = Oeste
```

### 3.5 Secuencia de Ejecución

```
[DEM / MDT + AOI]
       │
       ▼
Paso 0a: Recortar al AOI
       │
       ▼
Paso 0b: Reprojectar a CRS métrico (si no lo está)
       │
       ▼
Paso 0c: Rellenar huecos NoData (si los hay)
       │
       ├─────────────────────────────────────────────────────┐
       │                                                     │
       ├──────────────────────────────────┐                  │
       │                                  │                  │
       ▼                                  ▼                  ▼
Paso 1: Hillshade               Paso 2: Slope       Paso 3: Aspect
       │                                  │                  │
       │                         Paso 8: Clasif.    Paso 8: Clasif.
       │                         slope (clases)     aspect (octantes)
       │                                  │                  │
       └──────────────┬───────────────────┘                  │
                      │                                      │
              ┌───────┴──────┐                               │
              ▼              ▼                               │
       Paso 4: Roughness   Paso 5: TPI                       │
              │              │                               │
              ▼              ▼                               │
       Paso 6: TRI    Paso 7: Estadísticas ◄─────────────────┘
              │
              ▼
[Hillshade / Slope / Aspect / Roughness / TPI / TRI + clases temáticas]
```

---

## 4. Parámetros Clave

### 4.1 Tabla de parámetros

| Parámetro              | Valor sugerido | Rango válido   | Impacto en el resultado                                           |
|------------------------|----------------|----------------|-------------------------------------------------------------------|
| Azimut hillshade       | 315°           | 0–360°         | Cambia la dirección de iluminación. 315° (NO) es el estándar cartográfico mundial |
| Altitud solar hillshade| 45°            | 15–70°         | Ángulo bajo realza el relieve; ángulo alto lo suaviza            |
| Factor Z (hillshade)   | 1.0            | 0.5–10.0       | Exagera verticalmente el relieve. En zonas planas usar 2–5       |
| Unidad slope           | Grados         | Grados / %     | Grados para análisis geomorfológico; % para usos de suelo        |
| Hillshade multidirec.  | No             | Sí / No        | Multidireccional elimina zonas de sombra pero pierde profundidad |

### 4.2 Variables críticas

> Claude/Antigravity/Gemini **no puede omitir** estos parámetros. Deben confirmarse o justificarse.

- **Factor Z en hillshade:** En relieves de gran altitud (Andes) usar Z=1. En llanuras
  o relieves suaves (costa, selva baja) usar Z=2 a 5 para que el relieve sea visible.
  Sin ajustar, el hillshade de zonas planas aparece completamente gris sin detalle.
- **CRS métrico:** Es la verificación más crítica de todo el proceso. El slope calculado
  sobre un DEM en grados decimales estará en unidades de grados/grado, completamente
  inútil para cualquier análisis cuantitativo.
- **Unidades del slope:** Confirmar con el usuario si necesita grados o porcentaje
  antes de procesar. Tienen usos muy distintos y no se debe asumir.

---

## 5. Salidas Esperadas

| # | Nombre de la capa         | Tipo   | Cargar al proyecto | Descripción                                            |
|---|---------------------------|--------|--------------------|--------------------------------------------------------|
| 1 | `Hillshade_az315_alt45`   | Raster | ✅ Sí              | Sombreado de relieve. Valores 0–255 (gris)             |
| 2 | `Slope_grados`            | Raster | ✅ Sí              | Pendiente en grados. Rango: 0–90°                      |
| 3 | `Slope_clasificado`       | Raster | ✅ Sí              | Mapa temático de clases de pendiente                   |
| 4 | `Aspect_azimut`           | Raster | ✅ Sí              | Orientación de ladera. Rango: 0–360°                   |
| 5 | `Aspect_octantes`         | Raster | ✅ Sí              | Aspect clasificado en 8 orientaciones + plano          |
| 6 | `Roughness`               | Raster | ✅ Sí              | Rugosidad local (máx−mín vecindad 3×3)                |
| 7 | `TPI`                     | Raster | ⚠️ Opcional        | Posición topográfica relativa                          |
| 8 | `TRI`                     | Raster | ⚠️ Opcional        | Índice de rugosidad de Riley (1999)                    |
| 9 | DEM procesado (sin huecos)| Raster | ❌ No (intermedia) | DEM con huecos rellenados, uso interno                 |

---

## 6. Lógica de Interpretación para Claude/Antigravity/Gemini

### 6.1 Reglas de interpretación — Slope (pendiente)

```
SI slope_medio > 45°
  → "Relieve muy escarpado. Laderas con pendientes superiores a 45° indican
     escarpes o paredes rocosas. Alto riesgo de procesos de remoción en masa."

SI slope_medio entre 30° y 45°
  → "Relieve escarpado. Laderas fuertemente inclinadas, típico de quebradas
     andinas o flancos de volcanes. Suelos superficiales con alta susceptibilidad
     a deslizamientos."

SI slope_medio entre 15° y 30°
  → "Relieve moderadamente inclinado. Ladera típica de sierra media. Apta para
     uso agrícola en terrazas (andenes). Riesgo moderado de erosión."

SI slope_medio entre 5° y 15°
  → "Relieve suavemente ondulado. Zonas de piedemonte o colinas bajas. Apto
     para agricultura sin mayor restricción de pendiente."

SI slope_medio < 5°
  → "Relieve casi plano o planicie. Fondos de valle, llanuras aluviales o
     altiplano. Posible acumulación de agua en época de lluvias."
```

### 6.2 Reglas de interpretación — Aspect (orientación)

```
SI aspect dominante entre 315° y 45° (Norte)
  → "Laderas de exposición norte. En el hemisferio sur reciben más radiación
     solar directa → suelos más secos, vegetación xerófila."

SI aspect dominante entre 135° y 225° (Sur)
  → "Laderas de exposición sur. En el hemisferio sur reciben menos radiación
     directa → suelos más húmedos, mayor cobertura vegetal, heladas más frecuentes."

SI aspect dominante entre 45° y 135° (Este)
  → "Laderas de exposición este. Reciben radiación en las mañanas.
     Microcima más templado que las laderas oeste."

SI aspect dominante entre 225° y 315° (Oeste)
  → "Laderas de exposición oeste. Reciben radiación en las tardes.
     En zonas andinas pueden ser más secas por efecto de sombra de lluvia."

SI zona extensa con aspect = -9999 o NoData
  → "Zona de pendiente casi nula (< 1°). No tiene orientación definida.
     Corresponde a planicie, cima plana o fondo de valle."
```

### 6.3 Reglas de interpretación — Roughness / TRI

```
SI roughness_medio alto (> 50 m en DEM de 30 m)
  → "Terreno muy irregular. Alta variabilidad local de elevación. Indica
     presencia de crestas, cárcavas o morfología kárstica/volcánica."

SI roughness_medio moderado (10–50 m)
  → "Terreno moderadamente rugoso. Relieve colinoso o de sierra media."

SI roughness_medio bajo (< 10 m)
  → "Terreno suave. Planicies, valles o superficies glaciares. Baja
     complejidad morfológica local."

SI TPI > 0 (positivo)
  → "Posición topográfica elevada: cresta, cima o interfluvio. El terreno
     está por encima de su entorno inmediato."

SI TPI ≈ 0 (cerca de cero)
  → "Ladera media o terreno en pendiente uniforme sin posición relativa
     dominante."

SI TPI < 0 (negativo)
  → "Posición topográfica baja: fondo de valle, cauce, depresión o cuenca
     cerrada. Zona de convergencia de flujos."
```

### 6.4 Flags de error

| Síntoma observado                          | Causa probable                                 | Acción correctiva                                               |
|--------------------------------------------|------------------------------------------------|-----------------------------------------------------------------|
| Slope con valores > 89° generalizados      | DEM en coordenadas geográficas (grados)        | Reprojectar DEM a CRS métrico y recalcular                      |
| Hillshade completamente gris uniforme      | Factor Z=1 en relieve muy plano               | Aumentar Factor Z a 3–10 según la escala del relieve            |
| Franjas paralelas o rayas en el slope      | Artefactos de striping en DEM SRTM            | Rellenar huecos con `gdal:fillnodata` y filtrar el DEM          |
| Aspect con grandes zonas en -9999          | Zonas de pendiente < 1° (planos)              | Normal. Asignar clase "Plano" en la clasificación de aspect     |
| TPI todo igual a 0                         | Ventana de vecindad demasiado pequeña         | Aumentar el radio de análisis del TPI                           |
| Roughness con valores extremos en bordes   | Efecto de borde en los límites del raster     | Recortar el AOI dejando un buffer extra de 5–10 píxeles         |
| NoData en bordes del hillshade/slope       | DEM sin buffer perimetral                     | Usar DEM ligeramente mayor al AOI antes de recortar la salida   |

---

## 7. Optimización y Visualización

### 7.1 Tips de rendimiento

- **Calcular en paralelo:** Hillshade, Slope, Aspect y Roughness son independientes
  entre sí — pueden ejecutarse en paralelo usando la Caja de Herramientas de Procesos
  en lote (Batch Processing) de QGIS.
- **Resolución adecuada:** Para visualización general, SRTM 30 m es suficiente. Para
  análisis de riesgo o agrícola, usar DEM ≤ 10 m. No sobre-precisar si el DEM es grueso.
- **Buffer perimetral:** Siempre recortar el DEM con un buffer de 5–10 píxeles extra
  respecto al AOI final. Esto evita los efectos de borde en todas las derivadas.
  Recortar las salidas finales al AOI exacto al terminar.

### 7.2 Preprocesamiento recomendado

1. **Rellenar huecos del DEM:** Especialmente para SRTM (tiene huecos en zonas de agua
   y alta pendiente). Usar `gdal:fillnodata` con radio de interpolación adecuado (5–20 px).
2. **Suavizado opcional:** Para DEMs muy ruidosos (ej: ASTER GDEM), aplicar un filtro
   de suavizado gaussiano suave antes de derivar slope y aspect. Reduce el ruido sin
   eliminar el relieve real.
3. **Verificar con hillshade primero:** Siempre generar el hillshade como primer paso.
   Es la forma más rápida de detectar artefactos, huecos, errores de NoData y
   problemas de CRS antes de calcular las demás derivadas.

### 7.3 Simbología sugerida

| Capa                   | Tipo de rampa  | Val. mín. | Val. máx. | Paleta sugerida                              |
|------------------------|----------------|-----------|-----------|----------------------------------------------|
| `Hillshade_...`        | Gris continuo  | 0         | 255       | Grises (negro→blanco). Transparencia 40–60%  |
| `Slope_grados`         | Continua       | 0°        | 60°       | YlOrRd (amarillo→rojo)                       |
| `Slope_clasificado`    | Clasificada    | —         | —         | Ver tabla de clases abajo                    |
| `Aspect_azimut`        | Cíclica        | 0°        | 360°      | Aspecto (HSV circular — rosa de los vientos) |
| `Aspect_octantes`      | Clasificada    | —         | —         | Colores por punto cardinal                   |
| `Roughness`            | Continua       | 0         | P95       | Magma / Viridis (oscuro→claro)               |
| `TPI`                  | Divergente     | -max      | +max      | RdBu (rojo=cresta, azul=depresión)           |
| `TRI`                  | Continua       | 0         | P95       | Plasma                                       |

> **Tip de visualización profesional:** Superponer el Hillshade (transparencia 50%)
> sobre cualquier capa temática (Slope, NDVI, etc.) añade sensación de profundidad
> 3D al mapa sin necesitar software adicional. Es el truco más usado en cartografía
> geomorfológica.

**Tabla de clasificación de Slope:**

| Clase | Rango (°)    | Rango (%)    | Color sugerido | Descripción geomorfológica          |
|-------|-------------|--------------|----------------|-------------------------------------|
| 1     | 0° – 2°     | 0 – 3.5%     | Blanco         | Plano / Sin pendiente               |
| 2     | 2° – 5°     | 3.5 – 8.7%   | Verde claro    | Casi plano / Suavemente inclinado   |
| 3     | 5° – 15°    | 8.7 – 26.8%  | Amarillo       | Inclinado / Ondulado                |
| 4     | 15° – 30°   | 26.8 – 57.7% | Naranja        | Fuertemente inclinado               |
| 5     | 30° – 45°   | 57.7 – 100%  | Rojo           | Escarpado                           |
| 6     | > 45°       | > 100%       | Rojo oscuro    | Muy escarpado / Escarpe             |

**Tabla de clasificación de Aspect (octantes):**

| Clase | Rango (°)    | Orientación | Color sugerido |
|-------|-------------|-------------|----------------|
| 0     | —           | Plano       | Gris           |
| 1     | 337.5–360° / 0–22.5° | Norte (N)  | Azul oscuro |
| 2     | 22.5–67.5°  | Noreste (NE)| Azul claro     |
| 3     | 67.5–112.5° | Este (E)    | Verde          |
| 4     | 112.5–157.5°| Sureste (SE)| Amarillo       |
| 5     | 157.5–202.5°| Sur (S)     | Naranja        |
| 6     | 202.5–247.5°| Suroeste (SO)| Rojo          |
| 7     | 247.5–292.5°| Oeste (O)   | Púrpura        |
| 8     | 292.5–337.5°| Noroeste (NO)| Rosa          |

---

## 8. Referencias y Fuentes

- Horn, B.K.P., 1981. *Hill Shading and the Reflectance Map.* Proceedings of the IEEE.
  (Algoritmo estándar de slope/aspect — base de GDAL)
- Riley, S.J., DeGloria, S.D. & Elliot, R., 1999. *A Terrain Ruggedness Index that
  Quantifies Topographic Heterogeneity.* Intermountain Journal of Sciences. (TRI)
- Weiss, A., 2001. *Topographic Position and Landforms Analysis.* ESRI User Conference.
  (TPI y clasificación de formas del terreno)
- GDAL Terrain Analysis Documentation: https://gdal.org/programs/gdaldem.html
- Copernicus DEM (GLO-30): https://spacedata.copernicus.eu/collections/copernicus-digital-elevation-model
- USGS EarthExplorer (SRTM): https://earthexplorer.usgs.gov

---

*Versión: 1.0 — Geomorphology Terrain Skills / Isaias Quintana / MCP-GIS*

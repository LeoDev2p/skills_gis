---
name: gis-severidad-incendios
description: >
  Usar este skill para calcular e interpretar índices de severidad de incendios
  forestales y quemas en QGIS vía MCP. Activar cuando el usuario mencione: NBR,
  dNBR, RdNBR, severidad de incendio, área quemada, cicatriz de incendio,
  incendio forestal, burn severity, mapeo de quemas, detección de áreas
  afectadas por fuego, evaluación de daño por incendio, regeneración post-fuego,
  o cuando trabaje con bandas NIR y SWIR de imágenes pre y post incendio de
  Sentinel-2 o Landsat. No omitir este skill si la tarea involucra cualquier
  análisis antes/después (pre/post) de un evento de fuego o si se busca
  cuantificar el área afectada y clasificar la severidad del daño.
---

# Severidad de Incendios — NBR y dNBR

---

## 1. Misión y Dominio

| Campo         | Valor                                                                                         |
|---------------|-----------------------------------------------------------------------------------------------|
| **Objetivo**  | Mapear y clasificar la severidad de incendios forestales mediante el análisis de cambio espectral pre/post evento usando los índices NBR y dNBR. |
| **Dominio**   | Teledetección / Ecología del Fuego / Gestión de Riesgos / Conservación                       |
| **Escala**    | Regional (Sentinel-2: 20 m) / Regional-Global (Landsat 8/9: 30 m)                            |
| **Nivel**     | Avanzado                                                                                      |

---

## 2. Insumos Requeridos

### 2.1 Capas necesarias

| # | Capa                      | Tipo   | Contenido                                        | Resolución sugerida | Obligatoria       |
|---|---------------------------|--------|--------------------------------------------------|---------------------|-------------------|
| 1 | Banda NIR — PRE incendio  | Raster | B08 (Sentinel-2) / B5 (Landsat 8/9) — fecha anterior al evento | 10–30 m | ✅ Sí |
| 2 | Banda SWIR-2 — PRE incendio | Raster | B12 (Sentinel-2) / B7 (Landsat 8/9) — fecha anterior       | 20–30 m | ✅ Sí |
| 3 | Banda NIR — POST incendio | Raster | B08 (Sentinel-2) / B5 (Landsat 8/9) — fecha posterior al evento | 10–30 m | ✅ Sí |
| 4 | Banda SWIR-2 — POST incendio | Raster | B12 (Sentinel-2) / B7 (Landsat 8/9) — fecha posterior    | 20–30 m | ✅ Sí |
| 5 | Máscara de nubes PRE      | Raster | SCL (Sentinel-2) / QA_PIXEL (Landsat)           | Misma que bandas    | ⚠️ Recomendada    |
| 6 | Máscara de nubes POST     | Raster | SCL (Sentinel-2) / QA_PIXEL (Landsat)           | Misma que bandas    | ⚠️ Recomendada    |
| 7 | Límite área de estudio    | Vector | Polígono del AOI o perímetro del incendio        | —                   | ⚠️ Recomendada    |

> **Correspondencia de bandas por sensor:**
>
> | Banda  | Sentinel-2      | Landsat 8/9  |
> |--------|-----------------|--------------|
> | NIR    | B08 (10 m)      | B5 (30 m)    |
> | SWIR-2 | B12 (20 m)      | B7 (30 m)    |
>
> ⚠️ **Importante:** NBR usa SWIR-2 (no SWIR-1). En Sentinel-2 es B12, no B11.

> **Ventana temporal recomendada:**
>
> | Imagen    | Timing ideal                                                    |
> |-----------|-----------------------------------------------------------------|
> | PRE       | 15–60 días antes del incendio, misma época del año si es posible |
> | POST      | 15–30 días después de extinguido el fuego (no durante el humo)   |

### 2.2 Validación Pre-proceso ⚠️ CRÍTICO

> Claude/Antigravity/Gemini **debe verificar todos estos puntos** antes de ejecutar. El análisis
> de cambio pre/post es muy sensible a diferencias fenológicas, de escala y
> de corrección atmosférica entre imágenes.

- [ ] **Nivel de procesamiento y factor de escala:** ⚠️ LEER ANTES DE CALCULAR
  Claude/Antigravity/Gemini debe preguntar al usuario qué sensor y nivel de producto tiene, o pedirle
  que revise el archivo de metadatos. **AMBAS imágenes (PRE y POST) deben ser
  del mismo nivel y mismo sensor.**

  | Sensor | Nivel | ¿Usar? | Fórmula de escalado | Resultado |
  |--------|-------|--------|---------------------|-----------|
  | Sentinel-2 | **L2A** | ✅ Sí | `DN / 10000` | Reflectancia 0–1 |
  | Sentinel-2 | **L1C** | ❌ No usar directo | Requiere corrección atmosférica (Sen2Cor) | TOA, no válido |
  | Landsat 8/9 | **Collection 2 Level-2** | ✅ Sí | `(DN × 0.0000275) − 0.2` | Reflectancia 0–1 |
  | Landsat 8/9 | **Collection 2 Level-1** | ❌ No usar directo | Requiere corrección atmosférica | TOA, no válido |

  > **¿Cómo saber el nivel?**
  > - **Landsat:** archivo `*_MTL.txt` → campo `PROCESSING_LEVEL`. Si dice `L2SP` → Level-2 ✅
  > - **Sentinel-2:** nombre del producto → si contiene `L2A` → Level-2 ✅ / `L1C` → ❌
  > - Si el usuario no tiene el MTL, pedirle el nombre completo del archivo — contiene el nivel.
- [ ] **CRS:** Ambas imágenes deben estar en el mismo CRS métrico (UTM).
  Si la zona UTM cambia entre fechas → reprojectar a la misma.
- [ ] **Resolución uniforme:** NIR (10 m) y SWIR-2 (20 m) en Sentinel-2 son distintas.
  Remuestrear SWIR-2 a 10 m (o NIR a 20 m) antes de calcular NBR.
- [ ] **Cobertura de nubes:** Verificar que el área del incendio esté libre de nubes
  en AMBAS imágenes. Una nube en la imagen POST produce falsos de alta severidad.
- [ ] **Factor de escala según sensor:** Aplicar la fórmula correcta antes de calcular NBR.
  Sentinel-2 L2A → `÷10000`. Landsat Collection 2 Level-2 → `×0.0000275 − 0.2` (con offset).
- [ ] **Fecha razonable:** Confirmar que la imagen POST es realmente posterior al evento.
  Verificar con el usuario las fechas exactas antes de procesar.

---

## 3. Flujo de Trabajo

### 3.1 Diccionario de Algoritmos

| Paso | ID Técnico QGIS                       | Herramienta | Descripción                                              |
|------|---------------------------------------|-------------|----------------------------------------------------------|
| 0a   | `gdal:cliprasterbymasklayer`          | GDAL        | Recortar todas las bandas al AOI                         |
| 0b   | `gdal:warpreproject`                  | GDAL        | Homogenizar resolución NIR y SWIR-2 (remuestrear)       |
| 0c   | `gdal:rastercalculator`               | GDAL        | Escalar reflectancias según sensor (ver sección 3.2)     |
| 1    | `gdal:rastercalculator`               | GDAL        | Calcular NBR_pre = (NIR_pre − SWIR2_pre)/(NIR_pre + SWIR2_pre) |
| 2    | `gdal:rastercalculator`               | GDAL        | Calcular NBR_post = (NIR_post − SWIR2_post)/(NIR_post + SWIR2_post) |
| 3    | `gdal:rastercalculator`               | GDAL        | Calcular dNBR = NBR_pre − NBR_post                       |
| 4    | `native:reclassifybytable`            | QGIS        | Clasificar dNBR en niveles de severidad (USGS)           |
| 5    | `gdal:polygonize`                     | GDAL        | Vectorizar clases de severidad                           |
| 6    | `native:dissolve`                     | QGIS        | Disolver polígonos por clase de severidad                |
| 7    | `native:fieldcalculator`              | QGIS        | Calcular área de cada clase en hectáreas                 |
| 8    | `native:rasterlayerstatistics`        | QGIS        | Estadísticas de NBR_pre, NBR_post y dNBR                 |

### 3.2 Fórmulas exactas para `gdal:rastercalculator`

> ⚠️ **Aplicar el escalado correcto según sensor ANTES de calcular NBR.**
> Mezclar fórmulas entre sensores o usar imagen PRE con distinto nivel que POST invalida el dNBR.

```
# ── SENTINEL-2 L2A ─────────────────────────────────────────────────
# Factor de escala: DN / 10000
NIR_pre_s2    = "B08_pre@1"    / 10000.0
SWIR2_pre_s2  = "B12_pre@1"   / 10000.0
NIR_post_s2   = "B08_post@1"   / 10000.0
SWIR2_post_s2 = "B12_post@1"  / 10000.0

# ── LANDSAT 8/9 Collection 2 Level-2 ───────────────────────────────
# Factor de escala: (DN × 0.0000275) − 0.2
# ⚠️ El offset −0.2 es OBLIGATORIO. Sin él el NBR estará desplazado
#    y el dNBR producirá clasificaciones de severidad incorrectas.
NIR_pre_ls    = ("B5_pre@1"  * 0.0000275) - 0.2
SWIR2_pre_ls  = ("B7_pre@1"  * 0.0000275) - 0.2
NIR_post_ls   = ("B5_post@1" * 0.0000275) - 0.2
SWIR2_post_ls = ("B7_post@1" * 0.0000275) - 0.2

# ── ÍNDICES (usar las variables escaladas del sensor correspondiente) ─
# NBR pre-incendio:
("NIR_pre@1" - "SWIR2_pre@1") / ("NIR_pre@1" + "SWIR2_pre@1")

# NBR post-incendio:
("NIR_post@1" - "SWIR2_post@1") / ("NIR_post@1" + "SWIR2_post@1")

# dNBR (diferencia de cambio):
"NBR_pre@1" - "NBR_post@1"

# Nota: dNBR positivo = área quemada. dNBR negativo = posible
# revegetación o zona no afectada.
```

### 3.3 Tabla de clasificación dNBR (estándar USGS/Key & Benson 2006)

| Clase | Rango dNBR        | Nivel de severidad             |
|-------|-------------------|--------------------------------|
| 1     | < -0.100          | Mayor verdor post-fuego (no quemado / revegetación) |
| 2     | -0.100 – 0.099    | Sin quema / No afectado        |
| 3     | 0.100 – 0.269     | Severidad baja                 |
| 4     | 0.270 – 0.439     | Severidad baja-moderada        |
| 5     | 0.440 – 0.659     | Severidad moderada-alta        |
| 6     | 0.660 – 1.300     | Severidad alta                 |
| 7     | > 1.300           | Severidad muy alta / extrema   |

### 3.4 Secuencia de Ejecución

```
[Bandas NIR_pre, SWIR2_pre, NIR_post, SWIR2_post + AOI]
                      │
                      ▼
        Paso 0a: Recortar todas las bandas al AOI
                      │
                      ▼
        Paso 0b: Homogenizar resolución (NIR ↔ SWIR-2)
                      │
                      ▼
        Paso 0c: Escalar si son enteros (÷10000)
                      │
              ┌───────┴───────┐
              ▼               ▼
    Paso 1: NBR_pre   Paso 2: NBR_post
              │               │
              └───────┬───────┘
                      ▼
            Paso 3: dNBR = NBR_pre − NBR_post
                      │
                      ▼
        Paso 4: Reclasificación → Mapa de severidad
                      │
                      ▼
        Paso 5: Vectorizar clases de severidad
                      │
                      ▼
        Paso 6: Disolver por clase
                      │
                      ▼
        Paso 7: Calcular áreas (hectáreas)
                      │
                      ▼
    [NBR_pre / NBR_post / dNBR / Mapa_severidad / Tabla_areas]
```

---

## 4. Parámetros Clave

### 4.1 Tabla de parámetros

| Parámetro                | Valor sugerido | Rango válido | Impacto en el resultado                                         |
|--------------------------|----------------|--------------|-----------------------------------------------------------------|
| Umbral mínimo de quema (dNBR) | 0.10      | 0.05 – 0.20  | Subir reduce falsos positivos pero puede omitir quemas leves    |
| Resolución de trabajo    | 20 m           | 10–30 m      | A 10 m más detalle pero mayor tiempo de cómputo                |
| Método de remuestreo SWIR | Bilineal      | —            | No usar vecino más cercano para bandas continuas de reflectancia |

### 4.2 Variables críticas

> Claude/Antigravity/Gemini **no puede asumir** las fechas de las imágenes. Debe confirmar con el
> usuario la fecha exacta del incendio y las fechas de adquisición de ambas imágenes.

- **Fechas PRE y POST:** La elección incorrecta de fechas es el error más común.
  Una imagen "post" tomada durante el humo activo producirá dNBR inválido. Confirmar
  siempre con el usuario.
- **Misma época estacional:** Idealmente PRE y POST deben ser de la misma temporada
  del año (ej: ambas en temporada seca) para evitar que la variación fenológica se
  confunda con el daño del incendio.
- **Banda SWIR-2, no SWIR-1:** El NBR usa SWIR-2 (B12 en Sentinel-2, B7 en Landsat).
  Usar SWIR-1 (B11/B6) produce resultados incorrectos. Verificar siempre.

---

## 5. Salidas Esperadas

| # | Nombre de la capa              | Tipo   | Cargar al proyecto | Descripción                                              |
|---|--------------------------------|--------|--------------------|----------------------------------------------------------|
| 1 | `NBR_pre_[fecha]`              | Raster | ✅ Sí              | NBR pre-incendio. Rango típico: -1 a 1                   |
| 2 | `NBR_post_[fecha]`             | Raster | ✅ Sí              | NBR post-incendio. Rango típico: -1 a 1                  |
| 3 | `dNBR_[fecha_pre]_[fecha_post]`| Raster | ✅ Sí              | Diferencia de cambio. Positivo = quemado                 |
| 4 | `Severidad_incendio`           | Raster | ✅ Sí              | Mapa temático clasificado en niveles de severidad        |
| 5 | `Perimetro_quema_clases`       | Vector | ✅ Sí              | Polígonos por nivel de severidad con área en ha          |
| 6 | Bandas escaladas               | Raster | ❌ No (intermedia) | Bandas en reflectancia 0–1, uso interno                  |

---

## 6. Lógica de Interpretación para Claude/Antigravity/Gemini

### 6.1 Reglas de interpretación

```
SI dNBR_media del área total > 0.66
  → "Incendio de alta severidad dominante. La mayor parte de la vegetación
     fue consumida. Se espera mortalidad arbórea elevada y suelo mineral expuesto."

SI dNBR_media entre 0.44 y 0.66
  → "Severidad moderada a alta. Afectación significativa del dosel. Probable
     mortalidad parcial de árboles con regeneración natural posible."

SI dNBR_media entre 0.27 y 0.44
  → "Severidad baja a moderada. El fuego afectó principalmente el sotobosque.
     Los árboles adultos probablemente sobrevivieron."

SI dNBR_media entre 0.10 y 0.27
  → "Severidad baja. Quema superficial. Vegetación herbácea y hojarasca
     afectada. El dosel arbóreo presenta daño mínimo."

SI dNBR_media < 0.10
  → "Sin evidencia de quema o área no afectada. Verificar que las fechas
     de imagen sean correctas."

SI hay píxeles con dNBR < -0.10 dentro del perímetro esperado
  → "Posible error de enmascarado de nubes en imagen POST, o zonas con
     revegetación activa inusualmente rápida. Verificar con imagen RGB."

SI NBR_pre tiene valores anómalos muy bajos (< 0) en vegetación
  → "La imagen PRE puede tener nubes no enmascaradas. El dNBR resultante
     no es confiable en esas zonas."
```

### 6.2 Reporte de áreas — Claude/Antigravity/Gemini debe comunicar siempre

Al finalizar, Claude/Antigravity/Gemini debe calcular y comunicar:
- Área total quemada (dNBR ≥ 0.10) en **hectáreas**
- Distribución porcentual por clase de severidad
- Clase de severidad dominante
- Fecha PRE y fecha POST usadas en el análisis

### 6.3 Flags de error

| Síntoma observado                         | Causa probable                                   | Acción correctiva                                               |
|-------------------------------------------|--------------------------------------------------|-----------------------------------------------------------------|
| dNBR completamente negativo               | Imágenes PRE y POST invertidas                   | Intercambiar imágenes y recalcular                              |
| dNBR con valores >> 2 o << -2             | Bandas no escaladas o sensor incorrecto          | Verificar: Sentinel-2 → `÷10000`; Landsat → `×0.0000275 − 0.2`. Revisar bandas (NIR vs SWIR-2) |
| Parches de alta severidad en zonas sin quema | Nubes en imagen POST no enmascaradas          | Aplicar máscara de nubes POST y recalcular                      |
| Área quemada igual a 0 ha                 | Umbral demasiado alto o imágenes incorrectas     | Bajar umbral a 0.05 y verificar fechas de imágenes              |
| NBR_pre y NBR_post idénticos              | Se usó la misma imagen para pre y post          | Verificar que las rutas de archivo sean distintas               |
| Error de alineación en rastercalculator   | CRS o extensión diferente entre imágenes         | Reprojectar y recortar a extensión común antes de calcular      |

---

## 7. Optimización y Visualización

### 7.1 Tips de rendimiento

- **Trabajar a 20 m:** Para Sentinel-2, trabajar con SWIR-2 (20 m) como resolución
  base y remuestrear NIR a 20 m. Evita imágenes excesivamente grandes y es suficiente
  para mapeo de severidad.
- **Recortar al perímetro estimado:** Si se conoce el perímetro aproximado del incendio
  (de noticias, FIRMS/VIIRS, o inspección visual), recortar a ese AOI antes de procesar.
- **FIRMS para orientación:** Usar el servicio FIRMS de NASA (firms.modaps.eosdis.nasa.gov)
  para verificar la ubicación y fecha del incendio antes de seleccionar imágenes.

### 7.2 Preprocesamiento recomendado

1. **Verificar fechas con FIRMS/VIIRS:** Antes de seleccionar imágenes, consultar
   los puntos de calor activos para confirmar la fecha y extensión del incendio.
2. **Enmascarar nubes en AMBAS imágenes:** Especialmente en la imagen POST — las nubes
   producen valores de dNBR extremadamente altos (falsos de severidad alta).
3. **Selección de imagen PRE de la misma temporada:** Evitar comparar una imagen en
   época lluviosa (vegetación verde) con una POST en época seca (vegetación seca
   sin incendio) — esto produciría dNBR positivo sin que haya habido fuego real.

### 7.3 Simbología sugerida

| Capa                    | Tipo de rampa | Val. mín. | Val. máx. | Paleta sugerida                          |
|-------------------------|---------------|-----------|-----------|------------------------------------------|
| `NBR_pre_[fecha]`       | Continua      | -0.5      | 1.0       | RdYlGn (verde = vegetación sana)         |
| `NBR_post_[fecha]`      | Continua      | -0.5      | 1.0       | RdYlGn                                   |
| `dNBR_[...]`            | Continua      | -0.5      | 1.3       | YlOrRd (amarillo → rojo oscuro)          |
| `Severidad_incendio`    | Clasificada   | —         | —         | Ver tabla de colores abajo               |

**Tabla de colores por clase de severidad (estándar USGS):**

| Clase | Nivel                    | Color HEX   | Descripción            |
|-------|--------------------------|-------------|------------------------|
| 1     | Mayor verdor             | `#006400`   | Verde oscuro           |
| 2     | Sin quema / No afectado  | `#90EE90`   | Verde claro            |
| 3     | Severidad baja           | `#FFFF00`   | Amarillo               |
| 4     | Severidad baja-moderada  | `#FFA500`   | Naranja                |
| 5     | Severidad moderada-alta  | `#FF4500`   | Rojo-naranja           |
| 6     | Severidad alta           | `#8B0000`   | Rojo oscuro            |
| 7     | Severidad muy alta       | `#4B0082`   | Violeta/Morado         |

---

## 8. Referencias y Fuentes

- Key, C.H. & Benson, N.C., 2006. *Landscape Assessment: Ground measure of severity,
  the Composite Burn Index.* FIREMON: Fire Effects Monitoring and Inventory System.
  USDA Forest Service. (Clasificación dNBR estándar)
- Miller, J.D. & Thode, A.E., 2007. *Quantifying burn severity in a heterogeneous
  landscape with a relative version of the delta Normalized Burn Ratio (RdNBR).*
  Remote Sensing of Environment.
- NASA FIRMS (Fire Information for Resource Management System): https://firms.modaps.eosdis.nasa.gov
- ESA Sentinel-2 User Handbook: https://sentinel.esa.int
- USGS Landsat Burned Area Products: https://www.usgs.gov/landsat-missions

---

*Versión: 1.0 — Spectral Analysis Skills / Isaias Quintana / MCP-GIS*

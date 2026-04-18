---
name: gis-dinamica-hidrica
description: >
  Usar este skill para calcular e interpretar índices espectrales de agua y
  humedad en QGIS vía MCP. Activar cuando el usuario mencione: NDWI, MNDWI,
  índice de agua, cuerpos de agua, lagunas, ríos, humedales, humedad del suelo,
  inundaciones, detección de agua superficial, mapeo de espejo de agua,
  variación de nivel lacustre, retroceso glaciar, monitoreo de embalses, sequías
  hídricas, o cuando trabaje con bandas Verde (Green), NIR y SWIR de Sentinel-2
  o Landsat. No omitir este skill si la tarea implica detectar, mapear o
  cuantificar agua o humedad a partir de imágenes satelitales multiespectrales.
---

# Dinámica Hídrica — NDWI y MNDWI

---

## 1. Misión y Dominio

| Campo         | Valor                                                                                       |
|---------------|---------------------------------------------------------------------------------------------|
| **Objetivo**  | Detectar, delimitar y monitorear cuerpos de agua superficial y zonas de alta humedad mediante índices espectrales de agua derivados de reflectancia superficial. |
| **Dominio**   | Teledetección / Hidrología / Gestión de Recursos Hídricos                                   |
| **Escala**    | Regional (Sentinel-2: 10–20 m) / Regional-Global (Landsat 8/9: 30 m)                       |
| **Nivel**     | Intermedio                                                                                  |

---

## 2. Insumos Requeridos

### 2.1 Capas necesarias

| # | Capa                   | Tipo   | Contenido                                       | Resolución sugerida | Obligatoria      |
|---|------------------------|--------|-------------------------------------------------|---------------------|------------------|
| 1 | Banda Verde (Green)    | Raster | B03 (Sentinel-2) / B3 (Landsat 8/9)            | 10–30 m             | ✅ Sí            |
| 2 | Banda NIR              | Raster | B08 (Sentinel-2) / B5 (Landsat 8/9)            | 10–30 m             | ✅ Solo NDWI     |
| 3 | Banda SWIR-1           | Raster | B11 (Sentinel-2, 20m) / B6 (Landsat 8/9, 30m) | 20–30 m             | ✅ Solo MNDWI    |
| 4 | Máscara de nubes       | Raster | SCL (Sentinel-2) / QA_PIXEL (Landsat)          | Misma que bandas    | ⚠️ Recomendada   |
| 5 | Límite área de estudio | Vector | Polígono del área de interés (AOI)             | —                   | ⚠️ Recomendada   |

> **Correspondencia de bandas por sensor:**
>
> | Índice | Sentinel-2        | Landsat 8/9 |
> |--------|-------------------|-------------|
> | Verde  | B03               | B3          |
> | NIR    | B08               | B5          |
> | SWIR-1 | B11 (20 m)        | B6 (30 m)   |

> **¿Cuándo usar NDWI vs MNDWI?**
>
> | Índice | Mejor para                                         | Limitación                                    |
> |--------|----------------------------------------------------|-----------------------------------------------|
> | NDWI   | Cuerpos de agua abiertos, lagunas grandes          | Confunde agua con vegetación húmeda           |
> | MNDWI  | Agua en zonas urbanas, ríos estrechos, humedales   | Requiere banda SWIR (20–30 m, menor detalle)  |

### 2.2 Validación Pre-proceso ⚠️ CRÍTICO

> Claude/Antigravity/Gemini **debe verificar todos estos puntos** antes de ejecutar. Índices de
> agua calculados con bandas mal escaladas o con resolución inconsistente
> producen delimitaciones incorrectas sin advertencia visible.

- [ ] **Nivel de procesamiento y factor de escala:** ⚠️ LEER ANTES DE CALCULAR
  Claude/Antigravity/Gemini debe preguntar al usuario qué sensor y nivel de producto tiene, o pedirle
  que revise el archivo de metadatos (`MTL.txt` en Landsat / `MTD_MSIL2A.xml` en Sentinel-2).

  | Sensor | Nivel | ¿Usar? | Fórmula de escalado | Resultado |
  |--------|-------|--------|---------------------|-----------|
  | Sentinel-2 | **L2A** | ✅ Sí | `DN / 10000` | Reflectancia 0–1 |
  | Sentinel-2 | **L1C** | ❌ No usar directo | Requiere corrección atmosférica (Sen2Cor) | TOA, no válido |
  | Landsat 8/9 | **Collection 2 Level-2** | ✅ Sí | `(DN × 0.0000275) − 0.2` | Reflectancia 0–1 |
  | Landsat 8/9 | **Collection 2 Level-1** | ❌ No usar directo | Requiere corrección atmosférica | TOA, no válido |

  > **¿Cómo saber el nivel?** Pedir al usuario que revise:
  > - **Landsat:** archivo `*_MTL.txt` → campo `PROCESSING_LEVEL`. Si dice `L2SP` → Level-2 ✅
  > - **Sentinel-2:** nombre del archivo → si contiene `L2A` → Level-2 ✅ / `L1C` → TOA ❌
- [ ] **CRS:** Debe ser métrico (UTM). Si está en EPSG:4326 → reprojectar primero.
- [ ] **Resolución uniforme:** NDWI usa NIR (10 m) y Green (10 m): compatible.
  MNDWI usa SWIR-1 (20 m) y Green (10 m): **remuestrear Green a 20 m** o SWIR a 10 m.
- [ ] **NoData:** Definir correctamente antes de calcular (usualmente 0).
- [ ] **Nubes:** Enmascarar antes — las nubes producen valores de agua falsos positivos.
- [ ] **Sombras de nubes:** Las sombras de nube también generan falsos positivos de agua.

---

## 3. Flujo de Trabajo

### 3.1 Diccionario de Algoritmos

| Paso | ID Técnico QGIS                       | Herramienta | Descripción                                         |
|------|---------------------------------------|-------------|-----------------------------------------------------|
| 0a   | `gdal:cliprasterbymasklayer`          | GDAL        | Recortar bandas al AOI                              |
| 0b   | `gdal:warpreproject`                  | GDAL        | Homogenizar resolución (ej: SWIR a 10 m)           |
| 0c   | `gdal:rastercalculator`               | GDAL        | Escalar bandas si son enteros (÷10000)              |
| 1    | `gdal:rastercalculator`               | GDAL        | Calcular NDWI = (Green − NIR) / (Green + NIR)      |
| 2    | `gdal:rastercalculator`               | GDAL        | Calcular MNDWI = (Green − SWIR) / (Green + SWIR)   |
| 3    | `gdal:rastercalculator`               | GDAL        | Umbralizar índice → máscara binaria de agua (≥ 0)  |
| 4    | `gdal:polygonize`                     | GDAL        | Convertir máscara raster a polígonos de agua        |
| 5    | `native:deleteholes`                  | QGIS        | Eliminar islas internas menores a área mínima       |
| 6    | `native:rasterlayerstatistics`        | QGIS        | Estadísticas del índice (media, std, min, max)      |

### 3.2 Fórmulas exactas para `gdal:rastercalculator`

> ⚠️ **Aplicar el escalado correcto según sensor ANTES de calcular cualquier índice.**

```
# ── SENTINEL-2 L2A ─────────────────────────────────────────────────
# Factor de escala: DN / 10000
Green_s2 = "B03@1" / 10000.0
NIR_s2   = "B08@1" / 10000.0
SWIR_s2  = "B11@1" / 10000.0

# ── LANDSAT 8/9 Collection 2 Level-2 ───────────────────────────────
# Factor de escala: (DN × 0.0000275) − 0.2
# ⚠️ El offset −0.2 es OBLIGATORIO. Sin él los valores son incorrectos.
Green_ls = ("B3@1" * 0.0000275) - 0.2
NIR_ls   = ("B5@1" * 0.0000275) - 0.2
SWIR_ls  = ("B6@1" * 0.0000275) - 0.2

# ── ÍNDICES (usar las variables escaladas del sensor correspondiente) ─
# NDWI (McFeeters, 1996):
("Green@1" - "NIR@1") / ("Green@1" + "NIR@1")

# MNDWI (Xu, 2006):
("Green@1" - "SWIR@1") / ("Green@1" + "SWIR@1")

# Máscara binaria de agua (umbral 0):
("NDWI@1" >= 0) * 1
# O para MNDWI:
("MNDWI@1" >= 0) * 1
```

### 3.3 Secuencia de Ejecución

```
[Bandas Green, NIR, SWIR + AOI]
          │
          ▼
Paso 0a: Recortar al AOI
          │
          ▼
Paso 0b: Homogenizar resolución (si bandas tienen diferente pixel size)
          │
          ▼
Paso 0c: Escalar reflectancias (si valores > 1)
          │
          ├──────────────────────┬───────────────────────┐
          ▼                      ▼                       │
Paso 1: Calcular NDWI   Paso 2: Calcular MNDWI          │
  (Green, NIR)            (Green, SWIR)                  │
          │                      │                       │
          ▼                      ▼                       │
Paso 3: Umbralizar → Máscara binaria de agua             │
          │                                              │
          ▼                                              │
Paso 4: Vectorizar (Polygonize)                          │
          │                                              │
          ▼                                              │
Paso 5: Limpiar polígonos (DeleteHoles)                  │
          │                                              │
          ▼                                              ▼
Paso 6: Estadísticas del índice              [Capas de índices continuas]
          │
          ▼
[Polígono de cuerpos de agua + índices continuos]
```

---

## 4. Parámetros Clave

### 4.1 Tabla de parámetros

| Parámetro            | Valor sugerido | Rango válido | Impacto en el resultado                                         |
|----------------------|----------------|--------------|-----------------------------------------------------------------|
| Umbral de agua       | 0.0            | -0.3 – 0.3   | Subir el umbral (ej: 0.1) reduce falsos positivos pero pierde ríos estrechos |
| Área mínima polígono | 500 m²         | 100 – 5000 m²| Filtra pequeñas islas y ruido. Ajustar según resolución y escala |
| Resolución de remuestreo | Bilinear   | —            | Usar bilineal para SWIR. No usar vecino más cercano en bandas continuas |

### 4.2 Variables críticas

> Claude/Antigravity/Gemini **no puede asumir el umbral de agua** sin contexto. Debe preguntar al
> usuario o justificar el valor elegido.

- **Umbral de binarización:** El valor 0.0 es estándar pero puede generar falsos
  positivos en zonas de sombra y suelos húmedos. En zonas áridas con suelo muy
  reflectante puede ser necesario subir a 0.1 o 0.15.
- **Elección de índice:** En zonas urbanas con muchas superficies impermeables usar
  MNDWI. En zonas naturales sin infraestructura, NDWI es suficiente.

---

## 5. Salidas Esperadas

| # | Nombre de la capa          | Tipo   | Cargar al proyecto | Descripción                                           |
|---|----------------------------|--------|--------------------|-------------------------------------------------------|
| 1 | `NDWI_[fecha]`             | Raster | ✅ Sí              | Índice de agua normalizado. Rango: -1 a 1             |
| 2 | `MNDWI_[fecha]`            | Raster | ✅ Sí              | Índice modificado de agua. Rango: -1 a 1              |
| 3 | `Agua_mascara_[fecha]`     | Raster | ⚠️ Opcional        | Máscara binaria: 1=agua, 0=no agua                    |
| 4 | `Cuerpos_agua_[fecha]`     | Vector | ✅ Sí              | Polígonos de cuerpos de agua delimitados              |
| 5 | Bandas escaladas           | Raster | ❌ No (intermedia) | Bandas en reflectancia 0–1, uso interno               |

---

## 6. Lógica de Interpretación para Claude/Antigravity/Gemini

### 6.1 Reglas de interpretación

```
SI valor NDWI/MNDWI > 0.3
  → "Agua abierta con alta certeza. Lagunas, embalses o ríos anchos."

SI valor NDWI/MNDWI entre 0.0 y 0.3
  → "Agua superficial probable o suelo con alta humedad. Puede incluir
     humedales, orillas de ríos o zonas anegadas. Verificar con imagen RGB."

SI valor NDWI/MNDWI entre -0.1 y 0.0
  → "Zona de transición. Posible vegetación acuática emergente, suelo húmedo
     o sombra de nubes. Requiere validación en campo o con imagen RGB."

SI valor NDWI/MNDWI < -0.1
  → "Sin presencia de agua. Suelo seco, vegetación o zona urbana."

SI área total de agua calculada es muy pequeña (< 1 ha) para el área de estudio
  → "El umbral puede estar muy restrictivo o hay presencia significativa de
     nubes. Verificar imagen RGB y considerar bajar el umbral de binarización."

SI el MNDWI detecta significativamente más agua que el NDWI
  → "Es esperado en zonas urbanas o con vegetación densa adyacente al agua.
     El MNDWI suprime mejor la vegetación húmeda. Use MNDWI como referencia."
```

### 6.2 Flags de error

| Síntoma observado                        | Causa probable                               | Acción correctiva                                          |
|------------------------------------------|----------------------------------------------|------------------------------------------------------------|
| Índice todo = NoData                     | Bandas con diferente CRS o extensión         | Reprojectar y verificar solapamiento                       |
| Polígonos de agua en zonas claramente secas | Sombras de nubes no enmascaradas          | Aplicar máscara de nubes/sombras antes de umbralizar       |
| Valores de índice >> 1                   | Bandas no escaladas o fórmula Landsat sin offset | Verificar sensor: Sentinel-2 → `÷10000`; Landsat → `×0.0000275 − 0.2` y recalcular |
| Ríos estrechos no detectados             | Umbral muy alto o resolución insuficiente    | Bajar umbral a -0.05 o usar MNDWI con imagen de mayor res. |
| Error en Polygonize: "no valid pixels"   | Máscara binaria completamente en 0           | Revisar umbral — puede estar muy restrictivo               |
| SWIR y Green con diferente resolución    | Sentinel-2 B11 a 20m vs B03 a 10m           | Remuestrear una de las dos antes del cálculo               |

---

## 7. Optimización y Visualización

### 7.1 Tips de rendimiento

- **Pre-recorte obligatorio:** Las escenas Sentinel-2 completas pesan 1+ GB. Recortar
  al AOI antes de cualquier operación es crítico para el rendimiento.
- **MNDWI sobre SWIR a 20 m:** No remuestrear a 10 m si no es necesario. Trabajar
  a 20 m reduce el tamaño del raster a 1/4 sin perder calidad para la mayoría de análisis.
- **Vectorización con área mínima:** Al poligonizar, filtrar inmediatamente polígonos
  menores al área mínima para evitar miles de polígonos de ruido.

### 7.2 Preprocesamiento recomendado

1. **Enmascarar nubes y sombras:** Usar SCL de Sentinel-2 (clase 3=sombra de nube,
   8/9/10=nubes) antes de calcular. Las sombras de nubes son el principal fuente de
   falsos positivos de agua.
2. **Composición multitemporal:** Para mapear variación de espejo de agua, calcular
   el índice en múltiples fechas y comparar los polígonos resultantes.
3. **Corrección atmosférica:** Indispensable para MNDWI — la banda SWIR es muy sensible
   a efectos atmosféricos en imágenes TOA.

### 7.3 Simbología sugerida

| Capa                     | Tipo de rampa | Val. mín. | Val. máx. | Paleta sugerida                   |
|--------------------------|---------------|-----------|-----------|-----------------------------------|
| `NDWI_[fecha]`           | Continua      | -0.5      | 0.8       | RdYlBu invertido (rojo→azul)      |
| `MNDWI_[fecha]`          | Continua      | -0.5      | 0.8       | RdYlBu invertido                  |
| `Cuerpos_agua_[fecha]`   | Único valor   | —         | —         | Azul (#1a6faf), sin borde o borde blanco fino |

---

## 8. Referencias y Fuentes

- McFeeters, S.K., 1996. *The use of the Normalized Difference Water Index (NDWI)
  in the delineation of open water features.* International Journal of Remote Sensing.
- Xu, H., 2006. *Modification of Normalised Difference Water Index (NDWI) to enhance
  open water features in remotely sensed imagery.* International Journal of Remote Sensing.
- ESA Sentinel-2 L2A Product Description: https://sentinel.esa.int
- USGS Landsat Collection 2: https://www.usgs.gov/landsat-missions

---

*Versión: 1.0 — Spectral Analysis Skills / Isaias Quintana / MCP-GIS*

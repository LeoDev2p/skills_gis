---
name: gis-salud-vegetal
description: >
  Usar este skill para calcular e interpretar índices espectrales de vegetación
  en QGIS vía MCP. Activar cuando el usuario mencione: NDVI, EVI, SAVI, salud
  vegetal, vigor de cultivos, cobertura vegetal, estrés hídrico en plantas,
  monitoreo de vegetación, análisis de biomasa, verdor, fenología, estado de
  cultivos, detección de zonas degradadas, o cuando trabaje con bandas NIR y
  Rojo de Sentinel-2 o Landsat. También activar si el usuario pide clasificar
  o mapear vegetación a partir de imágenes satelitales. No omitir este skill
  si la tarea involucra cualquier índice basado en la reflectancia del
  infrarrojo cercano (NIR) y el rojo visible.
---

# Análisis de Salud Vegetal — NDVI, EVI, SAVI

---

## 1. Misión y Dominio

| Campo         | Valor                                                                                      |
|---------------|--------------------------------------------------------------------------------------------|
| **Objetivo**  | Cuantificar el estado, vigor y densidad de la vegetación mediante índices espectrales derivados de reflectancia superficial satelital. |
| **Dominio**   | Teledetección / Ecología del Paisaje / Agricultura de Precisión                            |
| **Escala**    | Regional (Sentinel-2: 10 m) / Regional-Global (Landsat 8/9: 30 m)                         |
| **Nivel**     | Intermedio                                                                                 |

---

## 2. Insumos Requeridos

### 2.1 Capas necesarias

| # | Capa                  | Tipo   | Contenido                                      | Resolución sugerida | Obligatoria |
|---|-----------------------|--------|------------------------------------------------|---------------------|-------------|
| 1 | Banda Rojo (Red)      | Raster | B04 (Sentinel-2) / B4 (Landsat 8/9)           | 10–30 m             | ✅ Sí       |
| 2 | Banda NIR             | Raster | B08 (Sentinel-2) / B5 (Landsat 8/9)           | 10–30 m             | ✅ Sí       |
| 3 | Banda Azul (Blue)     | Raster | B02 (Sentinel-2) / B2 (Landsat 8/9)           | 10–30 m             | ⚠️ Solo EVI |
| 4 | Máscara de nubes      | Raster | SCL (Sentinel-2) / QA_PIXEL (Landsat)         | Misma que bandas    | ⚠️ Recomendada |
| 5 | Límite área de estudio| Vector | Polígono del área de interés (AOI)            | —                   | ⚠️ Recomendada |

> **Correspondencia de bandas por sensor:**
>
> | Índice | Sentinel-2       | Landsat 8/9     |
> |--------|------------------|-----------------|
> | Rojo   | B04              | B4              |
> | NIR    | B08              | B5              |
> | Azul   | B02              | B2              |

### 2.2 Validación Pre-proceso ⚠️ CRÍTICO

> Claude/Antigravity/Gemini **debe verificar todos estos puntos** antes de ejecutar cualquier
> algoritmo. Un error aquí produce índices incorrectos sin advertencia.

- [ ] **Nivel de procesamiento y factor de escala:** ⚠️ LEER ANTES DE CALCULAR
  Claude/Antigravity/Gemini debe preguntar al usuario qué sensor y nivel de producto tiene, o pedirle
  que revise el archivo de metadatos (`MTL.txt` en Landsat / `MTD_MSIL2A.xml` en Sentinel-2).

  | Sensor | Nivel | ¿Usar? | Fórmula de escalado | Resultado |
  |--------|-------|--------|---------------------|-----------|
  | Sentinel-2 | **L2A** | ✅ Sí | `DN / 10000` | Reflectancia 0–1 |
  | Sentinel-2 | **L1C** | ❌ No usar directo | Requiere corrección atmosférica (Sen2Cor) | TOA, no válido para índices cuantitativos |
  | Landsat 8/9 | **Collection 2 Level-2** | ✅ Sí | `(DN × 0.0000275) − 0.2` | Reflectancia 0–1 |
  | Landsat 8/9 | **Collection 2 Level-1** | ❌ No usar directo | Requiere corrección atmosférica | TOA, no válido para índices cuantitativos |

  > **¿Cómo saber el nivel?** Pedir al usuario que revise:
  > - **Landsat:** archivo `*_MTL.txt` → campo `PROCESSING_LEVEL`. Si dice `L2SP` → Level-2 ✅
  > - **Sentinel-2:** nombre del archivo → si contiene `L2A` → Level-2 ✅ / `L1C` → TOA ❌
  > - **Sentinel-2:** archivo `MTD_MSIL2A.xml` → campo `PROCESSING_BASELINE` confirma el nivel.
- [ ] **CRS:** Debe ser métrico (UTM).
  - Sentinel-2: verificar zona UTM correcta según área (ej: EPSG:32718 para Perú sur).
  - Si está en WGS84 geográfico (EPSG:4326) → reprojectar primero.
- [ ] **Resolución uniforme:** Todas las bandas deben tener la misma resolución.
  - Si mezcla B08 (10 m) con B8A (20 m) de Sentinel-2 → remuestrear a resolución común.
- [ ] **NoData:** Verificar que esté definido (usualmente 0 en Sentinel-2).
- [ ] **Nubes:** Si hay nubes visibles → aplicar máscara antes de calcular índices.
- [ ] **Extensión:** Si hay AOI disponible → recortar bandas antes de procesar.

---

## 3. Flujo de Trabajo

### 3.1 Diccionario de Algoritmos

| Paso | ID Técnico QGIS                        | Herramienta | Descripción                              |
|------|----------------------------------------|-------------|------------------------------------------|
| 0a   | `gdal:cliprasterbymasklayer`           | GDAL        | Recortar bandas al AOI (opcional pero recomendado) |
| 0b   | `gdal:rastercalculator`                | GDAL        | Escalar reflectancias si son enteros (÷10000) |
| 1    | `gdal:rastercalculator`                | GDAL        | Calcular NDVI = (NIR − Red) / (NIR + Red) |
| 2    | `gdal:rastercalculator`                | GDAL        | Calcular EVI = 2.5 × (NIR−Red)/(NIR+6×Red−7.5×Blue+1) |
| 3    | `gdal:rastercalculator`                | GDAL        | Calcular SAVI = ((NIR−Red)/(NIR+Red+L)) × (1+L), L=0.5 |
| 4    | `native:rasterlayerstatistics`         | QGIS        | Estadísticas de cada índice (min, max, media, std) |
| 5    | `native:reclassifybytable`             | QGIS        | Clasificar índice en categorías temáticas |

### 3.2 Fórmulas exactas para `gdal:rastercalculator`

> ⚠️ **Aplicar el escalado correcto según sensor ANTES de calcular cualquier índice.**
> Usar la fórmula incorrecta produce índices fuera del rango [-1, 1] sin advertencia visible.

```
# ── SENTINEL-2 L2A ─────────────────────────────────────────────────
# Factor de escala: DN / 10000
NIR_s2  = "B08@1" / 10000.0
Red_s2  = "B04@1" / 10000.0
Blue_s2 = "B02@1" / 10000.0

# ── LANDSAT 8/9 Collection 2 Level-2 ───────────────────────────────
# Factor de escala: (DN × 0.0000275) − 0.2
# ⚠️ El offset −0.2 es OBLIGATORIO. Sin él los valores son ~20% mayores.
NIR_ls  = ("B5@1"  * 0.0000275) - 0.2
Red_ls  = ("B4@1"  * 0.0000275) - 0.2
Blue_ls = ("B2@1"  * 0.0000275) - 0.2

# ── ÍNDICES (usar las variables escaladas del sensor correspondiente) ─
# NDVI:
("NIR@1" - "Red@1") / ("NIR@1" + "Red@1")

# EVI:
2.5 * (("NIR@1" - "Red@1") / ("NIR@1" + 6.0 * "Red@1" - 7.5 * "Blue@1" + 1.0))

# SAVI (L=0.5):
(("NIR@1" - "Red@1") / ("NIR@1" + "Red@1" + 0.5)) * 1.5
```

### 3.3 Secuencia de Ejecución

```
[Bandas Red, NIR, Blue + AOI]
        │
        ▼
Paso 0a: Recortar al AOI (si disponible)
        │
        ▼
Paso 0b: Escalar reflectancias (si valores > 1)
        │
        ├──────────────────────────────────────┐
        ▼                                      ▼
Paso 1: Calcular NDVI            Paso 0b: ¿Tiene banda Azul?
        │                                      │
        ▼                              Sí ─────▼
Paso 3: Calcular SAVI            Paso 2: Calcular EVI
        │                                      │
        └──────────────┬────────────────────────┘
                       ▼
              Paso 4: Estadísticas
                       │
                       ▼
              Paso 5: Clasificación temática
                       │
                       ▼
              [Capas de índices + mapa clasificado]
```

---

## 4. Parámetros Clave

### 4.1 Tabla de parámetros

| Parámetro      | Valor sugerido | Rango válido | Impacto en el resultado                                    |
|----------------|----------------|--------------|------------------------------------------------------------|
| L (SAVI)       | 0.5            | 0.0 – 1.0    | 0 = zonas con vegetación densa; 1 = zonas con suelo árido  |
| G (EVI)        | 2.5            | Fijo         | Factor de ganancia. No modificar salvo calibración especial |
| C1 (EVI)       | 6.0            | Fijo         | Coeficiente de resistencia aerosol. No modificar            |
| C2 (EVI)       | 7.5            | Fijo         | Coeficiente de corrección azul. No modificar                |
| L_EVI          | 1.0            | Fijo         | Ajuste de fondo de suelo en EVI. No modificar               |
| Tipo de salida | Float32        | —            | Usar siempre Float32 para preservar decimales del índice    |

### 4.2 Variables críticas

> Claude/Antigravity/Gemini **no puede omitir ni asumir** estos parámetros sin confirmar con el usuario.

- **L en SAVI:** Si el área tiene vegetación densa (bosques) usar L=0.25. Si tiene suelo
  muy expuesto (zonas áridas, post-cosecha) usar L=1.0. El valor estándar 0.5 es para
  coberturas mixtas.
- **Factor de escala:** Si las bandas tienen valores enteros (rango 0–10000 típico de
  Sentinel-2 L2A en formato GeoTIFF sin escalar) → OBLIGATORIO dividir entre 10000
  antes de calcular cualquier índice. Omitir esto produce índices con valores >>1, inválidos.

---

## 5. Salidas Esperadas

| # | Nombre de la capa       | Tipo   | Cargar al proyecto | Descripción                                        |
|---|-------------------------|--------|--------------------|----------------------------------------------------|
| 1 | `NDVI_[fecha]`          | Raster | ✅ Sí              | Índice de vegetación normalizado. Rango: -1 a 1    |
| 2 | `EVI_[fecha]`           | Raster | ✅ Sí              | Índice mejorado de vegetación. Rango: -1 a 1       |
| 3 | `SAVI_[fecha]`          | Raster | ✅ Sí              | Índice ajustado por suelo. Rango: -1 a 1           |
| 4 | `Vegetacion_clasificada`| Raster | ✅ Sí              | Mapa temático con categorías de cobertura vegetal  |
| 5 | Bandas escaladas        | Raster | ❌ No (intermedia) | Bandas en reflectancia 0–1, uso interno            |

---

## 6. Lógica de Interpretación para Claude/Antigravity/Gemini

### 6.1 Reglas de interpretación — NDVI

```
SI valor promedio NDVI > 0.8
  → "Vegetación muy densa y saludable. Típico de bosques densos o cultivos
     en plena madurez. Biomasa alta."

SI valor promedio NDVI entre 0.6 y 0.8
  → "Vegetación densa con buen vigor. Pastos bien desarrollados, cultivos
     en crecimiento activo o bosque secundario."

SI valor promedio NDVI entre 0.4 y 0.6
  → "Vegetación moderada. Puede corresponder a arbustos, cultivos en etapa
     inicial o vegetación con leve estrés."

SI valor promedio NDVI entre 0.2 y 0.4
  → "Vegetación escasa o en estrés. Suelo parcialmente expuesto. Posible
     zona degradada, pastizal seco o post-cosecha."

SI valor promedio NDVI entre 0.0 y 0.2
  → "Suelo desnudo, roca o vegetación muy escasa. Verificar con imagen RGB."

SI valor promedio NDVI < 0.0
  → "Cuerpo de agua, nieve, nubes o área urbana densa. NDVI negativo es
     esperado en estas coberturas."
```

### 6.2 Reglas de interpretación — EVI vs NDVI

```
SI EVI difiere significativamente del NDVI en zonas de vegetación densa
  → "El EVI es más sensible en áreas de alta biomasa donde el NDVI tiende
     a saturarse. Use EVI como referencia principal en zonas boscosas."

SI SAVI y NDVI difieren mucho en zonas con vegetación escasa
  → "El SAVI corrige la influencia del suelo. En zonas áridas o semiáridas
     el SAVI es más confiable que el NDVI."
```

### 6.3 Flags de error

| Síntoma observado                     | Causa probable                           | Acción correctiva                                           |
|---------------------------------------|------------------------------------------|-------------------------------------------------------------|
| Índice con valores >> 1 o << -1       | Bandas no escaladas o fórmula Landsat sin offset | Verificar sensor: Sentinel-2 → `÷10000`; Landsat → `×0.0000275 − 0.2` y recalcular |
| Índice todo = NoData                  | Bandas con diferente extensión/CRS       | Verificar solapamiento y reprojectar                        |
| Valores negativos en toda la imagen   | Bandas invertidas (NIR y Red confundidas)| Verificar asignación correcta de bandas por sensor          |
| EVI con valores fuera de [-1, 1]      | Píxeles de nubes o agua no enmascarados  | Aplicar máscara de nubes y recalcular                       |
| División por cero en píxeles aislados | NoData no definido en bandas             | Establecer NoData=0 y agregar condición en la fórmula       |
| Error de rastercalculator "no layer"  | Nombre de banda incorrecto en la fórmula | Verificar nombre exacto de la capa cargada en QGIS          |

---

## 7. Optimización y Visualización

### 7.1 Tips de rendimiento

- **Pre-recorte al AOI:** Recortar bandas antes de calcular. En imágenes Sentinel-2
  completas (100×100 km) sin recorte, la calculadora raster puede tardar varios minutos.
- **Procesar una sola banda NIR y Red:** No cargar el stack completo si solo se necesitan
  2–3 bandas. Cargar solo las necesarias reduce memoria RAM.
- **Guardar como COG:** Para índices de áreas grandes, guardar como Cloud Optimized
  GeoTIFF mejora la visualización en QGIS.

### 7.2 Preprocesamiento recomendado

1. **Enmascarar nubes:** Usar la capa SCL de Sentinel-2 (valores 3,8,9,10,11 = nubes/sombras)
   o QA_PIXEL de Landsat antes de calcular índices. Las nubes producen valores erróneos.
2. **Corrección atmosférica:** Si las imágenes son L1C (TOA), aplicar Sen2Cor antes
   de continuar. Los índices sobre TOA son menos precisos para análisis cuantitativo.
3. **Composición temporal:** Para análisis de grandes áreas, considerar un mosaico
   libre de nubes antes de calcular el índice.

### 7.3 Simbología sugerida

| Capa                    | Tipo de rampa | Val. mín. | Val. máx. | Paleta sugerida                        |
|-------------------------|---------------|-----------|-----------|----------------------------------------|
| `NDVI_[fecha]`          | Continua      | -0.2      | 0.9       | RdYlGn (rojo→amarillo→verde)           |
| `EVI_[fecha]`           | Continua      | -0.2      | 0.8       | RdYlGn                                 |
| `SAVI_[fecha]`          | Continua      | -0.2      | 0.8       | BrBG (marrón→verde)                    |
| `Vegetacion_clasificada`| Clasificada   | —         | —         | Verde oscuro / Verde / Amarillo / Rojo |

**Tabla de clasificación sugerida para mapa temático:**

| Clase | Rango NDVI   | Color sugerido | Etiqueta                  |
|-------|-------------|----------------|---------------------------|
| 1     | < 0.0       | Azul           | Agua / No vegetación      |
| 2     | 0.0 – 0.2   | Gris claro     | Suelo desnudo / Urbano    |
| 3     | 0.2 – 0.4   | Amarillo       | Vegetación escasa         |
| 4     | 0.4 – 0.6   | Verde claro    | Vegetación moderada       |
| 5     | 0.6 – 0.8   | Verde          | Vegetación densa          |
| 6     | > 0.8       | Verde oscuro   | Vegetación muy densa      |

---

## 8. Referencias y Fuentes

- Rouse et al., 1974. *Monitoring vegetation systems in the Great Plains with ERTS.*
  NASA Goddard Space Flight Center. (NDVI original)
- Huete et al., 2002. *Overview of the radiometric and biophysical performance of the
  MODIS vegetation indices.* Remote Sensing of Environment. (EVI)
- Huete, 1988. *A soil-adjusted vegetation index (SAVI).* Remote Sensing of Environment. (SAVI)
- ESA Sentinel-2 User Handbook: https://sentinel.esa.int/web/sentinel/user-guides/sentinel-2-msi
- USGS Landsat Collection 2 Surface Reflectance: https://www.usgs.gov/landsat-missions

---

*Versión: 1.0 — Spectral Analysis Skills / Isaias Quintana / MCP-GIS*

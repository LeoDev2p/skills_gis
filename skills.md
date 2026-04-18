# 📐 Plantilla Maestra — MCP-GIS Skills

> **Instrucciones de uso:** Copia este archivo, renómbralo según el proceso
> (ej: `cuencas-morfometria.md`) y rellena cada sección. Las líneas en
> *cursiva* son instrucciones para el autor — elimínalas en la versión final.

---

## YAML Frontmatter (obligatorio)

```yaml
---
name: gis-[id-proceso]
description: >
  Usar este skill para [nombre del proceso] en QGIS vía MCP. Activar cuando
  el usuario mencione: [lista de palabras clave y sinónimos], análisis de
  [dominio], o solicite flujos de trabajo con [insumos típicos].
  También aplica cuando el usuario pida procesar [tipo de dato] para obtener
  [tipo de resultado]. No omitir este skill si la tarea implica [contexto
  específico del dominio].
---
```

---

# [Nombre Completo del Proceso]

*Ejemplo: Delimitación Morfométrica de Cuencas Hidrográficas*

---

## 1. Misión y Dominio

| Campo            | Valor                                                                 |
|------------------|-----------------------------------------------------------------------|
| **Objetivo**     | *Una frase. Qué problema resuelve este proceso.*                      |
| **Dominio**      | `Hidrología` / `Teledetección` / `Análisis Urbano` / `Geomorfología` / `Otro` |
| **Escala**       | `Local (UAV/LiDAR < 5 km²)` / `Regional (Sentinel/Landsat)` / `Nacional` |
| **Nivel**        | `Básico` / `Intermedio` / `Avanzado`                                  |

---

## 2. Insumos Requeridos

### 2.1 Capas necesarias

| # | Capa            | Tipo    | Contenido                        | Resolución sugerida | Obligatoria |
|---|-----------------|---------|----------------------------------|---------------------|-------------|
| 1 | *Nombre capa 1* | Raster  | *ej: DEM, banda espectral*       | *ej: ≤ 30 m*        | ✅ Sí       |
| 2 | *Nombre capa 2* | Vector  | *ej: límite de área de estudio*  | —                   | ⚠️ Opcional |
| 3 | *Nombre capa 3* | Raster  | *ej: Máscara de nubes*           | *Misma que capa 1*  | ⚠️ Opcional |

### 2.2 Validación Pre-proceso ⚠️ CRÍTICO

> Claude/Antigravity/Gemini **debe verificar todos estos puntos** antes de ejecutar cualquier
> algoritmo. Un error aquí produce resultados silenciosamente incorrectos.

- [ ] **CRS:** Debe ser métrico (UTM o proyección local equivalente).
  - Si la capa está en WGS84 geográfico (EPSG:4326) → reprojectar primero.
  - CRS recomendado para Perú/Sudamérica: `EPSG:32718` (UTM zona 18S) u otro según área.
- [ ] **Resolución:** Verificar que la resolución del raster sea ≤ [valor] m.
  - Si es mayor → remuestrear o advertir al usuario sobre pérdida de detalle.
- [ ] **NoData:** Comprobar que el valor NoData esté definido y sea coherente.
  - Si no está definido → establecerlo antes de procesar.
- [ ] **Extensión/Solapamiento:** Todas las capas deben cubrir la misma área.
  - Si no solapan → recortar al área común o advertir al usuario.
- [ ] **Tipo de dato:** Verificar que el tipo (Int16, Float32, etc.) sea compatible con el algoritmo.

---

## 3. Flujo de Trabajo

### 3.1 Diccionario de Algoritmos

| Paso | ID Técnico QGIS                    | Herramienta  | Descripción breve                   |
|------|------------------------------------|--------------|-------------------------------------|
| 1    | `provider:nombre_algoritmo`        | QGIS/SAGA/GRASS | *Qué hace este paso*             |
| 2    | `provider:nombre_algoritmo`        | QGIS/SAGA/GRASS | *Qué hace este paso*             |
| 3    | `provider:nombre_algoritmo`        | QGIS/SAGA/GRASS | *Qué hace este paso*             |

*Proveedores comunes: `native`, `qgis`, `gdal`, `saga`, `grass7`*

### 3.2 Secuencia de Ejecución

```
[Insumos] 
    │
    ▼
Paso 1: [nombre]
    │
    ▼
Paso 2: [nombre]
    │
    ├─── [condición A] ──▶ Paso 3a: [nombre]
    │
    └─── [condición B] ──▶ Paso 3b: [nombre]
    │
    ▼
[Salidas finales]
```

*Indicar claramente si hay pasos opcionales, paralelos o condicionales.*

---

## 4. Parámetros Clave

### 4.1 Tabla de parámetros

| Parámetro          | Valor sugerido | Rango válido  | Impacto en el resultado                     |
|--------------------|----------------|---------------|---------------------------------------------|
| *Nombre param. 1*  | *valor*        | *min – max*   | *Qué cambia si se aumenta o disminuye*      |
| *Nombre param. 2*  | *valor*        | *min – max*   | *Qué cambia si se aumenta o disminuye*      |

### 4.2 Variables críticas

> Claude/Antigravity/Gemini **no puede omitir ni asumir valores por defecto** para estos parámetros.
> Debe confirmarlos con el usuario o justificar el valor elegido.

- **[Parámetro 1]:** *Por qué es crítico y qué riesgo hay si se configura mal.*
- **[Parámetro 2]:** *Por qué es crítico y qué riesgo hay si se configura mal.*

---

## 5. Salidas Esperadas

| # | Nombre de la capa     | Tipo   | Cargar al proyecto | Descripción                         |
|---|-----------------------|--------|--------------------|-------------------------------------|
| 1 | *nombre_salida_1*     | Raster | ✅ Sí              | *Qué representa esta capa*          |
| 2 | *nombre_salida_2*     | Vector | ✅ Sí              | *Qué representa esta capa*          |
| 3 | *nombre_temporal_3*   | Raster | ❌ No (intermedia) | *Capa de uso interno del proceso*   |

---

## 6. Lógica de Interpretación para Claude/Antigravity/Gemini

> Esta sección define **cómo Claude/Antigravity/Gemini debe explicar los resultados** al usuario,
> actuando como experto en el dominio.

### 6.1 Reglas de interpretación

```
SI [métrica / valor / condición]  →  ENTONCES comunicar al usuario:
  "[Texto de interpretación experta en lenguaje claro]"
```

*Ejemplos a rellenar:*

- Si `NDVI > 0.8` → *"La vegetación es densa y saludable. Posiblemente bosque
  denso o cultivos en pleno desarrollo."*
- Si `NDVI entre 0.4 y 0.8` → *"Vegetación moderada. Puede corresponder a
  pastos, arbustos o cultivos en etapa inicial."*
- Si `NDVI < 0.2` → *"Suelo desnudo, áreas urbanas o cuerpos de agua.
  Verificar con imagen RGB para confirmar."*
- Si área de cuenca `< 1 km²` → *"Microcuenca. Resultados sensibles a la
  resolución del DEM."*

### 6.2 Flags de error

| Síntoma observado                    | Causa probable                        | Acción correctiva para Claude/Antigravity/Gemini                        |
|--------------------------------------|---------------------------------------|------------------------------------------------------|
| Capa de salida vacía (0 features)    | CRS no métrico o sin solapamiento     | Verificar CRS y extensión. Reprojectar si es necesario. |
| Valores todos = NoData               | NoData mal definido en el insumo      | Redefinir NoData y re-ejecutar desde Paso 1.         |
| Área calculada = 0 o negativa        | Capa en coordenadas geográficas       | Confirmar reproyección a CRS métrico.                |
| Error en algoritmo SAGA/GRASS        | Caracteres especiales en ruta de archivo | Mover archivos a ruta sin tildes ni espacios.     |
| Tiempo de proceso excesivo (>10 min) | Raster muy grande sin recorte previo  | Recortar al área de interés antes de procesar.       |
| *[Síntoma específico del proceso]*   | *[Causa específica]*                  | *[Acción específica]*                                |

---

## 7. Optimización y Visualización

### 7.1 Tips de rendimiento

- **Pre-recorte:** Siempre recortar los rasters al área de interés antes de
  procesar. Reduce tiempo de cómputo hasta un 80%.
- **Almacenamiento temporal:** Usar disco local (no red/nube) para capas
  intermedias en procesos pesados.
- **[Tip específico del proceso 1]:** *descripción.*
- **[Tip específico del proceso 2]:** *descripción.*

### 7.2 Preprocesamiento recomendado

*Pasos opcionales pero altamente recomendados antes del flujo principal:*

1. *[ej: Enmascarar nubes con la banda QA antes de calcular índices]*
2. *[ej: Aplicar corrección atmosférica si el análisis es multitemporal]*
3. *[ej: Rellenar depresiones del DEM antes de análisis hidrológico]*

### 7.3 Simbología sugerida

| Capa            | Tipo de rampa     | Valor mín. | Valor máx. | Paleta sugerida        |
|-----------------|-------------------|------------|------------|------------------------|
| *nombre_capa_1* | Continua          | *-1*       | *1*        | RdYlGn (rojo-verde)    |
| *nombre_capa_2* | Clasificada       | *0*        | *5000*     | Turbo / Terrain        |
| *nombre_capa_3* | Único valor       | —          | —          | Color por categoría    |

---

## 8. Referencias y Fuentes

*Incluir solo si el proceso sigue una metodología publicada o estándar oficial.*

- *[Autor, Año. Título. DOI o URL]*
- *[Estándar técnico: ej. USGS, FAO, ANA-Perú]*
- *[Documentación QGIS/SAGA/GRASS relevante]*

---

*Versión de plantilla: 1.0 — César / MCP-GIS Skills*
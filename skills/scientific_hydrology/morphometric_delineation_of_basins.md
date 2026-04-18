---
name: gis-cuencas-morfometria
description: >
  Usar este skill para la delimitación automática de cuencas hidrográficas y 
  obtención de parámetros morfométricos en QGIS vía MCP. Activar cuando
  el usuario mencione: delimitar cuenca, red hídrica, parteaguas, unidad 
  hidrográfica o análisis hidrológico. También aplica cuando el usuario 
  provea un DEM o MDE y solicite el área de contribución de un punto.
---

# Delimitación Morfométrica de Cuencas Hidrográficas

---

## 1. Misión y Dominio


| Campo            | Valor                                                                 |
|------------------|-----------------------------------------------------------------------|
| **Objetivo**     | Obtener el polígono de la cuenca y su red de drenaje a partir de un punto de salida (outlet). |
| **Dominio**      | `Hidrología`                                                         |
| **Escala**       | `Regional (Sentinel/Landsat/SRTM)` / `Local (ALOS PALSAR/LiDAR)`      |
| **Nivel**        | `Avanzado`                                                            |

---

## 2. Insumos Requeridos

### 2.1 Capas necesarias


| # | Capa            | Tipo    | Contenido                        | Resolución sugerida | Obligatoria |
|---|-----------------|---------|----------------------------------|---------------------|-------------|
| 1 | DEM             | Raster  | Modelo Digital de Elevación      | ≤ 30 m (SRTM/ALOS)  | ✅ Sí       |
| 2 | Punto de Salida | Vector  | Punto (Outlet) de la cuenca      | —                   | ✅ Sí       |

### 2.2 Validación Pre-proceso ⚠️ CRÍTICO

- [ ] **CRS:** DEBE ser métrico (ej: UTM Zona 18S). Si el DEM está en grados (WGS84), re-proyectar antes de iniciar.
- [ ] **Unidades:** Confirmar que las unidades horizontales y verticales sean metros.
- [ ] **Depresiones:** Es obligatorio ejecutar el Paso 1 (Fill Sinks) para evitar interrupciones en el flujo digital.

---

## 3. Flujo de Trabajo

### 3.1 Diccionario de Algoritmos


| Paso | ID Técnico QGIS                    | Herramienta  | Descripción breve                   |
|------|------------------------------------|--------------|-------------------------------------|
| 1    | `grass8:r.fill.dir`                | GRASS8       | Elimina imperfecciones y sumideros del DEM. |
| 2    | `grass8:r.watershed`               | GRASS8       | Genera el drainage direction y stream segments |
| 3    | `grass8:r.water.outlet`            | GRASS8       | Delimita el área de la cuenca desde el punto. |
| 4    | `native:polygonize`                | QGIS         | Convierte el raster resultante a polígono vector. |

### 3.2 Secuencia de Ejecución


graph TD
    A[DEM Original] --> B[Paso 1: Fill dir]
    B --> C[DEM Corregido]
    C --> D[Paso 2: drainage direction y stream segments]
    E[Punto de Salida] --> F[Paso 3: Delimitacion de la cuenca]
    D --> F
    F --> G[Paso 4: Poligonizar]
    G --> H[Cuenca Final Vector]



## 4. Parámetros Clave

### 4.1 Tabla de parámetros


| Parámetro          | Valor sugerido | Rango válido  | Impacto en el resultado                     |
|--------------------|----------------|---------------|---------------------------------------------|
| *Threshold*        | 1000           | 100 – 10,000  | Define la densidad de la red hídrica generada. |
| *Method*           | D8             | D8, MFD, Rho8 | Algoritmo de dirección; D8 es el más estable. |

### 4.2 Variables críticas

- **Punto de Salida (Outlet):** El punto debe coincidir con un píxel de alta acumulación. Si la cuenca falla, mover el punto al cauce más cercano.
- **Fill Sinks:** Si se omite, la cuenca se cortará prematuramente en cualquier bache del terreno.

---

## 5. Salidas Esperadas


| # | Nombre de la capa     | Tipo   | Cargar al proyecto | Descripción                         |
|---|-----------------------|--------|--------------------|-------------------------------------|
| 1 | `cuenca_delimitada`   | Vector | ✅ Sí              | Límite final de la unidad hidrográfica. |
| 2 | `red_drenaje`         | Vector | ✅ Sí              | Canales de flujo calculados.        |
| 3 | `dem_sin_huecos`      | Raster | ❌ No (intermedia) | Insumo corregido para el análisis.  |

---

## 6. Lógica de Interpretación para Claude/Antigravity/Gemini

### 6.1 Reglas de interpretación

- Si `Área > 2500 km²` → *"Se ha delimitado una Cuenca. Los tiempos de respuesta ante eventos de lluvia serán prolongados."*
- Si `Área entre 100 y 2500 km²` → *"Se ha delimitado una Subcuenca."*
- Si `Área < 100 km²` → *"Se ha delimitado una Microcuenca. Ideal para estudios de gestión local y control de erosión."*

### 6.2 Flags de error


| Síntoma observado                    | Causa probable                        | Acción correctiva para Claude/Antigravity/Gemini                        |
|--------------------------------------|---------------------------------------|------------------------------------------------------|
| La cuenca es un cuadrado diminuto    | Punto de salida fuera del cauce real  | Desplazar punto al píxel de mayor flujo adyacente.   |
| Área resultante errónea (casi 0)     | Capa en grados (EPSG:4326)            | Reproyectar a UTM (ej. 18S) y repetir proceso.       |
| Error en algoritmo SAGA              | Rutas con espacios o tildes           | Asegurar rutas de archivo simples (C:/temp/...).     |

---

## 7. Optimización y Visualización

### 7.1 Tips de rendimiento

- **Recorte:** Siempre recortar el DEM al área probable de la cuenca antes de procesar para reducir el uso de RAM.
- **D8 vs MFD:** Usar D8 para cuencas simples y MFD si el terreno es muy plano o se busca dispersión de flujo.

### 7.3 Simbología sugerida

- **Cuenca:** Contorno azul oscuro (ancho 0.6mm), relleno azul pálido (transparencia 70%).
- **Drenaje:** Color azul sólido, clasificado por orden de Strahler si está disponible.

*Versión: 1.0 — Spectral Analysis Skills / Isaias Quintana / MCP-GIS*
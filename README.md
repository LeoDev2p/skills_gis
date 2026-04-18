```

███████╗██╗  ██╗██╗██╗     ██╗███████╗   ██████╗ ██╗███████╗
██╔════╝██║ ██╔╝██║██║     ██║██╔════╝   ██╔════╝ ██║██╔════╝
███████╗█████╔╝ ██║██║     ██║███████╗   ██║  ███╗██║███████╗
╚════██║██╔═██╗ ██║██║     ██║╚════██║   ██║   ██║██║╚════██║
███████║██║  ██╗██║███████╗██║███████║   ╚██████╔╝██║███████║
╚══════╝╚═╝  ╚═╝╚═╝╚══════╝╚═╝╚══════╝    ╚═════╝ ╚═╝╚══════╝

```

# Skills GIS

Collection of advanced geospatial analysis workflows for QGIS.

---

## Objective

**Skills GIS** is a set of specialized tools for geospatial analysis optimized for remote sensing, hydrology, geomorphology, and spatial machine learning. Each skill is a complete scientific workflow with data validation and detailed documentation.

---

## Features

- 9 specialized skills across 4 scientific domains
- QGIS integration with GDAL, SAGA, and GRASS
- Multi-scale analysis from local to regional data
- Standard formats: GeoTIFF, GeoJSON, Shapefile
- Complete documentation with methodology and validation
- MCP Ready for AI integration

---

## Available Skills

### Geomorphology & Terrain
- [Surface Characterization](skills/geomorphology_terrain/surface_characterization.md) — Extract terrain attributes (slope, aspect, curvature)

### Scientific Hydrology
- [Morphometric Delineation of Basins](skills/scientific_hydrology/morphometric_delineation_of_basins.md) — Automated watershed delineation with morphometric parameters

### Spatial Machine Learning
- [Object Segmentation](skills/spatial_machine_learning/object_segmentation.md) — Semantic segmentation of satellite imagery
- [Supervised Classification](skills/spatial_machine_learning/supervised%20classification.md) — Classification based on training samples
- [Unsupervised Classification](skills/spatial_machine_learning/unsupervised_classification.md) — Automated cluster detection

### Spectral Analysis
- [Fire Severity](skills/Spectral_analysis/fire_severity.md) — Calculate burn severity indices
- [Vegetation Health](skills/Spectral_analysis/vegetation_health.md) — Vegetation indices (NDVI, MSAVI, EVI)
- [Water Dynamics](skills/Spectral_analysis/water_dynamics.md) — Detection of water bodies and hydrological changes

---

## Documentation

Each skill includes:
- YAML Metadata — Keywords and automatic activation
- Objective & Domain — Scientific context
- Required Inputs — Layers, format, resolution, and validation
- Workflow — QGIS algorithms and execution sequence
- Output Specification — Layer types and interpretation
- Troubleshooting — Common errors and solutions

All documentation is available in: `/skills/`

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- Submit new skills
- Report bugs
- Suggestions
- Improve documentation

---

## Project Statistics

| Metric | Value |
|--------|-------|
| **Total Skills** | 9 |
| **Documented Workflows** | 9/9 (100%) |
| **Scientific Domains** | 4 |
| **Documentation Lines** | 1000+ |
| **Last Updated** | April 2026 |

---

## License

MIT License — See [LICENCE](LICENCE)

Copyright (c) 2026 Isaias Quintana

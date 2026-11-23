# Plugin-QGIS-GCC-climate-scenarios-Brazil
Bilingual (PT/EN) QGIS plugin (3.30+ and 4.x) to generate, Delta-method bias-correct, and evaluate CMIP6 climate scenarios (WorldClim v2.1) for Brazil, with bundled INMET/BDMEP stations, climatological normals, ensembles, and aridity/rainfall indices.

# MCGBrasil Climate Scenarios (v1.7.0)

**MCGBrasil Climate Scenarios** is an open-source QGIS plugin developed at **Embrapa Hortaliças (Embrapa Vegetables, Brasília, Brazil)** to generate, bias-correct, and evaluate **CMIP6 General Circulation Model (GCM) climate scenarios for Brazil** from WorldClim v2.1 (30″, ~1 km²), using bundled INMET weather station databases and climatological normals.

> **Portuguese version:** *Cenários MCGBrasil* (available as a separate installable QGIS package).

---

## Installation

1. Open **QGIS 3.30 to 3.44 or QGIS 4.x** (tested on QGIS 3.34 Qt5 and QGIS 4.2.3 Qt6).
2. Go to **Plugins › Manage and Install Plugins › Install from ZIP**.
3. Select `mcgbrasil_climate_scenarios_v1.7.0.zip` (available on the [Releases](../../releases) page) and click **Install Plugin**.
4. **No external libraries are required:** the plugin relies exclusively on QGIS's native GDAL/OGR, NumPy, and PyQGIS stack.

---

## Workflow Overview

Open the **Phase Workflow Panel** from the toolbar icon (or **Plugins › MCGBrasil Climate Scenarios**). Select a single **project folder** once; each phase writes to its own subfolder and reads outputs from the previous phase.

| Phase | Description | Methodological Reference | Status |
|---|---|---|---|
| **1 – Climate Scenarios** | Partial bounding-box download of WorldClim v2.1 (1970–2000 historical + 14 CMIP6 GCMs, 2021–2100), mask clipping, derived variables, agroclimatic indices, ensembles, statistics, and bilingual maps | Fontenelle et al. (2025), *Revista DCS*, [doi:10.54899/dcs.v22i83.3456](https://doi.org/10.54899/dcs.v22i83.3456); Lima et al. (2025), *Revista Aracê*, [doi:10.56238/arev7n7-250](https://doi.org/10.56238/arev7n7-250) | Ready |
| **2 – Delta-Method Bias Correction** | Automated INMET/BDMEP station selection (bundled 1,163-station database), elevation lapse-rate adjustment, monthly Delta / multiplicative factor, Pearson *r*, and bias-corrected rasters | Lima et al. (2026), *Land Degradation & Development*, [doi:10.1002/ldr.70808](https://doi.org/10.1002/ldr.70808) (Section 2.6) | Ready |
| **3 – Trend Assessment** | Comparison of projected CMIP6 annual rates against bundled INMET climatological normals (1961–1990, 1981–2010, and 1991–2020) | Lima et al. (2026), *Land Degradation & Development*, [doi:10.1002/ldr.70808](https://doi.org/10.1002/ldr.70808) (Section 2.7) | Ready |
| **4 – WorldClim × Historical Eta (Optional)** | Correlation among WorldClim, historical Eta-HadGEM2-ES, and INMET observations | Lima et al. (2026) (Section 2.8) | In development |

---

## Key Features (v1.7.0)

### Phase 1 – Climate Scenarios & Derived Indices
* **Variables:** `Tmin`, `Tmax`, `Tmean` (`(Tmin + Tmax)/2`), `Prec`, and Potential Evapotranspiration (`PET`, Thornthwaite, 1948, with Willmott et al., 1985 for T ≥ 26.5 °C and latitude-based photoperiod correction).
* **Agroclimatic Indices:**
  * **Aridity Index (`IA`):** `P / PET` (monthly and annual) with UNEP-WCMC classes.
  * **Precipitation Concentration Index (`PCI`):** annual index (Oliver, 1980) with standard classes.
  * **Rainfall Severity Index (`ISC`):** `PCI × R / P` (Lima et al., 2025; Galdino, 2015).
* **CMIP6 Coverage:** 4 periods (`2021–2040`, `2041–2060`, `2061–2080`, `2081–2100`), 4 scenarios (`SSP1-2.6`, `SSP2-4.5`, `SSP3-7.0`, `SSP5-8.5`), and 14 WorldClim 30″ GCMs (`ACCESS-CM2`, `BCC-CSM2-MR`, `CMCC-ESM2`, `EC-Earth3-Veg`, `FIO-ESM-2-0`, `GFDL-ESM4`, `GISS-E2-1-G`, `HadGEM3-GC31-LL`, `INM-CM5-0`, `IPSL-CM6A-LR`, `MIROC6`, `MPI-ESM1-2-HR`, `MRI-ESM2-0`, `UKESM1-0-LL`), with real-time warnings for unavailable model–SSP combinations.
* **Multi-Model Ensemble:** equal-weight pixel-wise arithmetic mean across selected models, including monthly inter-model standard deviation and range (`incerteza/` subfolder).
* **Publication-Ready Cartography:** unified color scale across all periods (historical to 2100) and Phases 1–2, generating `.qgz` projects, 300-dpi bilingual (EN/PT) PNG maps, monthly `.txt`/`.csv` statistics (with true spherical pixel area), and SHA-256 JSON manifests.

### Phase 2 – Delta-Method Bias Correction (Bundled INMET/BDMEP)
* Reads the zipped Phase 1 rasters (`.zip`) and writes bias-corrected outputs to a dedicated output folder without modifying Phase 1.
* Uses the **bundled INMET/BDMEP monthly database** (1,163 stations: 565 conventional and 598 automatic, 1970–2020) with automatic handling of incomplete BDMEP headers.
* **v1.7.0 Defaults:** 200 km radius selection, minimum 120 valid months (10 years) per station, Pearson *r* ≥ 0.8 against WorldClim per station, temperature adjustment to the study area's mean elevation via lapse rate (`0.65 °C / 100 m` using WorldClim elevation `wc2.1_30s_elev`), and multiplicative precipitation correction (additive option retained in *Advanced Parameters*).

### Phase 3 – Trend Assessment (Bundled INMET Normals)
* Compares projected CMIP6 annual rates (`2021–2040 → 2081–2100`) against bundled **INMET Climatological Normals** (`1961–1990`, `1981–2010`, `1991–2020`; 430 stations), using paired stations within 200 km (auto-expanding up to 500 km if needed).

---

## Operational Validation & Performance

The v1.7.0 workflow was validated end-to-end in production (live WorldClim v2.1 downloads) across two climatically distinct Brazilian regions — **Distrito Federal** (Central Plateau Cerrado) and **Baixo São Francisco** (Semi-Arid to Coastal transition) — processing all **14 CMIP6 GCMs plus Ensemble** (30 model–region sets) in **~48 hours of unattended execution on two standard home computers**. Compared to the standalone Python scripts workflow in Fontenelle et al. (2025) (~8 days for a single GCM and 3 variables in one region), this represents a **>98% reduction in processing time** while adding 5 derived variables/indices, bias correction, and trend evaluation.

---

## Project Folder Structure

```text
<project>/
├── projeto_mcgbrasil.json
├── 1_cenarios_climaticos/   # <var>/historico/WorldClim/... and <var>/<period>/<model>/<SSP>/...
├── 2_correcao_delta/        # Bias-corrected scenarios and correcao_vies/ reports
└── 3_tendencia_normais/     # Observed vs. projected trend tables and reports

Each release includes SHA256SUMS.txt (per-file hashes inside the plugin) and a .sha256 checksum for the installable archive:FileSHA-256mcgbrasil_climate_scenarios_v1.7.0.zipb8f52b5fd6d4dfd11a3f6bbf5b68041684fe4025d0d873b7d9c3f421fbd23983

Authors
Carlos Eduardo Pacheco Lima — Embrapa Hortaliças, Brasília, DF, Brazil — ORCID: 0000-0002-6975-8560

Mariana Rodrigues Fontenelle — Embrapa Hortaliças, Brasília, DF, Brazil — ORCID: 0000-0001-8318-5933

Marcos Brandão Braga — Embrapa Hortaliças, Brasília, DF, Brazil — ORCID: 0000-0002-0896-8062

Ítalo Moraes Rocha Guedes — Embrapa Hortaliças, Brasília, DF, Brazil

Juscimar da Silva — Embrapa Hortaliças, Brasília, DF, Brazil

Fábio Akiyoshi Suinaga — Embrapa Hortaliças, Brasília, DF, Brazil

Scientific References
Fontenelle, M. R., Amorim, J. R. A., Cruz, M. A. S., & Lima, C. E. P. (2025). Prompt engineering and prompt chaining in artificial intelligence: tools for mapping future climate scenarios as mechanisms of adaptation and climate justice and just transition promotion. Revista DCS, 22(83), 01–17. https://doi.org/10.54899/dcs.v22i83.3456

Lima, C. E. P., Fontenelle, M. R., Braga, M. B., Silva, K. M. N. B., Rodrigues, G. S. S., & Silva, K. C. R. (2025). Clima, morfometria e erosividade em unidade hidrográfica do Cerrado do DF: implicações socioambientais e direitos difusos. Revista Aracê, 7(7), 39480–39503. https://doi.org/10.56238/arev7n7-250

Lima, C. E. P., et al. (2026). The Unfolding Desert. Land Degradation & Development. https://doi.org/10.1002/ldr.70808

Disclaimer
Climate scenarios are model projections and carry inherent uncertainties; users should evaluate multi-model ensembles and validate outputs against local observations before supporting decision-making. The authors and Embrapa are not responsible for how the outputs are used.

License
This software is licensed under the BSD 3-Clause License. See LICENSE for details.



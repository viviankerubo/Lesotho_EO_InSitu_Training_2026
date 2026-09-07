# Winter Wheat Survey Design

| Notebook | Open in Colab | Open in Binder |
|---|---|---|

| `Winter_wheat_survey_design_demo.ipynb` — interactive sampling design walkthrough with live sliders for variance, cost ratio, and PSU/SSU trade-offs | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/viviankerubo/Lesotho_EO_InSitu_Training_2026/blob/main/Winter_wheat_survey_design_demo.ipynb) | [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/viviankerubo/Lesotho_EO_InSitu_Training_2026/main?filepath=Winter_wheat_survey_design_demo.ipynb) |

| Folder | Contents |
|---|---|
| `input_data/Leribe/` | Wheat classification raster, wheat likelihood raster, slope/DEM raster |
| `input_data/Mafeteng/` | Wheat classification raster, wheat likelihood raster, slope/DEM raster |
| `input_data/Berea/` | Wheat classification raster, wheat likelihood raster, slope/DEM raster |
| `input_data/vectors/` | AEZ (agro-ecological zone) shapefile, district boundary shapefile |

## Methodology

Operationalizes the probabilistic, stratified two-stage sampling design from *Proposed Sampling Design
for Winter Wheat Mapping in Lesotho* to validate winter wheat classification and
estimate wheat area for the 2025/26 season in Leribe, Mafeteng, and Berea. Candidate blocks are
stratified by wheat-intensity class — derived from the classified wheat mask and likelihood surface —
crossed with agro-ecological zone (AEZ); blocks on steep terrain are excluded using a slope mask derived
from a DEM. Sample size and PSU/SSU allocation follow Cochran's cost-based two-stage formula, using
variance components (S²between, S²within) estimated from the likelihood surface within each stratum.
## Scope

This repository takes real classified wheat maps and their per-pixel likelihood surfaces — with no
pre-existing in-situ reference data — through to a probabilistic, stratified two-stage sample of
validation points (PSUs and SSUs) for field data collection, plus the achievable precision (CV) of the
resulting design. Reference (ground-truth) labels are collected by enumerators visiting each sampled
field in Lesotho, outside this codebase. One sampling design is run per district — Leribe, Mafeteng, and
Berea — sharing the same frame-build, stratification, allocation, and PSU/SSU-selection pipeline.

## Workflow — winter wheat sampling design (`winter_wheat_survey_tutorial.ipynb`)

1. **Build the area frame.** A 2 km x 2 km grid (`GRID_SIZE_M`, in EPSG:32735) is laid over each
   district's classified wheat raster.
2. **Compute wheat intensity and terrain per block.** Each grid cell's wheat intensity (% wheat pixels)
   is computed via zonal statistics on the classification mask; slope is derived from a DEM.
3. **Assign zones and filter viable blocks.** Each block is tagged with its majority agro-ecological
   zone (AEZ); blocks below a minimum wheat-intensity threshold, without a matched AEZ, or on steep
   terrain (slope > 30%) are dropped.
4. **Stratify.** Viable blocks are grouped into strata by wheat-intensity class (Natural Breaks / Jenks,
   on the classification mask) crossed with AEZ.
5. **Decompose variance per stratum.** Using the likelihood surface — not the binary mask — an
   ANOVA-style decomposition estimates between-block and within-block variance (S²between, S²within),
   the intra-class correlation (ICC), and the stratum's mean predicted probability (p).
6. **Allocate each district's fixed SSU budget across strata** (`allocate_ssus`), weighted by each
   stratum's share of the district's wheat area (Wi) and its variance (optimal allocation).
7. **Enforce feasibility and redistribute** (`enforce_feasibility_and_redistribute`). Any stratum whose
   target exceeds what its own viable-block count and Cochran-optimal cluster size can support is
   capped, and the excess is redistributed to strata with real headroom rather than being discarded.
8. **Jointly optimize PSUs and SSUs per stratum** (`optimize_two_stage_design`). Cochran's cost-based
   formula, using the real travel/survey cost ratio (c1/c2), solves the optimal SSUs-per-PSU (m), then
   the number of PSUs (a) that uses as much of the stratum's budget as possible without exceeding it.
9. **Select PSUs and generate SSU points** (`select_psus_and_ssus`). Exactly `a` blocks are randomly
   sampled per stratum, and `a x m` field points are placed inside them, distributed as evenly as
   possible across the selected blocks.
10. **Compute achievable precision** (`compute_achievable_cv` / `compute_district_cv`), using the
    realized — not theoretical — PSU/SSU counts, with a finite-population correction.
11. **Independent boundary-clipped check.** Total wheat area and percentage are recomputed by clipping
    the classification directly to each district's official administrative boundary, as a sanity check
    against the grid-based totals.
12. **Export PSUs and SSUs.** Per-district and combined national GeoJSON/shapefile layers are written
    for field deployment and mapping.

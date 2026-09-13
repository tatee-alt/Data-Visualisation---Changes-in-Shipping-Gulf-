# Data provenance

## Source

Sentinel-1 SAR vessel detections and scene footprints, downloaded from the
Global Fishing Watch Data Download Portal
(<https://globalfishingwatch.org/data-download/>), covering 1 September 2025 to
29 June 2026. Access requires a free GFW account. The underlying imagery is from
the European Space Agency's Sentinel-1 mission under the Copernicus open data
licence; GFW publishes the detections under CC BY-NC 4.0.

The original download is 4.21 GB across twenty CSV files. The three files in
this folder total 62 MB and contain only what the analysis reads.

## Files

| File | Rows | Size | Contents |
|---|---|---|---|
| `hormuz_aoi_detections.csv.gz` | 239,712 | 7.1 MB | Detections in the study area (lon 30–78°E, lat 0–32°N), all ten source columns |
| `world_detections_slim.csv.gz` | 2,394,698 | 53.8 MB | Global detections, seven columns needed for the control-region, corridor and size-filtered comparisons |
| `scene_footprints_aoi.csv.gz` | 2,533 | 1.0 MB | Water-masked footprint polygon of every Sentinel-1 scene intersecting the study area |

## Derivation

**1. Monthly de-duplication.** The download contains both
`sar_vessel_detections_pipev4_202606.csv` (covering 1–27 June 2026) and
`sar_vessel_detections_pipev4_20260629.csv` (covering 1–29 June 2026). These are
the same monthly export cut at two different dates, and reading the folder with
a wildcard counts June twice. One file per calendar month is retained, the
later cut, since it carries strictly more days. A further de-duplication on
`(scene_id, timestamp, lat, lon)` was applied as a safety net and removed no
additional rows.

**2. Spatial subsetting.** The AOI file retains detections inside the study
bounding box. The world file retains all detections but only the columns the
global comparisons require: `timestamp, lat, lon, length_m, mmsi, matching_score,
matched_category`. `length_m` is included because the size filter has to apply on
both sides of every comparison, and it is the only attribute present for dark
detections: `matched_category` is `unmatched` for all of them, so vessel class
cannot filter that population.

**3. Footprint reduction.** Scene footprints are the bulk of the raw download
(2.9 GB) because each row carries a detailed water-masked polygon averaging
21.6 kB of well-known text. Two reductions were applied:

- only scenes intersecting the study area are kept (2,533 of 54,000);
- geometries are simplified at a 0.02° tolerance and coordinates rounded to
  three decimal places, reducing the mean polygon to 1.8 kB.

The simplification tolerance is one fifth of the 0.1° grid the polygons are
rasterised onto, so it cannot move a grid-cell centre across a boundary in any
way that materially affects the coverage calculation. This was verified rather
than assumed: rasterising eight sample days from both the full and simplified
geometries gives observed cell counts differing by a mean of **0.33%**.

| Date | Cells (full) | Cells (simplified) | Difference |
|---|---|---|---|
| 2025-12-01 | 2,706 | 2,713 | +0.26% |
| 2025-12-21 | 2,077 | 2,081 | +0.19% |
| 2026-01-13 | 1,939 | 1,941 | +0.10% |
| 2026-02-02 | 2,140 | 2,142 | +0.09% |
| 2026-02-22 | 1,834 | 1,851 | +0.93% |
| 2026-03-17 | 3,470 | 3,471 | +0.03% |
| 2026-04-06 | 1,471 | 1,476 | +0.34% |
| 2026-04-26 | 1,117 | 1,125 | +0.72% |

## Known characteristics of the source data

- **Truncated months.** February 2026 ends on the 25th, May on the 28th and June
  on the 29th. Detection counts per calendar month dip in those months for
  clerical rather than maritime reasons. Every comparison in the analysis is
  therefore expressed per observed day.
- **Pipeline version change.** Files up to February 2026 are labelled `pipev3`
  and files from March 2026 `pipev4`. The provider changed its detection and
  matching pipeline mid-series. Section 4 of the notebook tests explicitly
  whether this confounds the result.
- **Footprint coverage begins December 2025.** No scene footprints are published
  for September to November 2025, so coverage-corrected statistics start in
  December and the earlier months are compared on raw counts only.
- **`matched_category == 'unmatched'` is ambiguous** and mixes detections with no
  AIS candidate against those whose candidate was rejected below a 0.01 score
  threshold. The notebook separates these.

## Reproducing the packaging

The script that produced these files is `package_dsm050.py` in the analysis
repository. It reads the raw GFW CSVs, applies the steps above and writes this
folder.

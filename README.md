DSM050 Data Visualisation, final coursework. A self-contained study of shipping
behaviour around the Strait of Hormuz between September 2025 and June 2026, using
Sentinel-1 SAR vessel detections from Global Fishing Watch.

## Contents

| Path | What it is |
|---|---|
| `Hormuz_SAR_Study.ipynb` | The full analysis, executed, with every figure embedded |
| `report/DSM050_Report.pdf` | The written report (also `.docx` and the `.md` source) |
| `data/` | The three prepared data files the notebook reads (62 MB) |
| `data/DATA_PROVENANCE.md` | How the prepared files were derived from the 4.2 GB download |
| `figures/` | Every figure as PNG and interactive HTML |

## Running it

Python 3.12 was used. Install the packages, then open the notebook in this folder and run
all cells, or execute it from the command line:

    pip install -r requirements.txt
    jupyter nbconvert --to notebook --execute --inplace Hormuz_SAR_Study.ipynb

A full run takes about five minutes and peaks at roughly 1 GB of memory. The notebook has no
dependency on any other file in this repository beyond `data/` and
`figures/fig01_containership_classes_rodrigue.png`.

### Packages

| Package | Version used | Purpose |
|---|---|---|
| pandas | 3.0.3 | tabular processing of detections |
| numpy | 2.4.4 | array arithmetic, rasterisation, change-point scan |
| plotly | 7.0.0 | every figure (`plotly.graph_objects`, `plotly.subplots`, `plotly.io`) |
| shapely | 2.1.2 | scene footprint polygons, clipping and rasterisation |
| kaleido | 1.3.0 | static PNG export of figures; optional, PNGs are skipped if absent |
| jupyter, ipykernel, nbconvert | 7.1.0, 7.16.6 | running the notebook |

Standard-library modules used: `pathlib`, `re`, `itertools`, `math`, `sys`, `time`,
`warnings`, `json`. Figures are rendered with Plotly's `notebook_connected` renderer, which
loads `plotly.js` from a CDN, so viewing the charts interactively needs an internet connection;
the analysis itself does not.

The report PDF and DOCX were produced from `report/DSM050_Report.md` with `reportlab` and
`python-docx`; those two packages are not needed to run the notebook.

## Story order

1. The world fleet by hull-length class (Section 4)
2. Is the Gulf change real? Controls, pipeline break, coverage (Section 5)
3. The strait, all vessels (Section 6)
4. The strait re-cut by size: grid, threshold ladder, 200 m+ routes, the
   "ships stayed" bar chart (Section 7)
5. What the identifiable fleet did: fleet fate, ports, corridors (Section 8)

`figures/fig01_containership_classes_rodrigue.png` is reproduced from Rodrigue,
*The Geography of Transport Systems*, under its educational-use terms.

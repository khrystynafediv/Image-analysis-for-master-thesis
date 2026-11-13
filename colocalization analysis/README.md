# Colocalisation analysis in cellular blebs

A Jupyter notebook that quantifies the colocalisation of two fluorescence channels
(here ER and ConA) inside manually drawn ROIs (cellular blebs) and generates the
figures for the associated manuscript.

For every ROI the notebook calculates

- **Pearson's r** and **Spearman's ρ** (pixel-wise intensity correlation),
- **Manders' M1 and M2** with automatic **Costes thresholds** (as in Fiji's *Coloc 2*),

and produces fluorograms (2D scatter plots), pseudo-colour colocalisation maps,
mean ± SEM bar graphs and box-and-whisker plots (minimum to maximum, all points shown).

## Repository contents

```
.
├── colocalisation_analysis.ipynb   # the analysis
├── environment.yml                 # conda environment
├── requirements.txt                # pip alternative
├── data/                           # put your images and ROI sets here (see below)
└── README.md
```

## Installation

```bash
conda env create -f environment.yml
conda activate bleb-coloc
jupyter lab
```

or, with pip (Python ≥ 3.10):

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

## Input data

Place each multichannel TIFF in `data/` together with a Fiji ROI archive of the **same
name**:

```
data/
├── cell1.tif   cell1.zip
├── cell2.tif   cell2.zip
└── cell3.tif   cell3.zip
```

- ROIs are created in Fiji with the ROI Manager and saved with *More > Save…* as a `.zip`.
- Supported images: ImageJ hyperstacks, OME-TIFF (e.g. Micro-Manager) and plain
  multipage TIFFs. Time points other than the first are ignored.
- Images without a matching `.zip` are skipped (a message is printed).
- Channels are selected with `CH_A` and `CH_B` in the configuration cell
  (0-based; the defaults `1, 2` correspond to channels 2 and 3).

## Usage

1. Open `colocalisation_analysis.ipynb`.
2. Adjust the settings in the **Setup** cell (channel indices, labels, display
   settings, scale bar length). If the pixel size cannot be read from the metadata,
   set `PIXEL_SIZE_UM`.
3. Run all cells. Results are written to `results/`.

## Output

| File | Content |
|---|---|
| `colocalisation_per_bleb.csv` | Coefficients and Costes thresholds for every ROI |
| `colocalisation_summary.csv` | Mean, SEM and *n* per cell |
| `fluorograms_per_cell.png` | 2D intensity histogram per cell |
| `pseudocolour_maps.png` | Channels, merge and colocalisation map for every cell |
| `fluorogram_all_cells.png` | Pooled scatter plot, one regression line per cell |
| `bar_graph.png` | Mean ± SEM per cell and for all blebs |
| `boxplot_all_blebs.png` | Box-and-whisker plot of all blebs |
| `figure_colocalisation.png` / `.pdf` | Composite figure (panels A–C) |

## Methods in brief

- Coefficients are calculated from the **raw, unscaled** pixel intensities inside each
  ROI, without background subtraction.
- ROIs assigned to a z-plane in Fiji are analysed on that plane; other ROIs and
  single-plane images use the maximum-intensity projection.
- ROIs with fewer than `MIN_ROI_PIXELS` pixels, or without intensity variance in either
  channel, are excluded.
- Costes thresholds are determined separately for every ROI.
- Intensity limits, gamma and colour maps are applied **to the figures only**.
  The colour scale of the colocalisation map is relative (top = 99th percentile of the
  channel product inside the ROIs), so colours cannot be compared between cells as
  absolute values.

## Caveats

- Blebs of the same cell are not statistically independent. The number of independent
  replicates is the number of cells, so pooled bleb data are descriptive.
- Manders' coefficients depend on the thresholds and approach 1 when thresholds are low;
  interpret them together with Pearson's r.
- The y axis of the box plot is truncated (`BOX_YLIM`); mention this in the figure legend.



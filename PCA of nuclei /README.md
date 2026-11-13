# Nuclear shape analysis across area-defined size classes

Contour-based morphometry of nuclei segmented in [ilastik](https://www.ilastik.org/) and analysed with [CellTool](https://github.com/zpincus/celltool) and Python (NumPy, SciPy, pandas, Matplotlib).
The repository contains the analysis notebook of a master's thesis. It starts from **binary masks** and ends with publication-style figures and tables.

## What the notebook does

1. **Contour extraction.** CellTool extracts closed 2D outlines as iso-intensity contours of each mask, converts them to micrometres, smooths them with a periodic cubic B-spline and resamples every outline to a fixed number of points. Masks may contain several nuclei of different sizes.
2. **Size classes.** Each nucleus is assigned to a class by its projected area (default: *Micronuclei*, *Unconfined nuclei*, *Mildly confined nuclei*, *Confined nuclei*). A histogram of areas and a helper that suggests thresholds help to place the class boundaries.
3. **Alignment.** Outlines are aligned rigidly (rotation, translation, optional reflection) per class and all together with `celltool align_contours`.
4. **PCA of shape**, computed in two variants:
   * `size_included`: PCA on the aligned outlines (size and shape together);
   * `shape_only`: every outline rescaled to the mean area before PCA (shape independent of size).
5. **Statistics.** Kruskal–Wallis test with the effect size ε² for each mode, and pairwise Kolmogorov–Smirnov tests (Bonferroni-corrected).
6. **Figures and tables.** A composite figure (panels A-C), supplementary views per class and CSV tables.

## Repository structure

```
.
├── nuclear_shape_analysis.ipynb   # the analysis (run top to bottom)
├── environment.yml                # conda environment
├── README.md
├── .gitignore
├── data/
│   └── masks/                     # put your binary masks here (not tracked by git)
└── output/                        # created by the notebook (not tracked by git)
    ├── contours/                  # extracted outlines (.contour)
    ├── aligned/                   # aligned outlines: one folder per class and All/
    ├── models/                    # CellTool shape models
    ├── figures/                   # PNG / SVG / PDF
    ├── tables/                    # CSV tables
    └── software_versions.txt
```

## Installation

```bash
git clone <your-repository-url>
cd <repository-folder>
conda env create -f environment.yml
conda activate nuclear-shape
jupyter lab
```

CellTool and its dependency `freeimage-py` are installed from GitHub (pinned commits in `environment.yml`), so `git` and an internet connection are required during installation. `setuptools<81` is pinned because CellTool imports `pkg_resources`.

## Data

Input: binary masks (nuclei foreground, background 0) in `.tif`, `.tiff` or `.png` format, exported from ilastik, in `data/masks/`. One image may contain any number of nuclei of any size. Notes:

* Objects touching the image border are discarded (only closed contours are kept) and so are objects smaller than `MIN_AREA_PX`.
* Some ilastik exports add the prefix `Mask of ` to file names. It is removed from contour names (`STRIP_PREFIX`).
* The pixel size (`PIXEL_SIZE_UM`) must be set to the value of your microscope and objective.
* The thesis data are not part of the repository. To try the pipeline without them, set `USE_DEMO_DATA = True`: the notebook then generates synthetic masks in `data/demo_masks/`. The synthetic nuclei have no biological meaning, so the demo output should not be interpreted.

## Usage

1. Open `nuclear_shape_analysis.ipynb` and edit the **configuration cell** (section 1): paths, pixel size, alignment options, size classes (`SIZE_GROUPS_PX`), class names and colours, figure settings.
2. Run all cells. A full run on the demo data (320 outlines) took about one minute in testing.
3. Collect the results from `output/`.

The most important parameters:

| Parameter | Meaning |
|---|---|
| `PIXEL_SIZE_UM` | micrometres per pixel |
| `MIN_AREA_PX`, `N_POINTS`, `SMOOTHING` | minimum object size, points per outline, B-spline smoothing of CellTool |
| `SIZE_GROUPS_PX` | integer area ranges (pixels, both ends inclusive) that define the size classes |
| `ALLOW_REFLECTION`, `MAX_ITER` | alignment options |
| `PCA_PANEL_A`, `PCA_PANEL_C` | which PCA variant (`shape_only` or `size_included`) is shown in panels A and C of the main figure |
| `ELLIPSE_LEVEL` | fraction of the data enclosed by the ellipses in panel C |
| `AREA_LOG_SCALE` | log scale for the area axis |

## Outputs

| File | Content |
|---|---|
| `output/figures/Figure_composite.{png,pdf,svg}` | main figure: shape modes (A), outlines and projected area (B), shape space (C) |
| `output/figures/S1_size_class_thresholds.png` | area distribution with class boundaries |
| `output/figures/S2a/S2b_outlines_by_class_*` | outlines per class, coloured by point order (extracted / aligned) |
| `output/figures/S3_modes_<class>.*` | shape modes of each class |
| `output/tables/all_nuclei_measurements.csv` | one row per nucleus: class, area, normalized mode scores of both PCA variants |
| `output/tables/summary_by_class.csv` | n and area statistics per class |
| `output/tables/pca_variance_and_area_correlation.csv` | variance explained per mode and Spearman correlation of each mode with area |
| `output/tables/kruskal_wallis_modes.csv`, `pairwise_ks_modes.csv` | test results (H, p, ε²; KS D, p, Bonferroni p) |
| `output/tables/celltool_measurements_<class>.csv` | CellTool `measure_contours` output (area, centroid, mode positions) |
| `output/software_versions.txt` | Python, package versions and the CellTool commit used |

## Interpretation notes

* **CellTool does not normalise size during alignment.** In the `size_included` PCA the leading mode usually reflects nuclear size. Check the Spearman correlation with area in `pca_variance_and_area_correlation.csv`, and use `shape_only` to ask about shape.
* **Classes are defined by area thresholds.** Area differences between the classes follow from the definition and are not tested. The classes are a proxy for nuclear flattening, not an independent measurement of confinement.
* **p-values with large n.** Almost any comparison reaches p < 0.001 with hundreds of nuclei. Interpret the effect sizes (ε², KS *D*) and the distributions.
* **Small objects.** Outlines of very small objects (for example micronuclei) are sensitive to pixel size, segmentation and smoothing; apparent shape differences can be partly technical.
* **Independence.** Nuclei from the same image are not independent observations; confirm key findings at the level of independent experiments or images.
* **Alignment warnings.** If CellTool reports `did not converge`, increase `MAX_ITER`, try `ALLOW_REFLECTION = False` and inspect the outlines for merged nuclei or debris (supplementary figure S2).

## Troubleshooting

| Problem | Remedy |
|---|---|
| `CellTool executable not found` | activate the conda environment before starting Jupyter |
| `No masks found in data/masks` | add masks, or set `USE_DEMO_DATA = True` |
| a class is empty or very small | adjust `SIZE_GROUPS_PX` using figure S1 and the threshold helper |
| `ModuleNotFoundError: pkg_resources` | install `setuptools<81` |

## Tested with

The notebook was run end-to-end in demo mode with Python 3.12.3 (Linux), NumPy 2.5.4, SciPy 1.18.1, pandas 3.0.6, Matplotlib 3.11.2, scikit-image 0.26.0, tifffile 2026.9.20, CellTool 2.1 (commit `ac6a0bb`) and freeimage 1.2.1 (commit `b7c2adf`)

## References

* Pincus, Z. & Theriot, J. A. Comparison of quantitative methods for cell-shape analysis. *J. Microsc.* 227, 140–156 (2007). https://doi.org/10.1111/j.1365-2818.2007.01799.x (CellTool)
* Berg, S. et al. ilastik: interactive machine learning for (bio)image analysis. *Nat. Methods* 16, 1226–1232 (2019). https://doi.org/10.1038/s41592-019-0582-9
* Jolliffe, I. T. & Cadima, J. Principal component analysis: a review and recent developments. *Phil. Trans. R. Soc. A* 374, 20150202 (2016). https://doi.org/10.1098/rsta.2015.0202

## Citation and license

If you use this code, please cite the thesis: *[author, title, institution, year]*

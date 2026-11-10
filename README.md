# LC3 Lipophagy Analysis

Analysis code for the LC3 lipophagy / TAX1BP1 experiment in the Kara Neurodegenerative Disease Lab. The notebook segments cells in four-channel `.czi` images, finds green-positive cells, detects LC3 dots and rings, and measures how they colocalize with lipid droplets and mitochondria.

## Notebooks

- [lc3_lipophagy_tax1bp1_analysisfinal.ipynb](lc3_lipophagy_tax1bp1_analysisfinal.ipynb): the main pipeline. Segments every image, saves masks, analyzes green-positive cells, and writes all CSVs.
- [additional_lc3_analysis_columns.ipynb](additional_lc3_analysis_columns.ipynb): adds the ring-around-droplet columns to a run that was made before those columns existed. It reads the saved masks and the main CSV, so the main notebook must be run first. New runs of the main notebook already include these columns.

## Channels

`CHANNEL_ORDER` in the parameters cell:

| Index | Channel | Meaning |
|---|---|---|
| 0 | red | lipid droplets |
| 1 | yellow | mutant LC3 |
| 2 | green | normal LC3 |
| 3 | blue | mitochondria |

Cells are segmented with Cellpose on the yellow + green composite.

## Setup

1. Clone the repo and create an environment (Python 3.12 was used):

   ```bash
   git clone https://github.com/timmyhsu07/LC3_Lipophagy_Analysis.git
   cd LC3_Lipophagy_Analysis
   python -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   pip install "cellpose<4" czifile numpy pandas scipy scikit-image matplotlib jupyter
   ```

   Cellpose must be below version 4. The notebook uses `models.Cellpose(model_type="cyto3")`, which was removed in Cellpose 4.

2. Put your images in a folder next to the notebook, or under `images/`:

   ```
   LC3_Lipophagy_Analysis/
     lc3_lipophagy_tax1bp1_analysisfinal.ipynb
     images/
       L3 Lipophagy images/
         ..._CH_....czi
         ..._noCH_....czi
   ```

3. Name files so the condition can be read. Any filename containing `_noCH_` is labeled `noCH`; every other file is labeled `CH`.

4. In the paths cell, set `IMAGE_FOLDER_NAMES` to your folder name(s). It currently lists `"L3 Lipophagy images"`.

5. If you have no GPU, set `USE_GPU = False`. Set `SHOW_PLOTS = False` for long batch runs.

6. Run the notebook top to bottom.

## Outputs

All written to `Results/`:

- `<folder>_LC3_lipophagy_mitophagy_analysis.csv`: one row per green-positive cell.
- `<folder>_Additional_Analysis_per_cell.csv`: the same cells with a `Condition` column, sorted CH first.
- `<folder>_Additional_Analysis_summary_by_condition.csv`: n, mean, and SD per metric for CH vs noCH.
- `Results/masks/<folder>/<image>/`: saved masks (`labeled_cells.npy`, `green_objects.npy`, `red_lipid.npy`, `mito.npy`, and others).

## Notes

- MFI is measured on raw channels only.
- Hollow LC3 rings are filled before overlap tests, so a droplet or mitochondrion inside a ring still counts as a match.
- Segmentation thresholds are in the parameters cell with comments. Check the QC panels before trusting a new dataset.

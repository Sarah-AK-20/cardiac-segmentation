# ACDC Cardiac MRI: Data Preprocessing

Preprocessing pipeline for the **ACDC** dataset, preparing cardiac MRI volumes for 2D segmentation with deep learning. It follows the [data exploration notebook](01_data_exploration_acdc_colab.ipynb) and produces the inputs used by the U-Net training notebooks. No augmentation or training is done here.

## What the notebook does

1. **In-plane resampling** to a common spacing of 1.5 mm. Slice thickness is kept unchanged, so the work stays 2D, slice by slice.
2. **Intensity normalisation** per volume: clipping at the 0.5th and 99.5th percentiles, then z-score.
3. **Fixed-size canvas** of 256 × 256 pixels, by centre cropping or zero padding. The crop does not use the mask, so the same pipeline works on images without annotation.
4. **ED and ES frames only**, the two annotated phases of each patient.
5. **Saving** each volume as a compressed `.npz` file, with a `.json` sidecar that stores the offsets needed to map predictions back to the original grid.
6. **Patient-level train/validation split**, stratified by clinical group, so that no slice of a patient appears in both sets.
7. **Quality checks**: heart voxels lost during cropping, empty slices, and a visual overlay check.

## Results

| Item | Value |
|---|---|
| Volumes processed | 300 (150 patients × ED and ES) |
| Train / validation / test | 160 / 40 / 100 volumes |
| Validation split | 4 patients per clinical group |
| Heart voxels lost by cropping | 0% on the training set |
| Slices without heart | 136 of 2,978 (4.6%) |

## Output layout

```
MyDrive/segmentation_cardiaque/preprocessed/
├── train/   patientXXX_ED.npz, patientXXX_ED.json, ...
├── test/    patientXXX_ES.npz, patientXXX_ES.json, ...
└── metadata.csv
```

Each `.npz` contains `image` (float16, 256 × 256 × Z) and `mask` (uint8, same shape). The `metadata.csv` file has one row per volume, with the `subset` column (`train`, `val` or `test`), the clinical group, the original shape and spacing, and the crop offsets.

## Requirements

- Google Colab (CPU is enough for this notebook)
- `nibabel`, `scikit-learn`, `tqdm`, `pandas`, `matplotlib`, `scipy` (installed in the first cell)
- The ACDC dataset on Google Drive, see the [data exploration README](README.md) for the expected layout

## How to run

1. Mount Google Drive and check that `DATA_ROOT` points to your dataset folder.
2. Run the cells in order. The processing step writes files to Drive, and it can resume if the Colab session stops: existing files are skipped unless `OVERWRITE = True`.

## Notes

- The crop is centred on the image, not on the heart. Very large hearts or unusual fields of view can lose part of the myocardium at the edges. The quality check reports this, and it was zero for the training set.
- Predictions can be mapped back to the original grid using `shift_x`, `shift_y` and `resampled_shape` from the `.json` sidecar, then resampled to the original spacing.

# ACDC Cardiac MRI: U-Net Segmentation (v2)

2D U-Net for segmenting the right ventricle (RV), myocardium (MYO) and left ventricle (LV) in short-axis cardiac MRI from the **ACDC** dataset. This version extends the baseline with a longer training schedule and a 3D, per-patient evaluation. It uses the preprocessed data from the [preprocessing notebook](02_preprocessing_acdc_colab.ipynb).

## What changed from the baseline

- **Training**: up to 80 epochs, cosine learning-rate decay from 1e-3 down to 1e-5, and early stopping after 15 epochs without improvement on validation Dice.
- **Loss**: cross-entropy plus soft Dice loss, to counter the strong background imbalance.
- **Evaluation**: besides the 2D slice-level scores, Dice and IoU are computed in 3D for each patient, on the whole volume, at both ED and ES.

## Setup

- 2D U-Net with four encoder/decoder levels, 32 base filters, and skip connections. About 7.8 M parameters.
- Data: 160 training, 40 validation and 100 test volumes, split by patient.
- Training stopped at epoch 52 out of 80; the best validation checkpoint was reached at epoch 37.

## Results

**2D, slices containing the heart**

| Split | RV (VD) | Myocardium | LV (VG) | Mean Dice | Mean IoU |
|---|---|---|---|---|---|
| Validation | 0.910 | 0.894 | 0.956 | 0.920 | 0.853 |
| Test | 0.917 | 0.893 | 0.957 | 0.922 | 0.857 |

**3D, whole volume, per patient (ED and ES)**

| Split | RV (VD) | Myocardium | LV (VG) | Mean Dice |
|---|---|---|---|---|
| Validation | 0.880 | 0.880 | 0.929 | 0.896 |
| Test | 0.896 | 0.881 | 0.911 | 0.896 |

Mean 3D Dice by clinical group, test set: DCM 0.915, RV 0.904, MINF 0.897, NOR 0.888, HCM 0.875.

## Interpretation

- The model is stable: validation and test scores are nearly identical in both 2D and 3D, so it generalises to unseen patients.
- The longer schedule did not improve the 2D score over the baseline (0.920 versus 0.920 on validation). The learning curves show a plateau around 0.92, with training Dice reaching about 0.96, so the model is starting to overfit.
- The myocardium is the hardest structure in every evaluation (about 0.88 to 0.89 Dice). Its thin, ring-shaped geometry makes boundary errors more costly.
- The HCM group has the lowest 3D Dice on the test set, which is consistent with the more heterogeneous shapes in that group.
- The 3D scores are lower than the 2D scores because the 3D evaluation also includes basal and apical slices that contain little or no heart.

## Requirements

- Google Colab with a GPU (T4 or better recommended)
- `torch`, `numpy`, `pandas`, `matplotlib`, `scikit-learn`, `tqdm`
- The preprocessed `.npz` files and `metadata.csv` produced by the preprocessing notebook, stored in `MyDrive/segmentation_cardiaque/preprocessed/`

## How to run

1. Mount Google Drive and check that `DATA_ROOT` points to your project folder.
2. Run the cells in order. Training and evaluation outputs are saved to `unet_runs/`.
3. The notebook uses a fixed random seed (42), but results can still vary slightly between GPUs.

## Limitations

- Each figure comes from a single training run, so differences of a few thousandths should not be over-interpreted.
- Evaluation uses a local split of the 150 ACDC patients, not the official challenge leaderboard, so scores are not directly comparable with challenge results.
- No data augmentation is used in this version.

# ACDC Cardiac MRI: U-Net Segmentation (v3)

2D U-Net for segmenting the right ventricle (RV), myocardium (MYO) and left ventricle (LV) in short-axis cardiac MRI from the **ACDC** dataset. This version adds data augmentation and exposes the main hyperparameters in a single configuration block. It uses the preprocessed data from the [preprocessing notebook](02_preprocessing_acdc_colab.ipynb).

## What this version does

- **Data augmentation** on the training batches only: random rotation (±10°), zoom (±10%), translation (±5% of the image size), contrast variation (±10%) and Gaussian noise. There is no horizontal flip, because it would swap the left and right ventricles. The mask receives exactly the same geometric transformation, with nearest-neighbour interpolation.
- **Configurable hyperparameters** in one block at the top of the notebook: kernel size, number of filters, dropout, learning rate, weight decay, batch size, number of epochs, early-stopping patience, loss type and augmentation strength.
- **Training schedule**: up to 80 epochs, cosine learning-rate decay from 1e-3 to 1e-5, and early stopping after 15 epochs without improvement on validation Dice.
- **Loss**: cross-entropy plus soft Dice loss.
- **Evaluation**: 2D slice-level Dice and IoU, and 3D Dice per patient on whole volumes at ED and ES.

## Configuration of the reported results

| Parameter | Value |
|---|---|
| Kernel size | 3 |
| Base filters | 32 (about 7.8 M parameters) |
| Dropout | 0 |
| Learning rate | 1e-3, cosine decay to 1e-5 |
| Weight decay | 0 |
| Batch size | 8 |
| Augmentation | rotation ±10°, zoom ±10%, translation ±5%, intensity ±10%, noise σ = 0.05 |

## Results

Data: 160 training, 40 validation and 100 test volumes, split by patient.

**2D, slices containing the heart**

| Split | RV (VD) | Myocardium | LV (VG) | Mean Dice | Mean IoU |
|---|---|---|---|---|---|
| Validation | 0.912 | 0.896 | 0.959 | 0.922 | 0.857 |
| Test | 0.925 | 0.897 | 0.957 | 0.926 | 0.863 |

**3D, whole volume, per patient (ED and ES)**

| Split | RV (VD) | Myocardium | LV (VG) | Mean Dice |
|---|---|---|---|---|
| Validation | 0.883 | 0.882 | 0.936 | 0.900 |
| Test | 0.911 | 0.886 | 0.915 | 0.904 |

Mean 3D Dice by clinical group, test set: DCM 0.918, MINF 0.909, RV 0.909, NOR 0.898, HCM 0.887.

## Interpretation

- Validation and test scores are close in both 2D and 3D, which indicates that the model generalises to unseen patients.
- The training Dice stays around 0.94 while validation stays around 0.92, so the gap is smaller than in the version without augmentation.
- The myocardium remains the hardest structure, at about 0.89 Dice in both 2D and 3D.
- Differences between runs of a few thousandths are within the noise: the standard deviation of the 3D Dice across patients is between 0.03 and 0.08.

## Requirements

- Google Colab with a GPU (T4 or better recommended)
- `torch`, `numpy`, `pandas`, `matplotlib`, `scikit-learn`, `tqdm`
- The preprocessed `.npz` files and `metadata.csv` from the preprocessing notebook, stored in `MyDrive/segmentation_cardiaque/preprocessed/`

## How to run

1. Mount Google Drive and check that `DATA_ROOT` points to your project folder.
2. To reproduce the reported results, set `KERNEL_SIZE = 3` in the configuration block. Other values can be tested by changing one hyperparameter at a time.
3. Run the cells in order. Checkpoints and results are saved to `unet_runs_v3/`.
4. The notebook uses a fixed random seed (42), but results can still vary slightly between GPUs.

## Limitations

- Each figure comes from a single training run per configuration.
- Evaluation uses a local split of the 150 ACDC patients, not the official challenge leaderboard, so scores are not directly comparable with challenge results.

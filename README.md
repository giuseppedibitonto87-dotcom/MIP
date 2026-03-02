# Medical Image Processing — Liver Steatosis Challenge

> *Medical Image Processing — Master's Degree in Biomedical Engineering, Politecnico di Torino, A.A. 2021/22*

## Project

Automated segmentation of **lipid vacuoles in histological liver images** (H&E staining) to quantify hepatic steatosis, a critical step in pre-transplant graft assessment. The project was structured as a **challenge**: each group developed and optimized an independent pipeline, with comparative evaluation across groups.

**Team:** Ayman Ratbi, Eduard Ejupi, Giuseppe Di Bitonto, Tommaso Ragni

## Clinical Context

Hepatic steatosis (intracellular accumulation of triglycerides) impairs liver graft quality and increases post-transplant rejection risk. Manual assessment via histopathology is affected by inter-observer variability; we propose a fully automated CNN-based pipeline that produces a binary segmentation mask (background vs. steatosis vacuoles) and quantifies steatosis percentage, vacuole count, and spatial overlap.

## Pipeline

```
H&E Image → Macenko Stain Normalization → SE-U-Net → Threshold Sweep → Morphological Opening → Binary Mask
```

1. **Preprocessing** — Macenko stain normalization to reduce chromatic variability across slides/labs
2. **Segmentation model** — SE-U-Net (U-Net with Squeeze-and-Excitation blocks in skip connections), trained with BCEWithLogits + Dice loss, AdamW optimizer
3. **Post-processing** — Optimal threshold selection on validation set (best_threshold = 0.60) + morphological opening to remove spurious detections

## Methods & Design Choices

- **Architecture ablation study**: 8 U-Net variants tested (capacity, depth, loss functions, SE blocks, transposed convolutions) → SE-U-Net (Network 7A) selected as best
- **Preprocessing ablation study**: full pipeline vs. removal of each step → final config: Macenko only (CLAHE and median filter removed for SE-U-Net)
- **Post-processing ablation**: 4 operators compared (small object removal, hole filling, closing, opening) → morphological opening selected
- **Dataset**: 510 H&E histological images, split 80/10/10 (TRS 398 / VS 56 / TS 56) with 3× geometric augmentation on training set

## Key Results (Test Set)

| Metric | Score |
|---|---|
| Dice | 0.633 ± 0.230 |
| IoU | 0.500 ± 0.220 |
| Precision | 0.681 ± 0.244 |
| Recall | 0.661 ± 0.217 |
| Count Error | 3.98 ± 6.14 |
| Steatosis Error (%) | 0.227 ± 0.333 |

## Repository Structure

```
MIP/
├── notebooks/
│   ├── Training_and_inference.ipynb       # Model training + inference pipeline
│   ├── postpro_and_metrics.ipynb          # Post-processing and performance metrics
│   └── AS17_mockup_submission_stu.ipynb   # Final submission notebook
├── model/
│   └── training_params.json              # Training hyperparameters
│   # model_epoch_16.pth                  # Trained weights (~30 MB — excluded from git)
└── report/
    └── AS17_Report.pdf                   # Full technical report
```

> Model weights (`model_epoch_16.pth`, ~30 MB) are excluded via `.gitignore`. Available on request.

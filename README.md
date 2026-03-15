# Virtual IHC from H&E — GAN-Based Histology Image Translation

**MSc Data Science Dissertation · Kingston University London · Distinction**

> Translating routine H&E microscopy slides into IHC-like images using generative adversarial networks, without requiring perfectly paired training data.

---

## What This Project Does

Immunohistochemistry (IHC) staining reveals protein-level signals critical for cancer diagnosis — but it is slower, more expensive, and more resource-intensive than standard haematoxylin-and-eosin (H&E) staining. **Virtual IHC** aims to generate IHC-equivalent images directly from H&E slides using deep learning, potentially reducing laboratory cost and turnaround time.

This project implements and compares two image-to-image translation architectures on the **MIST-HER2 dataset** (MICCAI 2023) — a real-world challenge where H&E and IHC tile pairs are only *imperfectly* aligned, which breaks assumptions made by standard paired models.

---

## Key Results

| Model | FID ↓ | KID ↓ | SSIM ↑ | PSNR ↑ |
|---|---|---|---|---|
| Pix2Pix (paired, 200 epochs) | 325.08 | — | 0.077 | — |
| CycleGAN hybrid (ASP + perceptual, 200 epochs) | 93.30 | 53.45×10⁻³ | 0.121 | — |
| CycleGAN vanilla (110 epochs) | 75.16 | — | 0.138 | — |
| **CycleGAN vanilla (150 epochs) — Best** | **69.24** | **32.41×10⁻³** | **0.140** | — |

**Main finding:** Under imperfect registration, unpaired cycle-consistent training (CycleGAN) substantially outperforms paired conditional training (Pix2Pix). Pix2Pix's reliance on pixel-aligned pairs causes blurring and ghosting artefacts when tile alignment is noisy.

---

## Architecture and Methods

### Models Compared
- **Pix2Pix** — Paired conditional GAN (U-Net generator + PatchGAN discriminator, L1 loss)
- **CycleGAN (vanilla)** — Unpaired cycle-consistent GAN (adversarial + cycle-consistency + identity loss)
- **CycleGAN (hybrid)** — Augmented with VGG-based perceptual loss and Adaptive Supervised PatchNCE (ASP) to exploit partial correspondence while downweighting misaligned regions

### Preprocessing
- Stain normalisation using **Macenko** and **Vahadane** methods (stain vectors fixed to a single reference slide per run to prevent drift)
- Patch extraction from the MIST MICCAI 2023 dataset, HER2 cohort

### Training Stabilisation
- **Spectral Normalisation** as default to reduce GAN oscillations
- **WGAN-GP** fallback where training instability persisted
- Constant-to-linear learning rate decay schedule for late-training stability

### Evaluation Protocol
- **FID / KID** — distributional metrics computed on the full held-out split (primary metric, robust to misregistration)
- **SSIM / PSNR** — structural metrics computed on a conservatively curated, credibly aligned subset only
- **LPIPS** — perceptual quality check

---

## Dataset

[MIST Dataset — MICCAI 2023](https://github.com/Spenhouet/mist)

The HER2 cohort contains a large proportion of unpaired tiles and imperfectly aligned pairs, making it a realistic testbed for virtual staining under clinical constraints.

---

## Tech Stack

- Python 3
- PyTorch
- NumPy, Pandas
- Jupyter Notebook
- Stain normalisation: Macenko / Vahadane (custom implementation)
- Metrics: FID, KID, SSIM, PSNR, LPIPS

---

## Project Structure

```
MSc-Project/
└── H&E_to_IHC_Project_final.ipynb   # Full pipeline: preprocessing → training → evaluation
```

The notebook is structured end-to-end and reproducible: data loading, stain normalisation, model training, and quantitative evaluation are all contained within a single runnable workflow.

---

## How to Run

1. Clone this repository
2. Download the MIST HER2 dataset and place it in the expected directory (see notebook cell 1)
3. Open `H&E_to_IHC_Project_final.ipynb` in Jupyter or Google Colab
4. Run all cells in order

> **Note:** Training was performed on GPU (Google Colab). Full training runs (150–200 epochs) require significant compute time. Checkpoints are not included due to size, but the training loop is fully reproducible from scratch.

---

## Background and Motivation

Standard H&E staining is fast and cheap, but cannot directly reveal protein expression (e.g. HER2 receptor status in breast cancer). IHC adds this signal but requires additional tissue sections, reagents, and lab time. Virtual IHC using GANs could allow pathologists to derive IHC signal from existing H&E scans — useful in resource-limited settings or when tissue is scarce.

This project specifically addresses the **imperfect pairing problem**: most publicly available H&E/IHC datasets contain tiles that are not perfectly registered, which causes paired models like Pix2Pix to fail. The results confirm that unpaired, cycle-consistent training is more robust in this realistic setting.

---

## Acknowledgements

Supervised by **Dimitrios Makris**, School of Computer Science and Mathematics, Kingston University London.

Dataset: Li et al., MICCAI 2023 (MIST).

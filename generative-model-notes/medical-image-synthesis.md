# Generative Models for Medical Image Synthesis

## Overview

Generative models have become increasingly important in medical imaging for addressing data scarcity, enabling cross-modality translation, and improving downstream analysis. This note covers the key applications and considerations when applying generative models to neuroimaging.

## Application Areas

### 1. Data Augmentation
**Problem**: Medical imaging datasets are often small due to acquisition costs, privacy constraints, and annotation burden.

**Approach**: Train a generative model on available data, then synthesize additional training samples.

Key considerations:
- Generated images must preserve anatomically plausible structures.
- Conditional generation (e.g., conditioned on age, sex, diagnosis) enables targeted augmentation.
- Evaluation: Does augmentation improve downstream task performance (segmentation, classification)?

Example pipeline:
```
Real MRI dataset (N=200) → Train conditional diffusion model
→ Generate synthetic images (N=1000) → Augment training set
→ Train segmentation model on real + synthetic data
→ Evaluate on held-out real test set
```

### 2. Cross-Modality Synthesis
**Problem**: Multi-modal imaging (T1, T2, FLAIR, DWI) is ideal but not always available.

**Approach**: Synthesize missing modalities from available ones.

Applications:
- T1 → T2/FLAIR synthesis for clinical scenarios where only T1 was acquired
- MRI → CT synthesis for radiation therapy planning (avoid extra CT scan)
- PET → MRI or MRI → PET for multimodal studies

Methods:
- **Pix2Pix / cGAN**: When paired data is available
- **CycleGAN**: When paired data is unavailable
- **Conditional diffusion models**: State-of-the-art quality

### 3. Anomaly Detection
**Problem**: Supervised detection of brain lesions/abnormalities requires large annotated datasets.

**Approach**: Train a generative model only on healthy brains. At inference, regions where the model struggles to reconstruct are likely abnormal.

Pipeline:
```
Train VAE/Diffusion on healthy brains only
→ Input patient brain → Reconstruct
→ Compute pixel-wise difference (residual map)
→ Threshold → Anomaly segmentation
```

Advantages:
- No lesion annotations needed for training
- Generalizes to unseen pathologies

### 4. Super-Resolution
**Problem**: Clinical MRI often has thick slices (e.g., 5mm) or low resolution.

**Approach**: Train models to upsample low-resolution images to high-resolution counterparts.

Methods:
- SRGAN / ESRGAN adapted for 3D MRI
- Diffusion-based super-resolution (condition on low-res input)

### 5. Harmonization
**Problem**: MRI data from different scanners/sites have different intensity characteristics, introducing confounding variance.

**Approach**: Use generative models to translate images from different sites to a common style while preserving anatomical content.

Methods:
- CycleGAN between sites
- Style transfer approaches
- ComBat (statistical harmonization, non-generative baseline)

## Evaluation Metrics

### Image Quality
- **FID (Fréchet Inception Distance)**: Lower = more realistic (adapted for medical images using med-specific feature extractors)
- **SSIM (Structural Similarity Index)**: Measures structural similarity to reference
- **PSNR (Peak Signal-to-Noise Ratio)**: Pixel-level fidelity

### Clinical Utility
- **Downstream task performance**: Does using synthetic data improve segmentation/classification?
- **Radiologist evaluation**: Do experts find synthesized images clinically realistic?
- **Anatomical consistency**: Are brain structures preserved correctly?

## Ethical Considerations
- Synthetic images must not be presented as real patient data.
- Privacy: Generative models can potentially memorize training data — evaluate memorization risks.
- Bias: Models trained on non-diverse datasets may generate biased outputs.
- Regulatory: Synthetic data in clinical pipelines requires careful validation and regulatory approval.

## Key References
- Pinaya et al. (2022) — Brain imaging generation with latent diffusion models (BrainLDM)
- Kwon et al. (2019) — Generation of 3D brain MRI using auto-encoding GANs
- Shin et al. (2018) — Medical image synthesis for data augmentation and anonymization using GANs

# Diffusion Tensor Imaging (DTI)

> Basser, P. J., Mattiello, J., & LeBihan, D. (1994). MR diffusion tensor spectroscopy and imaging. *Biophysical Journal*, 66(1), 259–267.

## Summary

This foundational paper introduces the diffusion tensor model for characterizing water diffusion in biological tissues using MRI. By measuring diffusion along multiple directions, the diffusion tensor captures the anisotropic nature of water movement in organized tissues such as white matter, enabling non-invasive mapping of neural fiber architecture.

## Key Concepts

### Diffusion Tensor
- A 3×3 symmetric, positive-definite matrix **D** that describes the 3D diffusion profile at each voxel.
- Requires diffusion-weighted images (DWI) along at least 6 non-collinear gradient directions, plus one b=0 image.
- Eigenvalue decomposition yields three eigenvalues (λ₁ ≥ λ₂ ≥ λ₃) and corresponding eigenvectors.

### Scalar Measures Derived from DTI
- **Fractional Anisotropy (FA)**: Ranges from 0 (isotropic) to 1 (perfectly anisotropic). Sensitive to white matter integrity.
  ```
  FA = sqrt(3/2) * sqrt((λ₁-MD)² + (λ₂-MD)² + (λ₃-MD)²) / sqrt(λ₁² + λ₂² + λ₃²)
  ```
- **Mean Diffusivity (MD)**: Average of the three eigenvalues. Reflects overall diffusion magnitude.
  ```
  MD = (λ₁ + λ₂ + λ₃) / 3
  ```
- **Axial Diffusivity (AD)**: λ₁, diffusion along the principal eigenvector.
- **Radial Diffusivity (RD)**: (λ₂ + λ₃) / 2, diffusion perpendicular to the principal eigenvector.

### DTI Acquisition
- Typical b-value: 1000 s/mm²
- Minimum 6 gradient directions, but 30+ directions recommended for robust tensor estimation
- Higher angular resolution enables better orientation estimation

### Color-Coded FA Maps
- Red: Left-right fibers
- Green: Anterior-posterior fibers
- Blue: Superior-inferior fibers
- Brightness modulated by FA value

## Strengths
- Simple, well-understood model
- Fast acquisition and processing
- Provides intuitive scalar measures

## Limitations
- Cannot resolve crossing fibers (only one orientation per voxel)
- FA is non-specific — affected by myelination, axonal density, fiber coherence, and partial volume effects
- Single-shell acquisition (one b-value)

## Beyond DTI
- **HARDI** (High Angular Resolution Diffusion Imaging): More gradient directions for complex fiber configurations
- **CSD** (Constrained Spherical Deconvolution): Estimates fiber orientation distribution function (fODF) to resolve crossings
- **DKI** (Diffusion Kurtosis Imaging): Captures non-Gaussian diffusion for additional tissue microstructure information

## 筆記

- FA 值的解讀要小心，不能直接說 FA 低 = 白質受損，因為太多因素會影響（crossing fibers、partial volume 等等）
- 這篇是 1994 年的經典，但現在 DTI 的 single tensor model 其實已經不太夠用了，CSD 之類的方法更好
- b-value 的選擇：臨床常用 b=1000，但研究上 multi-shell (b=1000, 2000, 3000) 可以做更多分析
- TODO: 整理一下 DTI vs DKI vs NODDI 的比較表

## Relevance
DTI remains the most widely used diffusion MRI technique, forming the basis for tractography and white matter analysis in both research and clinical settings.

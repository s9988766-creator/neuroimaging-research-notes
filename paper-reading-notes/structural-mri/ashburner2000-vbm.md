# Voxel-Based Morphometry (VBM)

> Ashburner, J., & Friston, K. J. (2000). Voxel-based morphometry — the methods. *NeuroImage*, 11(6), 805–821.

## Summary

This paper introduces Voxel-Based Morphometry (VBM), a whole-brain, automated technique for detecting regional differences in brain tissue composition (primarily gray matter) between groups. VBM operates on a voxel-by-voxel basis after spatially normalizing all brain images to the same stereotactic space.

## Key Concepts

### VBM Processing Pipeline
1. **Spatial normalization**: Warp each subject's T1 image to a common template (e.g., MNI152) using nonlinear registration.
2. **Segmentation**: Classify each voxel as gray matter (GM), white matter (WM), or cerebrospinal fluid (CSF).
3. **Modulation** (optional): Scale GM maps by the Jacobian determinant of the warp field to preserve total tissue volume.
4. **Smoothing**: Apply a Gaussian kernel (typically 8–12 mm FWHM) to improve signal-to-noise ratio and satisfy assumptions for statistical testing.
5. **Statistical analysis**: Use a general linear model (GLM) at each voxel to test for group differences, controlling for covariates (age, sex, total intracranial volume).

### Modulated vs. Unmodulated VBM
- **Unmodulated**: Tests for differences in regional GM *concentration* (proportion of GM at each voxel).
- **Modulated**: Tests for differences in regional GM *volume* (accounts for expansion/contraction during normalization).

### Multiple Comparisons Correction
- Family-wise error (FWE) correction using Random Field Theory
- False Discovery Rate (FDR)
- Threshold-Free Cluster Enhancement (TFCE) — commonly used with FSL's `randomise`

## Strengths
- Automated, whole-brain analysis without a priori region selection
- Scalable to large datasets
- Straightforward interpretation

## Limitations
- Results are sensitive to registration quality
- Smoothing kernel size affects sensitivity and spatial resolution
- Cannot distinguish between cortical thickness changes and cortical folding differences

## Software Implementation
- **SPM** (Statistical Parametric Mapping): Original implementation
- **FSL-VBM**: FSL-based pipeline using FMRIB tools
- **CAT12**: Extension of SPM with improved segmentation and surface-based measures

## Questions / TODO

- modulated vs unmodulated 這兩個我一開始搞不太懂，簡單說 modulated 看的是「體積」的差異，unmodulated 看的是「濃度/密度」的差異
- 想知道 VBM 跟 FreeSurfer cortical thickness 分析相比，哪個對 Alzheimer's 更敏感？
- TODO: 找幾篇比較 VBM vs SBM (surface-based morphometry) 的 review paper
- smoothing kernel size 的選擇好像很影響結果，8mm 是最常見的但不一定是最好的

## Relevance
VBM is one of the most commonly used techniques for studying structural brain differences in neurological and psychiatric disorders (e.g., Alzheimer's disease, schizophrenia, depression). It provides a complementary approach to surface-based methods like FreeSurfer.

# FreeSurfer

> Fischl, B. (2012). FreeSurfer. *NeuroImage*, 62(2), 774–781.

## Summary

This paper provides a comprehensive overview of the FreeSurfer software suite, which is widely used for processing and analyzing structural MRI data of the human brain. FreeSurfer automates the reconstruction of cortical surface models and performs cortical parcellation, subcortical segmentation, and various morphometric measurements.

## Key Concepts

### Cortical Surface Reconstruction
- FreeSurfer generates two primary cortical surfaces: the **white surface** (gray/white boundary) and the **pial surface** (gray/CSF boundary).
- The reconstruction pipeline includes motion correction, Talairach transformation, intensity normalization, skull stripping, and topology correction.

### Cortical Parcellation
- The Desikan-Killiany atlas divides each hemisphere into 34 regions based on sulcal and gyral anatomy.
- The Destrieux atlas offers finer parcellation with 74 regions per hemisphere, based on sulcal depth.

### Subcortical Segmentation
- Automated labeling of subcortical structures (hippocampus, amygdala, caudate, putamen, thalamus, etc.) using a probabilistic atlas.
- Segmentation relies on voxel intensity, spatial location, and neighborhood label information.

### Morphometric Measures
- **Cortical thickness**: Measured as the distance between white and pial surfaces at each vertex.
- **Surface area**: Computed from the tessellated surface mesh.
- **Cortical volume**: Product of thickness and surface area.
- **Curvature**: Captures local folding patterns of the cortical surface.

## Pipeline Overview

```
recon-all -s <subject> -i <input_T1.nii.gz> -all
```

Main stages:
1. Motion correction and averaging (if multiple T1 inputs)
2. Talairach transformation
3. Intensity normalization
4. Skull stripping
5. White matter segmentation
6. Surface tessellation and refinement
7. Surface inflation and registration to atlas
8. Cortical parcellation and subcortical segmentation

## Strengths
- Fully automated, reproducible pipeline
- Widely validated across populations and scanner types
- Rich set of morphometric outputs

## Limitations
- Long processing time (~6–8 hours per subject)
- Sensitive to motion artifacts and poor image quality
- Manual editing may be needed for subjects with significant pathology

## My Notes

- 第一次跑 recon-all 花了快 8 小時，結果發現 skull stripping 有問題要重跑... 下次先檢查 T1 品質
- Desikan-Killiany vs Destrieux 的差異主要在 parcellation 的精細度，大部分 paper 用的是 Desikan-Killiany
- TODO: 試試看用 FastSurfer (deep learning-based) 跟原版 FreeSurfer 比較速度和準確度
- 老師說 cortical thickness 跟 surface area 是遺傳上獨立的，這點蠻有趣的

## Relevance
FreeSurfer is the standard tool for structural brain analysis in large-scale studies (e.g., ADNI, UK Biobank, HCP). Understanding its pipeline is essential for any neuroimaging researcher working with cortical morphometry.

# Resting-State Functional Connectivity

> Biswal, B., Yetkin, F. Z., Haughton, V. M., & Hyde, J. S. (1995). Functional connectivity in the motor cortex of resting human brain using echo-planar MRI. *Magnetic Resonance in Medicine*, 34(4), 537–541.

## Summary

This landmark paper demonstrates that low-frequency fluctuations (< 0.1 Hz) in the BOLD signal during rest are temporally correlated between functionally related brain regions. Specifically, the authors showed that the left and right motor cortices exhibit strong functional connectivity even in the absence of a task, establishing the foundation for resting-state fMRI (rs-fMRI).

## Key Concepts

### Resting-State fMRI
- Subjects lie in the scanner without performing any task (eyes open or closed).
- BOLD signal fluctuations at low frequencies (0.01–0.1 Hz) reflect spontaneous neural activity.
- Temporal correlations between regions define **functional connectivity**.

### Resting-State Networks (RSNs)
Major networks identified in subsequent studies:
- **Default Mode Network (DMN)**: Medial prefrontal cortex, posterior cingulate cortex, precuneus, angular gyrus. Active during self-referential thought and mind-wandering.
- **Sensorimotor Network**: Primary motor and somatosensory cortices.
- **Visual Network**: Primary and secondary visual cortices.
- **Salience Network**: Anterior insula, dorsal anterior cingulate cortex.
- **Frontoparietal Network**: Dorsolateral prefrontal cortex, posterior parietal cortex.

### Preprocessing for rs-fMRI
1. Slice timing correction
2. Motion correction (realignment)
3. Registration to structural image and standard space
4. Nuisance regression (motion parameters, WM/CSF signals)
5. Bandpass filtering (0.01–0.1 Hz)
6. Spatial smoothing

### Analysis Methods
- **Seed-based correlation**: Correlate the time series of a seed region with all other voxels.
- **Independent Component Analysis (ICA)**: Data-driven decomposition into spatially independent networks.
- **Graph theory**: Model the brain as a network of nodes (regions) and edges (correlations).

## Strengths
- No task design needed — easy to acquire, even in clinical populations
- Reveals intrinsic brain organization
- Highly reproducible across subjects and sessions

## Limitations
- Susceptible to motion artifacts (especially head motion)
- Global signal regression is controversial
- Cannot infer causal or directional connectivity

## 一些想法

- 這篇 1995 年的 paper 才 5 頁，但影響力超大，開啟了整個 resting-state fMRI 的領域
- Global signal regression 到底要不要做？看了幾篇 paper 意見很分歧，Murphy & Fox (2017) 有一篇 review 專門討論這個
- head motion 真的是 rs-fMRI 最大的敵人，特別是做兒童或老人的研究... Power et al. (2012) 的 framewise displacement 是標準的 motion QC 指標
- TODO: 讀 Smith et al. (2009) 比較 ICA 跟 seed-based 的差異 ✓ (已讀，筆記在同資料夾)

## Relevance
Resting-state fMRI has become one of the most widely used neuroimaging paradigms, with applications in studying brain development, aging, neurological disorders, and psychiatric conditions.

# nilearn — Python Neuroimaging in Machine Learning

## Overview
nilearn is a Python library for statistical learning on neuroimaging data. It provides tools for decoding, connectivity analysis, and visualization, built on top of scikit-learn.

- **Website**: https://nilearn.github.io/
- **Install**: `pip install nilearn`
- **Dependencies**: numpy, scipy, scikit-learn, nibabel, matplotlib

## Basic Usage

### Loading and Visualizing Brain Images

```python
import nibabel as nib
from nilearn import plotting, datasets, image

# Load a NIfTI image
img = nib.load('brain.nii.gz')
print(f"Shape: {img.shape}, Affine:\n{img.affine}")

# Plot anatomical image
plotting.plot_anat(img, title="T1 Image")
plotting.show()

# Plot statistical map on template
plotting.plot_stat_map(stat_img, bg_img=anat_img,
                       threshold=3.0, title="Activation Map")
```

### Functional Connectivity Analysis

```python
from nilearn import datasets, input_data
from nilearn.connectome import ConnectivityMeasure

# Load atlas
atlas = datasets.fetch_atlas_schaefer_2018(n_rois=100)

# Extract time series from parcellated regions
masker = input_data.NiftiLabelsMasker(
    labels_img=atlas.maps,
    standardize=True,
    detrend=True,
    low_pass=0.1,
    high_pass=0.01,
    t_r=2.0
)
time_series = masker.fit_transform(func_img, confounds=confounds)

# Compute correlation matrix
conn = ConnectivityMeasure(kind='correlation')
correlation_matrix = conn.fit_transform([time_series])[0]

# Visualize
plotting.plot_matrix(correlation_matrix, labels=atlas.labels,
                     colorbar=True, vmax=0.8, vmin=-0.8)
```

### Brain Decoding (Classification)

```python
from nilearn.decoding import Decoder

# Decode brain states from fMRI
decoder = Decoder(
    estimator='svc',
    mask=mask_img,
    standardize=True,
    screening_percentile=5,
    cv=5
)
decoder.fit(func_imgs, conditions)
print(f"Accuracy: {decoder.cv_scores_['accuracy'].mean():.2f}")

# Plot weight map
plotting.plot_stat_map(decoder.coef_img_['face'],
                       title="Face vs. House Decoding Weights")
```

### Surface-Based Visualization

```python
from nilearn import surface, plotting

# Plot data on cortical surface
fsaverage = datasets.fetch_surf_fsaverage()
plotting.plot_surf_stat_map(
    fsaverage.infl_left,
    stat_map=texture,
    hemi='left',
    colorbar=True,
    title='Cortical Thickness'
)
```

## Common Atlases Available

| Atlas | Regions | Use Case |
|-------|---------|----------|
| `fetch_atlas_schaefer_2018` | 100–1000 | Functional connectivity |
| `fetch_atlas_destrieux_2009` | 148 | Anatomical parcellation |
| `fetch_atlas_aal` | 116 | ROI-based analysis |
| `fetch_atlas_harvard_oxford` | 48/21 | Cortical/subcortical ROIs |

## Tips
- Use `NiftiMasker` for whole-brain voxel-level analysis, `NiftiLabelsMasker` for ROI-based analysis.
- Always check confound regression — include motion parameters, WM, CSF signals.
- nilearn works seamlessly with scikit-learn pipelines for classification and regression.

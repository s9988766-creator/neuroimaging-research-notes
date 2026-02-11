# ANTs (Advanced Normalization Tools) — Quick Reference

## Overview
ANTs is a state-of-the-art software for image registration and segmentation. Its SyN (Symmetric Normalization) algorithm is consistently ranked among the best for brain image registration.

- **Website**: http://stnava.github.io/ANTs/
- **Install**: Build from source or use ANTsPy (Python wrapper)
- **Also available**: ANTsPy (`pip install antspyx`)

## Key Tools

### Brain Extraction
```bash
# ANTs brain extraction (uses a template-based approach)
antsBrainExtraction.sh -d 3 \
    -a T1.nii.gz \
    -e OASIS_template.nii.gz \
    -m OASIS_prior.nii.gz \
    -o output_

# Output: output_BrainExtractionBrain.nii.gz
```

### Registration (SyN)
```bash
# Full registration pipeline (rigid + affine + SyN)
antsRegistrationSyN.sh -d 3 \
    -f fixed.nii.gz \
    -m moving.nii.gz \
    -o output_ \
    -t s  # s=SyN, b=BSplineSyN, a=affine only

# Apply transforms
antsApplyTransforms -d 3 \
    -i input.nii.gz \
    -r reference.nii.gz \
    -o output.nii.gz \
    -t output_1Warp.nii.gz \
    -t output_0GenericAffine.mat
```

### Cortical Thickness (ANTs Cortical Thickness Pipeline)
```bash
# Full pipeline: registration, segmentation, cortical thickness
antsCorticalThickness.sh -d 3 \
    -a T1.nii.gz \
    -e template.nii.gz \
    -m template_prob.nii.gz \
    -p priors/prior%d.nii.gz \
    -o output_
```

### Atrophy Computation (Jacobian)
```bash
# Compute log Jacobian determinant from warp field
CreateJacobianDeterminantImage 3 warp.nii.gz jacobian.nii.gz 1 1
# The log Jacobian indicates local volume change:
#   Positive = expansion, Negative = contraction
```

## ANTsPy (Python Interface)

```python
import ants

# Read images
fixed = ants.image_read('template.nii.gz')
moving = ants.image_read('subject_T1.nii.gz')

# Registration
result = ants.registration(
    fixed=fixed,
    moving=moving,
    type_of_transform='SyN'
)
warped = result['warpedmovout']
warped.to_file('warped_subject.nii.gz')

# Brain extraction
brain = ants.brain_extraction(moving, modality='t1')

# Segmentation (Atropos)
seg = ants.atropos(a=moving, m='[0.2, 1x1x1]',
                    c='[5, 0]', i='kmeans[3]', x=mask)
```

## Why ANTs?
- SyN registration consistently outperforms other methods in benchmarks.
- Handles large deformations well (e.g., registering infant to adult brains).
- Modular: combine rigid, affine, and deformable transforms.
- ANTsPy makes it easy to integrate with Python-based ML pipelines.

## Tips
- For group studies, build a **study-specific template** using `antsMultivariateTemplateConstruction2.sh`.
- Use `N4BiasFieldCorrection` for intensity inhomogeneity correction before registration.
- Combine ANTs registration with FreeSurfer parcellation for robust analysis.

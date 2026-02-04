# FSL (FMRIB Software Library) — Quick Reference

## Overview
FSL is a comprehensive library of analysis tools for FMRI, MRI, and DTI brain imaging data. Developed by the Analysis Group at FMRIB, University of Oxford.

- **Website**: https://fsl.fmrib.ox.ac.uk/fsl/
- **Current version**: 6.x
- **License**: Free for non-commercial use

## Installation

```bash
# Install via fslinstaller.py
python fslinstaller.py

# Set environment variables
export FSLDIR=/usr/local/fsl
source $FSLDIR/etc/fslconf/fsl.sh
export PATH=$FSLDIR/bin:$PATH
```

## Key Tools

### BET (Brain Extraction Tool)
```bash
# Skull stripping
bet input.nii.gz output_brain.nii.gz -f 0.5 -g 0 -R

# -f: fractional intensity threshold (0→1, smaller = larger brain outline)
# -R: robust brain centre estimation
```

### FLIRT / FNIRT (Registration)
```bash
# Linear registration (affine)
flirt -in subject_brain.nii.gz -ref MNI152_T1_2mm_brain.nii.gz \
      -out subject2mni.nii.gz -omat subject2mni.mat -dof 12

# Non-linear registration
fnirt --in=subject_T1.nii.gz --ref=$FSLDIR/data/standard/MNI152_T1_2mm.nii.gz \
      --aff=subject2mni.mat --cout=subject2mni_warp
```

### FAST (Tissue Segmentation)
```bash
# Segment into GM, WM, CSF
fast -t 1 -n 3 -o output brain.nii.gz
# Produces: output_pve_0 (CSF), output_pve_1 (GM), output_pve_2 (WM)
```

### FEAT (fMRI Analysis)
```bash
# GUI-based setup
Feat &

# Command-line
feat design.fsf
```

### FSL-VBM
```bash
# Run VBM pipeline
fslvbm_1_bet     # Brain extraction
fslvbm_2_template # Create study-specific template
fslvbm_3_proc    # Register, segment, modulate, smooth
randomise -i GM_mod_merg_s3 -o results -d design.mat -t design.con -T 5000
```

### TBSS (Tract-Based Spatial Statistics)
```bash
# For DTI group analysis
tbss_1_preproc *.nii.gz   # Preprocessing
tbss_2_reg -T              # Registration to target
tbss_3_postreg -S          # Post-registration
tbss_4_prestats 0.2        # Pre-statistics thresholding
randomise -i all_FA_skeletonised -o tbss -d design.mat -t design.con -n 5000 --T2
```

### MELODIC (ICA for fMRI)
```bash
# Single-subject ICA
melodic -i func.nii.gz -o melodic_output --nobet --report

# Group ICA (temporal concatenation)
melodic -i input_list.txt -o group_melodic --report
```

## Useful Utilities

```bash
# View images
fsleyes image.nii.gz &

# Get image info
fslinfo image.nii.gz
fslhd image.nii.gz

# Math operations
fslmaths input.nii.gz -thr 0.5 -bin output_mask.nii.gz

# Extract ROI mean
fslstats image.nii.gz -k mask.nii.gz -M
```

## Tips
- Use `fsleyes` as the primary viewer (replacement for `fslview`).
- `fslmaths` is extremely versatile — master it for image manipulation.
- For group statistics, prefer `randomise` (permutation testing) over parametric methods.

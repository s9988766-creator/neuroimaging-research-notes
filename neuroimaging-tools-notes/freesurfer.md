# FreeSurfer — Quick Reference

## Overview
FreeSurfer is an open-source software suite for processing and analyzing structural brain MRI data. It provides automated cortical reconstruction, parcellation, subcortical segmentation, and a wide range of morphometric measurements.

- **Website**: https://surfer.nmr.mgh.harvard.edu/
- **Current version**: 7.x
- **License**: Free for non-commercial use

## Installation

```bash
# Download from official website and extract
export FREESURFER_HOME=/usr/local/freesurfer
source $FREESURFER_HOME/SetUpFreeSurfer.sh

# Set subjects directory
export SUBJECTS_DIR=/path/to/your/subjects
```

## Core Commands

### Full Reconstruction Pipeline
```bash
# Run complete pipeline (~6-8 hours per subject)
recon-all -s subject001 -i T1.nii.gz -all

# Run in parallel for multiple subjects
parallel recon-all -s {} -i {}.nii.gz -all ::: sub-01 sub-02 sub-03
```

### Checking Results
```bash
# View surfaces in FreeView
freeview -v subject001/mri/T1.mgz \
         -v subject001/mri/aparc+aseg.mgz:colormap=lut \
         -f subject001/surf/lh.pial:edgecolor=red \
         -f subject001/surf/lh.white:edgecolor=blue
```

### Extracting Statistics
```bash
# Cortical thickness and area per region (Desikan-Killiany atlas)
aparcstats2table --subjects sub-01 sub-02 sub-03 \
    --hemi lh --meas thickness --tablefile lh_thickness.txt

# Subcortical volumes
asegstats2table --subjects sub-01 sub-02 sub-03 \
    --meas volume --tablefile subcortical_volumes.txt
```

### Quality Control
```bash
# Generate QC snapshots
# Use ENIGMA QC protocol or FreeSurfer QA tools
recon-all -s subject001 -qcache
```

## Key Output Files

| File | Description |
|------|-------------|
| `mri/T1.mgz` | Conformed T1 image (256×256×256, 1mm isotropic) |
| `mri/brain.mgz` | Skull-stripped brain |
| `mri/aparc+aseg.mgz` | Parcellation + segmentation volume |
| `surf/lh.white` | Left hemisphere white surface |
| `surf/lh.pial` | Left hemisphere pial surface |
| `surf/lh.thickness` | Cortical thickness per vertex |
| `stats/lh.aparc.stats` | Left hemisphere regional statistics |
| `stats/aseg.stats` | Subcortical volume statistics |

## Common Issues and Solutions

**Problem**: Poor skull stripping
```bash
# Adjust watershed threshold
recon-all -s subject001 -skullstrip -wsthresh 25 -clean-BM -no-wsgcam
```

**Problem**: White matter segmentation errors
```bash
# Edit wm.mgz manually in FreeView, then re-run
recon-all -s subject001 -autorecon2 -autorecon3
```

**Problem**: Pial surface extends into dura
```bash
# Adjust pial surface parameters
recon-all -s subject001 -autorecon-pial
```

## Tips
- Always visually inspect output surfaces overlaid on T1 for every subject.
- For longitudinal studies, use the longitudinal pipeline: `recon-all -long`.
- For infant brains, use **infant FreeSurfer** or alternative tools (e.g., dHCP pipeline).

## 我遇到的問題

- 在 lab 的 server 上裝 FreeSurfer 7.4 的時候 license file 要另外申請，免費但要填表
- 第一次跑 recon-all 忘記設 SUBJECTS_DIR，結果 output 全部跑到預設路徑去了
- FreeView 在 ssh 遠端連線的時候需要 X11 forwarding (`ssh -X`)，不然開不起來
- TODO: 研究一下 FastSurfer 能不能完全取代 FreeSurfer，看起來跑一個 subject 只要幾分鐘

# Correspondence of the Brain's Functional Architecture During Activation and Rest

> Smith, S. M., Fox, P. T., Miller, K. L., et al. (2009). Correspondence of the brain's functional architecture during activation and rest. *PNAS*, 106(31), 13040–13045.

## Summary

This study demonstrates that the spatial patterns of resting-state networks (RSNs) identified through ICA closely correspond to activation patterns observed in task-based fMRI. Using BrainMap meta-analyses of ~7,000 task-fMRI experiments, the authors show a strong spatial overlap between resting-state and task-evoked networks, supporting the idea that intrinsic brain organization reflects the functional architecture used during active cognition.

## Key Concepts

### Independent Component Analysis (ICA) for rs-fMRI
- ICA decomposes 4D fMRI data into spatially independent components, each with a unique time course.
- **Temporal concatenation group ICA**: Concatenate all subjects' data along time dimension, then run ICA once.
- Components are classified as signal (neural networks) or noise (motion, physiological artifacts).

### BrainMap Meta-Analysis
- BrainMap is a database of published fMRI/PET activation coordinates.
- The authors applied ICA to the BrainMap coordinate database to extract co-activation patterns across thousands of experiments.
- These task-based co-activation maps were then compared to resting-state ICA maps.

### Key Finding
- 10 major RSNs showed one-to-one correspondence with BrainMap-derived co-activation networks.
- This suggests that spontaneous BOLD fluctuations at rest are not random noise but reflect meaningful neural organization.

## Identified Networks and Their Task Correspondences

| RSN | Task Correspondence |
|-----|-------------------|
| Default Mode | Self-referential processing, memory retrieval |
| Lateral Visual | Visual processing tasks |
| Medial Visual | Visual processing (retinotopic) |
| Auditory | Auditory and speech tasks |
| Sensorimotor | Motor execution tasks |
| Executive Control | Working memory, cognitive control |
| Right Frontoparietal | Attentional tasks |
| Left Frontoparietal | Language tasks |
| Cerebellar | Motor and cognitive tasks |

## Methodological Notes
- ICA dimensionality (number of components) affects network granularity.
- Low dimensionality (~20): Large-scale networks
- High dimensionality (~70–100): Finer sub-networks
- Tools: FSL MELODIC for ICA decomposition

## Relevance
This paper solidified the importance of resting-state fMRI as a tool for studying brain organization and provided a framework for interpreting RSNs in terms of cognitive function.

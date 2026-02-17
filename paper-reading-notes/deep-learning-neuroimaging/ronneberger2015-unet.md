# U-Net: Convolutional Networks for Biomedical Image Segmentation

> Ronneberger, O., Fischer, P., & Brox, T. (2015). U-Net: Convolutional Networks for Biomedical Image Segmentation. *MICCAI 2015*, 234–241.

## Summary

U-Net is a convolutional neural network architecture designed for biomedical image segmentation. Its encoder-decoder structure with skip connections enables precise localization while maintaining context, and it works effectively even with limited training data through aggressive data augmentation.

## Architecture

```
Input Image
    │
    ▼
[Encoder Path]          [Decoder Path]
Conv 3×3 → Conv 3×3     Conv 3×3 → Conv 3×3
    │ MaxPool 2×2            ▲ UpConv 2×2
Conv 3×3 → Conv 3×3 ──→ Conv 3×3 → Conv 3×3   (skip connection)
    │ MaxPool 2×2            ▲ UpConv 2×2
Conv 3×3 → Conv 3×3 ──→ Conv 3×3 → Conv 3×3   (skip connection)
    │ MaxPool 2×2            ▲ UpConv 2×2
Conv 3×3 → Conv 3×3 ──→ Conv 3×3 → Conv 3×3   (skip connection)
    │ MaxPool 2×2            ▲ UpConv 2×2
    └── Conv 3×3 → Conv 3×3 ┘  (bottleneck)
                    │
                    ▼
              1×1 Conv (output)
```

### Key Design Principles
- **Encoder (contracting path)**: Captures context through repeated convolution + max-pooling. Doubles feature channels at each level.
- **Decoder (expanding path)**: Up-convolutions + concatenation with cropped feature maps from the encoder (skip connections). Recovers spatial resolution.
- **Skip connections**: Concatenate encoder features with decoder features at corresponding levels, preserving fine-grained spatial information.
- **1×1 convolution**: Final layer maps features to the desired number of classes.

## Key Contributions
1. **Skip connections** enable the network to combine low-level (edges, textures) and high-level (semantic) features for precise boundary delineation.
2. **Data augmentation**: Elastic deformations, rotations, and flips to handle limited training data (common in medical imaging).
3. **Overlap-tile strategy**: Enables segmentation of arbitrarily large images by processing overlapping patches.
4. **Weighted loss function**: Applies higher weights to pixels near cell boundaries to encourage separation of touching objects.

## Applications in Neuroimaging
- **Brain tissue segmentation**: GM, WM, CSF classification
- **Tumor segmentation**: BraTS challenge (glioma segmentation)
- **Lesion segmentation**: White matter hyperintensities, stroke lesions
- **Hippocampus segmentation**: For volumetric studies in Alzheimer's disease

## Variants
- **3D U-Net**: Extends U-Net to volumetric data (Çiçek et al., 2016)
- **V-Net**: Uses volumetric convolutions with residual connections
- **nnU-Net**: Self-configuring framework that automatically adapts U-Net architecture and preprocessing to new datasets (Isensee et al., 2021)
- **Attention U-Net**: Adds attention gates to skip connections for improved focus on relevant features

## 心得

- skip connection 的設計真的很直覺，把 encoder 的空間細節直接接到 decoder，segmentation 的邊界就變得很清楚
- 最近看到 nnU-Net 幾乎在所有 medical image segmentation challenge 都是 top，核心就是自動化調參的 U-Net
- 3D U-Net 用在 brain MRI 上比 2D 效果好很多，但 GPU memory 需求也大很多
- 有試著用 PyTorch 實作簡單版的 2D U-Net，encoder 用 ResNet pretrained weights 效果不錯

## Relevance
U-Net and its variants are the dominant architecture for medical image segmentation, forming the backbone of most state-of-the-art brain segmentation pipelines.

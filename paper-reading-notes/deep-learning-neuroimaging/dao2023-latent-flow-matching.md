# Flow Matching in Latent Space (LFM)

> Dao, Q., Phung, H., Nguyen, B., & Tran, A. (2023). Flow Matching in Latent Space. *arXiv preprint arXiv:2307.08698*.

- **GitHub**: https://github.com/VinAIResearch/LFM
- **Project page**: https://vinairesearch.github.io/LFM/
- **arXiv**: https://arxiv.org/abs/2307.08698

## Summary

This paper proposes **Latent Flow Matching (LFM)**, which applies the flow matching framework in the latent space of pretrained autoencoders rather than in pixel space. This is analogous to how Latent Diffusion Models (LDM / Stable Diffusion) moved diffusion to latent space — but here applied to the flow matching paradigm. The approach significantly improves computational efficiency and enables high-resolution image synthesis on limited resources.

## Motivation

- Flow matching (FM) has shown strong empirical performance with simpler training than diffusion models.
- However, prior FM methods operated in **pixel space**, which is computationally expensive for high-resolution images.
- Latent diffusion models (LDM) proved that moving the generative process to latent space greatly reduces compute costs — but no one had done this for flow matching.
- This paper bridges that gap: **FM + latent space = LFM**.

## Method

### Architecture Overview

```
Training:
  Image x → Pretrained Encoder E → Latent z₁ = E(x)
  Sample z₀ ~ N(0, I)
  Interpolate: zₜ = (1 - t)z₀ + tz₁
  Train velocity network vθ(zₜ, t) to predict target velocity (z₁ - z₀)

Sampling:
  z₀ ~ N(0, I) → ODE solve with vθ → z₁ → Pretrained Decoder D → x̂ = D(z₁)
```

### Key Components

**1. Latent Flow Matching Objective**

The training loss operates entirely in latent space:
```
L_LFM = E_{t, z₀, z₁} [ ||vθ(zₜ, t) - (z₁ - z₀)||² ]
```
where:
- z₁ = E(x) is the encoded data sample
- z₀ ~ N(0, I) is Gaussian noise
- zₜ = (1-t)z₀ + tz₁ is the linear interpolation
- vθ is the learned velocity field (U-Net in latent space)

**2. Classifier-Free Velocity Field**

Inspired by classifier-free guidance in diffusion models, but adapted for the velocity field:
```
ṽ(zₜ, t, c) = (1 + w) · vθ(zₜ, t, c) - w · vθ(zₜ, t, ∅)
```
where:
- c is the condition (class label, semantic map, etc.)
- ∅ is the null condition (unconditional)
- w is the guidance scale

During training, the condition is randomly dropped with probability p (e.g., 10%) to enable both conditional and unconditional generation.

**3. Theoretical Guarantee**

The authors prove that the Wasserstein-2 distance between the reconstructed distribution (decoded latent flow output) and the true data distribution is upper-bounded by the latent flow matching objective plus the autoencoder reconstruction error:
```
W₂(p_generated, p_data) ≤ C · (L_LFM + L_reconstruction)
```
This provides theoretical justification for performing flow matching in latent space.

### Pretrained Autoencoder

- Uses the same autoencoder architecture as in Latent Diffusion Models (Rombach et al., 2022).
- The autoencoder is trained with a combination of reconstruction loss, perceptual loss, and KL regularization.
- Compression ratio: typically 8× spatial downsampling (e.g., 256×256 → 32×32 latent).

## Conditional Generation Tasks

LFM supports multiple conditioning mechanisms:

| Task | Condition | Method |
|------|-----------|--------|
| Class-conditional generation | Class label | Classifier-free velocity field |
| Image inpainting | Masked image | Concatenate mask + masked latent as input |
| Semantic-to-image | Semantic map | Encode semantic map, concatenate with noisy latent |

## Experiments

### Datasets
- CelebA-HQ 256×256
- FFHQ 256×256
- LSUN Church 256×256
- LSUN Bedroom 256×256
- ImageNet 256×256

### Key Results
- Achieves competitive FID scores compared to pixel-space flow matching and diffusion models.
- Significantly faster training and sampling due to operating in compressed latent space.
- Fewer number of function evaluations (NFE) needed during ODE solving compared to pixel-space FM.

## Comparison: Pixel-Space FM vs. Latent FM

| Aspect | Pixel-Space FM | Latent FM (this paper) |
|--------|---------------|----------------------|
| Operating space | Raw image (e.g., 256×256×3) | Latent (e.g., 32×32×4) |
| Compute cost | High | Significantly lower |
| Training time | Long | Faster |
| NFE for sampling | More steps needed | Fewer steps |
| Image quality | Good | Comparable or better |
| Requires pretrained AE | No | Yes |

## Relevance to Neuroimaging

This paper is highly relevant to neuroimaging applications:

- **Efficient brain MRI synthesis**: Operating in latent space makes it feasible to train generative models on 3D brain volumes with limited GPU memory.
- **MRI normalization**: The flow matching framework could learn a continuous transformation from individual brain anatomy to a template space.
- **Multi-modal synthesis**: Conditional LFM could enable T1→T2 or MRI→PET synthesis in latent space.
- **Data augmentation**: Generate realistic synthetic brain images more efficiently than pixel-space methods.
- **Anomaly detection**: Train on healthy brain latents, detect anomalies via reconstruction quality in latent space.

## Code Structure (GitHub repo)

```
VinAIResearch/LFM/
├── train_flow_latent.py       # Main training script
├── models/
│   ├── unet.py                # U-Net velocity network
│   └── autoencoder.py         # Pretrained autoencoder
├── datasets/                  # Data loading utilities
├── configs/                   # Training configurations
└── sample.py                  # Sampling / inference script
```

## Key Takeaways

1. Flow matching in latent space is a natural and effective extension, just as LDM was for diffusion models.
2. Classifier-free guidance transfers well to the velocity field formulation.
3. The theoretical bound connecting latent FM loss to data distribution quality provides a principled foundation.
4. This approach makes flow matching practical for high-resolution and potentially 3D medical image generation.

## 我的想法

- 這篇基本上就是把 Stable Diffusion 的 "latent space" 概念搬到 flow matching，idea 很 clean
- W₂ bound 那段數學我還沒完全看懂，有空要再推一次... 主要是 Theorem 1 的證明
- 想試試看拿 VinAI 的 code 改成 3D 版本，用 brain MRI 做 unconditional generation
- 跟 Lipman et al. (2023) 的原版 flow matching 比起來，latent space 的優勢在高解析度很明顯
- TODO: 看看 code 裡面 autoencoder 的部分能不能直接用 Rombach et al. 的 pretrained weights
- TODO: 跟 BrainLDM (Pinaya et al., 2022) 比較一下，那篇是 latent diffusion 用在 brain imaging 的

# Denoising Diffusion Probabilistic Models (DDPM)

> Ho, J., Jain, A., & Abbeel, P. (2020). Denoising Diffusion Probabilistic Models. *NeurIPS 2020*.

## Core Idea

Diffusion models learn to generate data by reversing a gradual noising process. The **forward process** incrementally adds Gaussian noise to data over T steps until it becomes pure noise. The **reverse process** learns to denoise step by step, recovering the original data distribution.

## Forward Process (Diffusion)

Given data x₀ ~ q(x₀), the forward process adds noise:

```
q(xₜ | xₜ₋₁) = N(xₜ; √(1-βₜ) xₜ₋₁, βₜI)
```

where β₁, β₂, ..., βT is a variance schedule (e.g., linear from 1e-4 to 0.02).

Closed-form sampling at any timestep t:
```
q(xₜ | x₀) = N(xₜ; √ᾱₜ x₀, (1-ᾱₜ)I)
where αₜ = 1 - βₜ,  ᾱₜ = ∏ₛ₌₁ᵗ αₛ
```

This means we can directly compute xₜ = √ᾱₜ · x₀ + √(1-ᾱₜ) · ε, where ε ~ N(0, I).

## Reverse Process (Denoising)

The model learns:
```
pθ(xₜ₋₁ | xₜ) = N(xₜ₋₁; μθ(xₜ, t), σₜ²I)
```

In practice, the neural network εθ(xₜ, t) predicts the noise ε added at step t.

## Training Objective

Simplified loss (noise prediction):
```
L_simple = E_{t, x₀, ε} [ ||ε - εθ(√ᾱₜ x₀ + √(1-ᾱₜ)ε, t)||² ]
```

Training algorithm:
1. Sample x₀ from dataset
2. Sample t ~ Uniform({1, ..., T})
3. Sample ε ~ N(0, I)
4. Compute xₜ = √ᾱₜ · x₀ + √(1-ᾱₜ) · ε
5. Train εθ to predict ε from (xₜ, t)

## Sampling

Starting from xT ~ N(0, I), iteratively denoise:
```
xₜ₋₁ = (1/√αₜ)(xₜ - (βₜ/√(1-ᾱₜ))εθ(xₜ, t)) + σₜz
where z ~ N(0, I) for t > 1, z = 0 for t = 1
```

## Key Improvements

### DDIM (Denoising Diffusion Implicit Models)
- Deterministic sampling with fewer steps (e.g., 50 instead of 1000)
- Same trained model, different sampling procedure

### Classifier-Free Guidance
- Conditional generation without a separate classifier
- Interpolate between conditional and unconditional predictions:
  ```
  ε̃ = (1 + w) · εθ(xₜ, t, c) - w · εθ(xₜ, t, ∅)
  ```
  where w is the guidance scale and c is the condition

### Latent Diffusion Models (LDM)
- Run diffusion in the latent space of a pre-trained autoencoder
- Much more computationally efficient for high-resolution images
- Basis of Stable Diffusion

## Architecture: U-Net for Diffusion
- Modified U-Net with residual blocks, self-attention layers, and time embedding
- Time step t is encoded via sinusoidal positional embedding and injected into each residual block
- Cross-attention layers for conditioning (text, class labels, etc.)

## Relevance to Neuroimaging
- MRI synthesis and data augmentation
- Brain anomaly detection (learn healthy distribution, detect deviations)
- Super-resolution of low-quality scans
- Harmonization across different scanners/sites

## 學習筆記

- forward process 的 closed-form 推導其實不難，就是把多步 Gaussian 合成一步，key insight 是 Gaussian 的性質
- variance schedule 的選擇蠻重要的，cosine schedule 比 linear schedule 通常效果更好 (Nichol & Dhariwal, 2021)
- DDIM 可以用同一個 trained model 做 deterministic sampling，這個設計真的很聰明
- 一開始搞不清楚 score-based model 跟 DDPM 的關係，後來看了 Song et al. (2021) 的 SDE 統一框架才比較懂
- TODO: 用 MNIST 實作一個簡單的 DDPM 練習

# VAE and GAN for Medical Imaging

## Variational Autoencoder (VAE)

> Kingma, D. P., & Welling, M. (2014). Auto-Encoding Variational Bayes. *ICLR 2014*.

### Core Idea
VAEs learn a latent representation of data by training an encoder-decoder pair with a probabilistic framework. The encoder maps input to a distribution in latent space, and the decoder generates data from latent samples.

### Architecture
```
Input x → Encoder → μ, σ² → z = μ + σ·ε (reparameterization trick) → Decoder → x̂
```

### Loss Function
```
L_VAE = E_q(z|x)[log p(x|z)] - KL(q(z|x) || p(z))
      = Reconstruction Loss  + KL Divergence
```
- **Reconstruction loss**: Measures how well the decoder reconstructs the input (MSE or BCE).
- **KL divergence**: Regularizes the latent space to be close to a prior N(0, I).

### Variants Relevant to Neuroimaging
- **β-VAE**: Adds a weight β to the KL term to control disentanglement of latent factors.
- **Conditional VAE (CVAE)**: Conditions generation on auxiliary information (e.g., diagnosis, age).
- **VQ-VAE**: Uses vector quantization for discrete latent codes, often used as the first stage of latent diffusion models.

### Applications
- Learning latent representations of brain structure for disease classification
- Detecting anomalies (high reconstruction error = abnormal)
- Disentangling age, sex, and disease effects in latent space

---

## Generative Adversarial Network (GAN)

> Goodfellow, I. J., et al. (2014). Generative Adversarial Nets. *NeurIPS 2014*.

### Core Idea
GANs consist of two networks trained adversarially: a **Generator** (G) that creates synthetic data, and a **Discriminator** (D) that distinguishes real from generated data. Through this competition, G learns to produce increasingly realistic samples.

### Training Objective
```
min_G max_D  E_x[log D(x)] + E_z[log(1 - D(G(z)))]
```

### Key GAN Variants for Medical Imaging

**Pix2Pix (Conditional GAN)**
- Paired image-to-image translation
- Use case: T1 → T2 MRI synthesis, MRI → CT synthesis

**CycleGAN**
- Unpaired image-to-image translation using cycle consistency loss
- Use case: Cross-modality synthesis when paired data is unavailable

**Progressive GAN / StyleGAN**
- Generate high-resolution images by progressively growing the network
- Use case: Generating realistic high-resolution brain MRI

**WGAN-GP (Wasserstein GAN with Gradient Penalty)**
- More stable training using Wasserstein distance
- Addresses mode collapse and training instability

### Applications in Neuroimaging
- **Data augmentation**: Generate synthetic brain images to augment small datasets
- **Cross-modality synthesis**: Generate T2 from T1, or PET from MRI
- **Super-resolution**: Enhance low-resolution MRI scans
- **Domain adaptation**: Harmonize images across different scanners

---

## VAE vs. GAN Comparison

| Aspect | VAE | GAN |
|--------|-----|-----|
| Training | Stable (likelihood-based) | Unstable (adversarial) |
| Sample quality | Blurry | Sharp |
| Latent space | Structured, interpretable | Less structured |
| Mode coverage | Good (covers full distribution) | May miss modes (mode collapse) |
| Anomaly detection | Natural (via reconstruction error) | Requires additional setup |
| Theory | Principled (variational inference) | Game theory |

## Current Trend
Modern approaches increasingly combine ideas from VAEs, GANs, and diffusion models. For example, Latent Diffusion Models use a VQ-VAE encoder to map images to a latent space, then apply diffusion in that compressed space for efficient high-quality generation.

## 一些筆記

- VAE 的 reparameterization trick 是整篇最精華的部分，讓 sampling 變成可微分的
- GAN training 真的很不穩定，自己試過幾次都遇到 mode collapse，WGAN-GP 確實有改善
- CycleGAN 的 unpaired translation 概念很厲害，但生成品質有時候不如 paired method
- 目前的趨勢是 diffusion-based 方法漸漸取代 GAN，因為 training 更穩定、sample diversity 更好

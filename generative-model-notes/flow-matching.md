# Flow Matching and Continuous Normalizing Flows

> Lipman, Y., Chen, R. T. Q., Ben-Hamu, H., & Nickel, M. (2023). Flow Matching for Generative Modeling. *ICLR 2023*.

## Core Idea

Flow Matching provides a simulation-free framework for training Continuous Normalizing Flows (CNFs). Instead of solving ODEs during training (as in the original CNF formulation), Flow Matching directly regresses a vector field that transports a simple prior distribution (e.g., Gaussian) to the data distribution.

## Background: Continuous Normalizing Flows

A CNF defines a time-dependent vector field vθ(x, t) that generates a flow φt through the ODE:
```
dφt(x)/dt = vθ(φt(x), t),  φ₀(x) = x
```

The flow transforms a source distribution p₀ (e.g., N(0, I)) into a target distribution p₁ (data) by continuously deforming the space from t=0 to t=1.

## Flow Matching Objective

Instead of maximizing likelihood (which requires expensive ODE solves), Flow Matching minimizes:
```
L_FM = E_{t, x} [ ||vθ(x, t) - uₜ(x)||² ]
```
where uₜ(x) is a target vector field that generates the desired probability path.

### Conditional Flow Matching (CFM)
In practice, we use the conditional version:
```
L_CFM = E_{t, x₁, xₜ} [ ||vθ(xₜ, t) - uₜ(xₜ | x₁)||² ]
```
where:
- x₁ ~ q(x₁) is a data sample
- xₜ = (1-t)x₀ + tx₁ is a linear interpolation (with x₀ ~ N(0, I))
- uₜ(xₜ | x₁) = x₁ - x₀ is the conditional vector field (for linear paths)

### Optimal Transport (OT) Paths
- **Linear (default)**: xₜ = (1-t)x₀ + tx₁, straight-line paths
- **OT-CFM**: Pairs noise and data samples via mini-batch optimal transport for straighter paths and faster convergence

## Training Algorithm

```python
def flow_matching_loss(model, x1):
    t = torch.rand(x1.shape[0], 1)         # Sample time
    x0 = torch.randn_like(x1)               # Sample noise
    xt = (1 - t) * x0 + t * x1              # Interpolate
    target = x1 - x0                         # Target vector field
    pred = model(xt, t)                      # Predicted vector field
    return ((pred - target) ** 2).mean()
```

## Sampling (Inference)

Solve the ODE from t=0 to t=1:
```python
from torchdiffeq import odeint

def sample(model, shape, n_steps=100):
    x = torch.randn(shape)  # Start from noise
    dt = 1.0 / n_steps
    for i in range(n_steps):
        t = torch.full((shape[0], 1), i * dt)
        x = x + model(x, t) * dt  # Euler step
    return x
```

For better accuracy, use adaptive ODE solvers (e.g., `dopri5`).

## Comparison with Diffusion Models

| Aspect | DDPM | Flow Matching |
|--------|------|---------------|
| Training | Noise prediction | Vector field regression |
| Forward process | Stochastic (add noise) | Deterministic (interpolation) |
| Sampling | Iterative denoising | ODE solving |
| Path | Curved (variance-preserving) | Straight (linear interpolation) |
| Steps needed | ~50–1000 | ~10–100 (straighter paths) |
| Theory | Score matching / SDE | Optimal transport / ODE |

## Advantages
- Simpler training objective (no noise schedule to tune)
- Straighter transport paths → fewer sampling steps
- Naturally connects to optimal transport theory
- Compatible with any coupling between source and target distributions

## Applications in Neuroimaging
- **MRI normalization**: Learn a flow from individual brain images to a reference template
- **Image-to-image translation**: e.g., T1 → T2 synthesis via conditional flow matching
- **Data augmentation**: Generate realistic brain images from noise
- **Longitudinal modeling**: Model brain changes over time as continuous flows

## 學習心得

- 跟 DDPM 比起來，flow matching 的 training objective 簡單很多，不用管 noise schedule
- OT-CFM 的 mini-batch optimal transport pairing 讓 path 更直，sampling 速度更快
- 這篇的數學符號跟 diffusion model 的文獻不太一樣，一開始看有點混亂，慢慢習慣了
- 重點是理解 vector field regression 和 score matching 本質上在做類似的事情，只是 formulation 不同
- 後來看到 Dao et al. 把 FM 搬到 latent space 的那篇 (LFM)，覺得是很自然的 extension

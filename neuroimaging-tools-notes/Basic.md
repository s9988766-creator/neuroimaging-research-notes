# MRI 基本概念：T1 與 T2

- **T1**：加強脂肪組織的訊號，並抑制水的訊號，使得液體在圖像中呈現**較暗**。
- **T2**：加強水的訊號，使得液體在圖像中呈現**較亮**，有助於顯示病理性變化。

> T1 重點在於**解剖結構**，圖像較為清晰；T2 重點在於**病理變化**，液體顯示較亮。

*來源：[MRI 中的 T1 和 T2 是什麼意思？它們有什麼不同？](https://www.shuzhiren.com/post/43770.html)*

---

# Normalize

## 自由度

表示：**允許影像如何移動的獨立參數**總共有幾個。

- 在二維空間，物體可以前後移動，所以自由度是 2
- 在三維空間，物體可以沿 x, y, z 軸平移，還可以沿著 x, y, z 軸旋轉，所以自由度是 3 + 3 = 6

---

## 空間變換（變形場）

### 1. 剛體變化（Rigid Body Transformation）

圖形移動到模板的位置只需要**平移**或**旋轉**。

> 自由度：沿 x, y, z 軸平移 (3) + 沿 x, y, z 軸旋轉 (3) = **6**

### 2. 仿射變化（Affine Transformation）

圖形移動到模板需要平移或旋轉 + **拉伸**（放大、縮小、傾斜）。

> 自由度：以矩陣來看，仿射變換可以寫成 `x' = Ax + t`
>
> 其中 A 是仿射矩陣（3×3，9 個參數），t 是平移向量（3 個參數）
>
> A 有 9 個可調控的參數 (自由度 9) + 沿 x, y, z 軸平移 (3) = **12**

### 3. 非線性變換（Nonlinear）

圖形需要**局部變換**。

- 例如：二維的圖像變形場，箭頭代表要往哪個方向移動多少毫米

<!-- TODO: 插入圖片 — 變形場示意圖（Deformation field 的 x-component 與 y-component）-->
![變形場示意圖](deformation_field.png)

---

### 配準比較圖

<!-- TODO: 插入圖片 — 腦部配準前後對比（紅色輪廓線覆蓋在腦部切面上）-->
![配準比較圖 - 切面 1](registration_compare_1.png)

![配準比較圖 - 切面 2](registration_compare_2.png)

---

### Q：如果遇到一個影像離變形場非常遙遠（位置十分奇怪、或病人病灶十分明顯以至於幾乎大部分改變了 MRI 影像的樣子），請問變形場要如何將該影像變形？

**A：**

- **a. 不參與配準（Lesion Masking）**
  在計算配準誤差時，把病灶區域當作不存在，不讓它影響變形場的估計。有一個 lesion mask，其中 1 對應正常腦、0 對應病灶腦，所以病灶處經過 mask 之後 voxel = 0，會被當作空的東西。

- **b. DARTEL 模板**
  用病人群體本身來建立平均型態當作模板。

- **c. 不追求完美對齊**
  不要求 voxel-to-voxel 一一對應，只要結構上合理即可：
  - **i. Limited Warp + Voxel-wise**：只用 affine / weak non-linear，接受對齊不完美，仍做 voxel-wise statistics
  - **ii. ROI-based Analysis**：不在乎 voxel 對 voxel，只看 ROI GMV
  - **iii. Network-level / Summary Metrics**：不追求完美對齊，如 global thickness、network strength

---

## 配準公式

$$E = E_{\text{similarity}} + \lambda \cdot E_{\text{smoothness}}$$

E 的數值越小，代表變形場越好。

- **影像配準（Registration）**：`E_similarity` — 現在這個 subject 的位置跟 template 的位置一不一樣
- **變形合不合理**：`E_smoothness` — 變形場要平滑、連續，不要突然亂彎移（不要局部拉伸、不連續變形、拓撲破壞）

$$E_{\text{smoothness}} = \int |\nabla v(x)|^2 \, dx$$

如果相鄰的 voxel 位移方向或大小差太多，則 |∇v(x)|（速度場變化量）會變得很大，所以 `E_smoothness` 就會變得很大。

**平衡權重 λ：**

| λ 大 | λ 小 |
|------|------|
| 保守 | 激進 |
| 變形小 | 變形大 |
| 對齊不完美 | 容易 overfit |

---

## 為什麼不能直接強行拉？

因為它還受三個硬約束：

### (1) Diffeomorphic（拓撲守恆）

- 不允許：一點對多點、空間折疊
- 病灶區如果硬拉，會產生 folding

### (2) Smoothness（平滑性）

- 相鄰 voxel 的位移不能劇烈不同
- 病灶通常是邊界陡峭、局部改變巨大
- 硬對齊會破壞整個變形場

所以在運算時，先找到想往哪裡走（算 similarity gradient），再經過平滑 kernel：

$$v_{\text{new}} = \text{smooth}(v_{\text{old}} - \eta \nabla E_{\text{similarity}})$$

### (3) 全局最小能量原則

根據公式 `E = E_similarity + λ·E_smoothness`，在病灶區：

- `E_similarity` 怎麼移都降不下來
- 繼續嘗試會讓 `E_smoothness` 爆炸

> 最小能量解 = **放棄該區的 similarity**

---

## 這樣的影像經過 Warp 之後會長怎樣？

變形場經過正常腦區的參與得出（計算 similarity → 決定變形方向），而這個變形場被套用到整張影像，包括病灶區。因此病灶區的 voxel 會依照鄰近的正常腦延伸出來的位移向量移動。

> 病灶區的位移由周圍正常組織的**平滑延拓（interpolation）**決定
>
> ⇒ 病灶區的形狀、位置被平移或拉伸，**無法反映正確的病灶資訊**

---

# 變形場的生成機制

先求一個速度場 `v(x)`，再透過對速度場做時間的積分而得到變形場：

$$\frac{d\phi(x,t)}{dt} = v(\phi(x,t)), \quad \phi(x,0) = x$$

- `v(x)`：速度場
- `ϕ(x,t)`：隨著時間變換的變形場

算從 t=0 到 t=1 所得到的所有變形：

$$\int_0^1 \phi(x,t) \, dt = \exp(v(x))$$

又 `v(x)` 是連續可微的，所以 `ϕ(x,t)` 存在唯一解且對初始條件連續相依。

---

# 定義 Diffeomorphic（微分同胚）

## 達成微分同胚的兩個前提

1. 該映射**可逆**（invertible）
2. 該映射與逆映射**可微**（differentiable）

## 定義可逆 & 平滑

### a. 直覺理解——彈力繩比喻

- **可逆的**：它可以被拉伸或扭曲成不同的形狀，而且過程中不會被撕破或黏在一起，之後可以自己收縮回去（縮回去的過程也不會重疊）
- **平滑的**：在拉伸扭曲或縮短的過程中是沒有斷裂或尖角的

### b. 數學定義（線性代數）

我們有兩個 D 維度的空間 R^D，有一個變換 T : R^D → R^D：

- **可逆性**：T 有一個反矩陣 T⁻¹，使得 T · T⁻¹ = I
- **平滑性**：T 與 T⁻¹ 都是可微的

---

## DARTEL：能保留拓撲結構的變化

假設一個恆定的速度場 A。對一個 3D 圖像 `[pixel, pixel, pixel]` 來說，這個速度場 A 是一個 3D 向量場 `[v(x), v(y), v(z)]`，兩者的長寬高一樣。

- 對 **3D 圖像**來說：每個體素代表該位置的訊號強度
- 對 **速度場 A** 來說：每個體素都是一個向量，描述該位置在當前時刻配準過程中移動的方向（恆定速度場）

把配準過程化簡成一個簡單的常微分方程（ODE）：

$$\frac{d\phi}{dt} = A \times \phi$$

- `ϕ`：隨時間變換的變形場
- `A`：速度場

**求解過程：**

$$\frac{1}{\phi} \, d\phi = A \, dt$$

對兩側求積分：

$$\ln \phi = At + c$$

$$\Rightarrow \phi = e^{At+c} = e^c \cdot e^{At} = \phi_0 \cdot e^{At}$$

其中 `ϕ₀` 是初始條件（恆等變換），即對任何向量滿足 ϕ₀(A) = A。

**驗證可逆性：**

$$\phi \cdot \phi^{-1} = e^{At} \cdot e^{-At} \cdot \phi_0 \cdot \phi_0 = I \cdot \phi_0^2 = \phi_0$$

> 這個微分方程已保證變形場 ϕ 是**可微**的，且上述推導證明它是**可逆**的。
>
> ⇒ 得證：這個變形場是一個**微分同胚的變形場**。

---

# 平滑延拓

在病灶區不存在「往哪裡移會更像模板」的方向：

$$\nabla E_{\text{similarity}}(x) \approx 0 \text{（或不穩定）}$$

這時候，變形場更新方程裡只剩下 `−λ·E_smoothness`。

$$E_{\text{smoothness}} = \int |\nabla v(x)|^2 \, dx$$

這代表：懲罰「位移變化太快」，鼓勵「相鄰位置的位移相近」。

位移更新公式：

$$v_{\text{new}} = \text{smooth}(v_{\text{old}} - \eta \nabla E_{\text{similarity}})$$

### 正常腦區

- ∇E_similarity(x) ≠ 0
- 更新方向明確
- smoothing 只是稍微修飾
- ⇒ 位移是「**自己決定的**」

### 病灶區

- ∇E_similarity(x) ≈ 0
- 更新後只剩：`v_new = smooth(v_old − 0)`
- ⇒ 位移**完全來自鄰域**
- 這一步就叫**平滑延拓（smooth interpolation / extension）**

### 橡皮膜比喻

想像一張橡皮膜：

- **正常腦區**：有人拿手指在拉（similarity force）
- **病灶區**：沒有人拉，但橡皮膜本身有「不能折、不喜歡尖角」的特性（smoothness）

結果是：沒人拉的地方，會被周圍被拉的地方「帶著一起動」。

> 這個「被帶著動」⇒ 就是**平滑延拓**。

---

# SBM Segmentation

Segmentation 的結果**不是 0 或 1**，而是**機率分布**：

| 類別 | 名稱 | 說明 |
|------|------|------|
| **c1** | Gray Matter (GM) probability map | `c1(x) ∈ [0,1]`：該 voxel 屬於灰質的機率 |
| **c2** | White Matter (WM) probability map | `c2(x) ∈ [0,1]`：該 voxel 屬於白質的機率 |
| **c3** | CSF (腦脊髓液) | `c3(x) ∈ [0,1]`：該 voxel 屬於 CSF 的機率 |

> 經過 c1 segment 的影像會比原本 T1 warped 過的影像**更乾淨**，因為它只保留了灰質的機率資訊，去除了其他組織的干擾。

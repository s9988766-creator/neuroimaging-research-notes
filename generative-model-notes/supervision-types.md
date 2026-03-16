# 監管方式 (Supervision Types in Machine Learning)

## Overview

機器學習的三大監管方式，差異在於 training data 有沒有 label (ground truth y)。

| 監管方式 | 訓練資料 | 典型問題 |
|---------|---------|---------|
| Supervised | (x, y) 都有 | Classification, Regression |
| Unsupervised | 只有 x，沒有 y | Clustering, Dimension Reduction |
| Weak Supervised | y 不完整、不精確、或粗粒度 | 介於兩者之間 |

---

## Supervised Learning（監督式學習）

Training sample = feature vector (x) + ground-truth label (y)

根據 y 的類型分成兩種問題：

**Classification（分類）**：y 是離散的類別標籤
- 例：給一顆西瓜的特徵 → 判斷 Good or Bad
- 常用演算法：Decision Tree, Naive Bayes, Logistic Regression, Random Forest, SVM, Neural Network, Gradient Boosting Tree

**Regression（迴歸）**：y 是連續的數值
- 例：給一顆西瓜的特徵 → 預測價格 3000円? 4000円?
- 常用演算法：Linear Regression, Decision Tree, Random Forest, Neural Network, Gradient Boosting Tree

---

## Unsupervised Learning（非監督式學習）

Training sample = 只有 feature vector (x)，完全沒有 label y

模型自己從資料中找出結構和 pattern。

**Clustering（分群）**：把相似的資料分在一起
- 例：一堆西瓜，不知道好壞，讓模型自己分群
- 常用演算法：K-means

**Association（關聯）**：找出資料之間的關聯規則
- 常用演算法：Apriori

**Dimension Reduction（降維）**：把高維資料壓縮到低維
- 常用演算法：PCA

---

## Weak Supervised Learning（弱監督學習）

介於 supervised 跟 unsupervised 之間，有 label 但 label 的品質不完美。沒有完整的 ground truth y。

> 參考：[浅谈弱监督学习 - 知乎](https://zhuanlan.zhihu.com/p/81404885)

### 三種類型

**1. Incomplete Supervision（不完全監督）**

訓練資料中只有一部分有標籤，其他資料沒有標籤。

例：100 張西瓜圖片，只有 20 張標了 watermelon / not watermelon，其餘 80 張沒標。

解決方法：
- **Active Learning（主動學習）**：模型主動挑選最有價值的 unlabeled data，請人類標註，用最少的標註量達到最好效果
- **Semi-supervised Learning（半監督學習）**：同時利用少量 labeled data 和大量 unlabeled data 來訓練

相關論文：
- *Large-scale interactive object segmentation with human annotators* (2019, CVPR) — 結合人類標註者和互動式分割的大規模方法

**2. Inexact Supervision（不確切監督）**

訓練資料只有粗粒度的標籤，沒有細粒度的標籤。

例：知道一張圖裡「有西瓜」，但不知道西瓜在圖片的哪個位置（只有 image-level label，沒有 pixel-level label）。可以想像成一個「包」(bag)，裡面有多個示例 (instances)，只知道整個包的標籤 Y/N，不知道每個示例的標籤。

解決方法：
- **Multi-instance Learning（多實例學習）**：把每個訓練樣本視為一個 bag，bag 內有多個 instances。如果 bag 是 positive，至少有一個 instance 是 positive；如果 bag 是 negative，所有 instances 都是 negative。

**3. Inaccurate Supervision（不精確監督）**

給出的標籤不總是正確的，存在 noise。

例：本來是西瓜卻被標成 pineapple，或本來應該是 Y 的被錯誤標記成 N。

解決方法：
- Noise-robust learning methods
- Label cleaning / correction
- Confidence learning (e.g., cleanlab)

---

## 各種監管方式的比較

```
            有完整 label?
                |
        ┌───── Yes ─────┐
        |                |
   Supervised        完全沒有?
   (完整 y)              |
                  ┌── Yes ──┐
                  |          |
             Unsupervised   部分/不完美?
             (沒有 y)           |
                            Weak Supervised
                           ┌────┼────┐
                     Incomplete  Inexact  Inaccurate
                     (部分沒標)  (粗粒度)  (標錯)
```

## 跟 Neuroimaging 的關係

- 腦影像標註非常貴（需要專業放射科醫師），所以弱監督和半監督方法在 neuroimaging 特別重要
- Brain lesion segmentation 常常只有 image-level 診斷（有/沒有病變），沒有 pixel-level 標註 → 典型的 inexact supervision
- 不同標註者之間的 inter-rater variability 造成 label noise → inaccurate supervision
- Active learning 可以用來優化標註流程，讓醫師只標最有價值的 case

## 我的筆記

- 之前一直搞混 semi-supervised 跟 weak supervised 的差別，現在比較清楚了：semi-supervised 是 weak supervised 底下 incomplete supervision 的一種解法
- Multi-instance learning 的概念蠻有趣的，在 pathology (病理學) 的 whole slide image 分析裡面很常用
- TODO: 看看 neuroimaging 領域有沒有用 active learning 來減少標註成本的 paper
- TODO: 了解 cleanlab 這個工具，看能不能用在腦影像的 noisy label 問題上

# 方法 / Methods（含 ConvLEM 數學推導）

## 4.1 波場預測網路架構 / Wavefield Forecasting Network

## 原文 (English)

> WaveCastNet is composed of four main components:
> - **Embedding layer**: Maps input wavefields into a latent space. Two types: (i) convolutional layers with batch normalization and LeakyReLU for dense inputs; (ii) fully connected + convolutional layers for sparse inputs, trained with Masked Autoencoder (MAE) strategy (80% stations randomly masked).
> - **Encoder**: Processes the embedded sequence into a fixed-size encoder state.
> - **Decoder**: Predicts each element of the target sequence one at a time, using previous output combined with encoder state.
> - **Reconstruction layer**: Recovers detailed spatial information using transposed convolutional layers and pixel-shuffle techniques.
>
> The Seq2Seq framework seeks to find:
> Ỹ = argmax p(Y|X) ≈ D_decoder(E_encoder(X))
>
> WaveCastNet minimizes the Huber loss during training:
> L_Huber = Σ L_δ(X̂[c,h,w], X[c,h,w]) / (TCHW)
>
> where L_δ(x̂, x) = ½(x̂-x)² for |x̂-x| ≤ δ, else δ(|x̂-x| - ½δ)

## 中文翻譯與解說

### 架構四大組件

**1. 嵌入層（Embedding Layer）**
- 稠密輸入：卷積層 + 批次正規化 + LeakyReLU
- 稀疏輸入：全連接層 → 卷積層，採用 **MAE（遮罩自編碼器）策略**
  - 80% 台站隨機遮罩，模型學習從剩餘 20% 重建完整表徵
  - 同一 batch 內所有時間步使用相同遮罩，確保時序連續性

**2. 編碼器（Encoder）**：處理嵌入後的時序，壓縮成固定大小的潛在狀態

**3. 解碼器（Decoder）**：以潛在狀態為條件，逐步自回歸生成未來波場序列

**4. 重建層（Reconstruction Layer）**：反卷積 + 像素重排（pixel-shuffle），從潛在空間恢復高解析度波場

### 損失函數：Huber Loss

選用 Huber Loss 而非純 L2 Loss 的原因：
- **平衡 L1 與 L2 的優點**：小誤差用 L2（平滑），大誤差用 L1（抗離群值）
- 特別適合地震多種情境下的 PGV 模式學習
- 改善高震級事件的泛化能力，收斂更快

---

## 4.2 ConvLEM 卷積長表達記憶 / Convolutional Long Expressive Memory

## 原文 (English)

### Background: Dynamical Systems View

> Traditional recurrent units viewed as dynamical systems:
> dh/dt = τ · f_θ(h(t), x(t))
> Limited to a fixed temporal scale τ.

### LEM Foundation

> The Long Expressive Memory (LEM) unit is based on coupled ODEs:
> dc(t)/dt = g_c ⊙ [f^c_θc(h(t), x(t)) − c(t)]
> dh(t)/dt = g_h ⊙ [f^h_θh(c(t), x(t)) − h(t)]
>
> where h(t) ∈ R^l and c(t) ∈ R^l are slow- and fast-evolving hidden states, and g_c, g_h are gating functions introducing variability in temporal scales.

### ConvLEM Formulation

> Extending LEM with convolutional operations:
> dC(t)/dt = g_c ⊙ [f^c_θc(H(t), X(t)) − C(t)]
> dH(t)/dt = g_h ⊙ [f^h_θh(C(t), X(t)) − H(t)]
>
> where H(t) ∈ R^{r×p×q} and C(t) ∈ R^{r×p×q} are the slow and fast hidden states as spatial tensors.

### IMEX Discretization (Equations 8–10)

> Δt^n_c = Δt · g_c
> Δt^n_h = Δt · g_h
> C_n = (1 − Δt^n_c) ⊙ C_{n-1} + Δt^n_c ⊙ f^c_θc
> H_n = (1 − Δt^n_h) ⊙ H_{n-1} + Δt^n_h ⊙ f^h_θh
>
> Update functions:
> f^c_θc = tanh(W_hc * H_{n-1} + W_xc * X_n)
> f^h_θh = tanh(W_ch * C_n + W_xh * X_n)
>
> Gating functions:
> g_h = σ(W_xt * X_n + W_ht * H_{n-1})
> g_c = σ(W_xt * X_n + W_ht * H_{n-1})

### Reset Gate (Equation 11–12)

> g_reset = σ(W_xr * X_n + W_hr * H_{n-1})
> f^h_θh = tanh(g_reset ⊙ (W_ch * C_n) + W_xh * X_n)

### Peephole Connections (Equation 13)

> g_h = σ(W_xt * X_n + W_ht * H_{n-1} + W_ct ⊙ C_{n-1})
> g_c = σ(W_xt * X_n + W_ht * H_{n-1} + W_ct ⊙ C_n)
> g_reset = σ(W_xr * X_n + W_hr * H_{n-1} + W_cr ⊙ C_n)

## 中文翻譯與數學解說

### 為何需要 ConvLEM？

| 架構 | 問題 |
|------|------|
| 標準 RNN/LSTM/GRU | 全連接層，無法保留二維空間信息 |
| ConvLSTM | 加入卷積，能捕捉空間多尺度，但時間尺度固定 |
| **ConvLEM** | **同時捕捉空間多尺度 + 時間多尺度** |

### 核心思路：雙時間尺度隱藏狀態

ConvLEM 維護**兩個**隱藏狀態張量：

- **C(t)（快速隱藏狀態）**：追蹤短期、快速變化的動態（如波前通過）
- **H(t)（慢速隱藏狀態）**：追蹤長期、慢速變化的動態（如背景地動趨勢）

兩者透過門控函數 g_c、g_h 動態調節更新速率，實現多時間尺度建模。

### 離散化方案：IMEX（隱式-顯式）

連續微分方程需要離散化才能用反向傳播訓練。IMEX 方案：

```
# 時間步長根據門控值自適應調整
Δt_c = Δt × g_c  （快速狀態的有效步長）
Δt_h = Δt × g_h  （慢速狀態的有效步長）

# 殘差更新（類似 ResNet 的跳連）
C_n = (1 - Δt_c) × C_{n-1} + Δt_c × f_c(H_{n-1}, X_n)
H_n = (1 - Δt_h) × H_{n-1} + Δt_h × f_h(C_n, X_n)
```

直觀理解：g 越大 → 更新越快（更關注當前輸入）；g 越小 → 保留更多歷史記憶。

### 卷積操作的作用

所有矩陣乘法改為**卷積（∗）**，使得：
- 保留空間局部相關性（鄰近格點的物理耦合）
- 共享卷積核（大幅減少參數量）
- 能處理任意解析度的空間輸入

### 重置門（Reset Gate）

重置門 g_reset 控制快速狀態 C 對慢速狀態 H 更新的影響程度：
```
g_reset = σ(W_xr * X_n + W_hr * H_{n-1})
f_h = tanh(g_reset ⊙ (W_ch * C_n) + W_xh * X_n)
```

作用：在不相關的快速波動不應影響長期記憶時，門控值趨近 0 以隔離影響。

### 窺孔連接（Peephole Connections）

進一步將快速隱藏狀態 C 的信息注入到門控函數中：
```
g_h = σ(W_xt * X_n + W_ht * H_{n-1} + W_ct ⊙ C_{n-1})
```

讓門控機制「看到」快速狀態的當前值，提升對複雜多尺度動力學的建模能力。

---

## 4.3 正規化策略 / Normalization

## 原文 (English)

> Particle velocity-wise normalization for each snapshot:
> X̄_t = (X_t[c,h,w] − X_mean[c,h,w]) / X_std[c,h,w]
>
> For domain-shifted settings (higher magnitudes), additional channel-wise normalization using the standard deviation from the initial input window (t₁ to t₂) is applied to ensure a reasonable input range.

## 中文翻譯與解說

### 粒子速度逐格點正規化

對訓練集中每個空間格點 [c,h,w]，計算所有時刻、所有樣本的均值和標準差，然後正規化：

```
X̄_t[c,h,w] = (X_t[c,h,w] - X_mean[c,h,w]) / X_std[c,h,w]
```

**優點**：避免不同地理位置地動強度差異過大影響訓練，同時**防止稀疏輸入場景下的空間信息洩漏**（每個台站獨立正規化）

### 域偏移正規化（高震級場景）

M4.5–M7 的地動振幅差可達 80 倍以上，需額外以**輸入窗口的標準差**做通道正規化，確保輸入值在合理範圍內。

---

## 4.4 資料生成 / Data Generation

## 原文 (English)

> Simulations use the open-source SW4 package (fourth-order finite-difference viscoelastic model), with the USGS San Francisco Bay Region 3D seismic velocity model v21.1. Grid size: 150 m³ on surface (doubled at 2.2 km and 6.6 km depth). Simulation runs for 120 seconds at Δt = 0.0260134 s (4,613 time steps). Three-component particle velocity recorded every 10 steps (0.26014 s) on 150 m × 150 m grids, downsampled to 300 m × 300 m for training.

## 中文翻譯與解說

- 軟體：SW4（開源，四階有限差分黏彈性波方程求解器）
- 速度模型：USGS 舊金山灣區三維速度模型 v21.1
- 最小 S 波速度：500 m/s
- 網格大小：地表 150 m³，深度 2.2 km 和 6.6 km 後各加倍至 600 m³
- 總格點數：約 9.59 百萬
- 模擬時長：120 秒，時間步 0.026 s
- 記錄間隔：每 10 步 = 0.26 s
- 輸出解析度：降採樣至 300 m × 300 m 供訓練使用

---

## 4.5 延遲測試 / Latency Tests

## 原文 (English)

> Random time shifts sampled from N(0,1). Adjusted shift: sign(s) · min(⌈|s|⌉, 4) · Δt, with maximum time difference at 4Δt.

## 中文翻譯與解說

模擬真實 EEW 系統中資料傳輸不均的情況：從標準正態分佈抽取隨機時移，最大 ±4 個時間步（約 ±1 秒），測試模型在延遲不均情況下的穩健性。

---

## 4.6 真實資料預處理 / Real Data Preparation

## 原文 (English)

> 2018 Berkeley earthquake records from NCEDC, starting 20 s before origin time, 140 s total. Resampled to 0.01 s, instrument response removed, bandpass filtered 0.06–0.5 Hz, downsampled to 0.26 s intervals.

## 中文翻譯與解說

- 來源：北加州地震資料中心（NCEDC）
- 事件：2018 年 M4.4 Berkeley 地震（深度 12.3 km）
- 預處理：儀器響應移除 → 帶通濾波（0.06–0.5 Hz）→ 降採樣至 0.26 s
- 台站數：178 站（101 個 ShakeAlert + 額外 Berkeley 數位地震網絡及 USGS 台站）

---

## 評估指標 / Evaluation Metrics

$$\text{PGV}(\mathbf{X}) = \max_t \sqrt{X_t^2[c_X] + X_t^2[c_Y]}$$

$$T_{pgv}(\mathbf{X}) = \arg\max_t \sqrt{X_t^2[c_X] + X_t^2[c_Y]}$$

$$\text{ACC} = \frac{\sum_{t,h,w} \hat{X}_t[c,h,w] \cdot X_t[c,h,w]}{\sqrt{\sum \hat{X}_t^2[c,h,w] \cdot \sum X_t^2[c,h,w]}}$$

$$\text{RFNE} = \sqrt{\frac{\sum_{t,h,w} (\hat{X}_t[c,h,w] - X_t[c,h,w])^2}{\sum_{t,h,w} X_t^2[c,h,w]}}$$

- **ACC（精度）**：預測與真值的空間相關性，越接近 1 越好
- **RFNE（相對 Frobenius 範數誤差）**：相對誤差大小，越接近 0 越好

---

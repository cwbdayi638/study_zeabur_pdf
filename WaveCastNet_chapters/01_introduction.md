# 引言 / Introduction

## 原文 (English)

> Earthquakes generate complex seismic wavefields as energy is released from the rupture and propagates through the Earth's interior and surface, producing ground motions that can cause significant damage. Real-time prediction of how these ground motions evolve in space and time following the onset of rupture is critical for impact assessment and hazard mitigation, forming the foundation of earthquake early warning systems (EEW). Such forecasts provide immediate information that supports timely and potentially life-saving decision making.
>
> Traditional approaches often rely on estimating earthquake source parameters, such as location, magnitude, and fault geometry, and use empirical ground-motion models to predict shaking intensity. These methods are highly sensitive to errors in early parameter estimates, particularly magnitude, which can result in missed or false alerts by warning systems.
>
> We propose WaveCastNet, based on the sequence-to-sequence (Seq2Seq) framework. The core architecture consists of an encoder and a decoder: the encoder processes the input sequence of seismic wavefields and summarizes it into a single latent state, while the decoder generates the future sequence conditioned on this state.
>
> We design a convolutional long expressive memory (ConvLEM) model, an extension of the LEM architecture, that integrates convolutional layers into the LEM framework. ConvLEM effectively models the joint spatial and temporal correlations that govern the evolution of the wavefield.

## 中文翻譯與解說

### 問題背景

地震在破裂後釋放能量，透過地球內部與地表傳播，產生可能造成重大破壞的地面運動。即時預測這些地面運動如何在空間和時間上演變，對於衝擊評估和災害減緩至關重要，也是**地震預警系統（EEW）**的核心基礎。

### 傳統方法的挑戰

傳統方法通常需要先估算地震源參數（如位置、震級、斷層幾何），再使用**經驗性地動預測方程（GMPE）**來估算地動強度。

**主要問題：**
- 早期震源參數估算常有誤差，特別是震級估算，導致漏報或誤報
- GMPE 通常假設「平均性（ergodicity）」，忽略重要的區域性和路徑相關的波傳播差異
- 基於物理數值模擬的方法計算成本過高，難以即時部署

### WaveCastNet 解決方案

**架構概述：**

1. **問題形式化**：給定 J 個時刻的波場序列 X₁, X₂, ..., X_J，預測未來 K 個時刻 X_{J+1}, ..., X_{J+K}，其中每個 X_t ∈ R^{C×H×W}（C=3個速度分量，H×W 為空間格點）

2. **Seq2Seq 架構**：
   - **Encoder**：壓縮輸入波場序列為一個潛在狀態（latent state）
   - **Decoder**：以潛在狀態為條件，逐步生成未來波場序列

3. **ConvLEM**：設計一種結合卷積層的長表達記憶單元，同時捕捉空間多尺度模式和時間長程依賴

### WaveCastNet 主要優勢

| 優勢 | 說明 |
|------|------|
| 預測精度 | 優於 ConvLSTM、ConvGRU、Transformer 等基準模型 |
| 靈活性 | 支援密集波場輸入與稀疏台站觀測兩種模式 |
| 泛化能力 | 可推廣到訓練外的震級與震源位置，zero-shot 支援真實地震 |
| 抗噪性 | 背景雜訊和資料延遲情況下仍可維持性能 |
| 不確定性估計 | 支援集成模型（ensemble）量化預測不確定性 |
| 即時可用 | 100 秒預報僅需 0.56 秒（NVIDIA A100 GPU） |

---

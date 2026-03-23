# 第二章：方法與模型架構 / Methods & Model Architecture

## 架構概覽

FisH 模型由三個核心模組組成：
1. **Embedder（嵌入器）**：將即時串流波形轉換為波形嵌入向量（Wave Embeddings）
2. **Encoder（編碼器）**：利用 RetNet 提取嵌入間的相關性，生成預測嵌入（Prediction Embeddings）
3. **Decoder（解碼器）**：將預測嵌入解碼為三項任務的最終輸出

---

## 2.1 Embedder（嵌入器）

### 原文 (English)

> The Embedder is a critical module designed to convert various input data types into a compatible format for the Encoder. It takes real waveform data (Waveform) as inputs, and transforms them into the Embedding which can be easily processed by the Encoder.
>
> The Embedder module is built on an intuitive design that primarily utilizes local operators to aggregate waveform information, ensuring the data is centrelized and that temporal data is effectively distilled.
>
> The Embedder module is composed of series MultiScalerLayers (MSL) which converts the input Waveform into its WaveEmbedding.
> WaveEmbedder = MSL_n ∘ ··· MSL_2 ∘ MSL_1
>
> Each MSL module consists of a MultiScalerFeature (MSF), a non-linear layer (NonLi), and a normalization layer (Norm).
> MSL_i = Norm_i ∘ NonLi ∘ MSF_i
>
> The MSF layer is a multi-perceptual aggregator composed of N parallel CNN Layers and the absolute value of the entire input ABS(X), which is then compressed to a vector in dimension D through a linear transformation.
> MSF_i(X) = M @ [CNN_i^I(X), CNN_i^II(X), CNN_i^III(X), ABS(X)]^T

### 中文翻譯與解說

Embedder 是將波形資料轉換為 Encoder 可處理格式的關鍵模組。它接受原始波形資料作為輸入，將其轉換為高維嵌入向量。

**設計特點：**
- 主要使用**局部算子（local operators）**聚合波形資訊
- 確保資料中心化，有效提煉時序特徵
- 進階情況下還可接受時變變數（如站外監測資料）和非時變變數（如台站資訊和類型）

**架構：串聯多個 MultiScalerLayer（MSL）**

每個 MSL 包含：
- **MSF（MultiScalerFeature）**：多感知尺度聚合器，使用多個不同感受野的 CNN 並行抽取特徵
- **NonLi**：非線性激活層
- **Norm**：正規化層

**MSF 的設計精妙之處：**

```
MSF_i(X) = M @ [CNN_i^I(X), CNN_i^II(X), CNN_i^III(X), ABS(X)]^T
```

同時使用三條不同感受野的 CNN 分支，**再加上波形的絕對值 ABS(X)**，最後用線性變換壓縮到維度 D。

**三個設計細節值得注意：**
1. **EWM（指數加權移動）**：對非中心化資料（如 DiTing 資料集）自動進行在線波形分解，即時分離「背景趨勢」與「地震行為」
2. **反對稱卷積核（anti-symmetric kernel）**：消除常數波形（直流偏移），解決採樣點差異導致的相位反轉問題
3. **對稱性設計**：考慮兩個僅差一個採樣點的波形在極值處的不可察覺差異

**白話解說：**

Embedder 就像「波形翻譯官」——把地震儀輸出的原始電壓值序列，翻譯成神經網路懂的語言（高維向量）。它之所以用多個不同大小的 CNN 並行，是因為地震波形在不同時間尺度下呈現不同特徵：短時窗看細節（P 波到達的尖銳變化），長時窗看整體（波形包絡趨勢）。把這些多尺度特徵融合在一起，讓後續的 Encoder 有更豐富的資訊可用。

---

## 2.2 Encoder（編碼器）— RetNet 架構

### 原文 (English)

> The Encoder module is designed to allow for our FisH to leverage the nonlinear coupled information among the wave embeddings to generate representative prediction embeddings for the Decoder module. To this end, we apply the recently proposed RetNet self-regressive architecture. The RetNet architecture is characterized by its parallel training capabilities, sequential inference with low cost, and robust performance, which are suitable for the real-time streaming seismic data and the EEW task.
>
> The core of the RetNet architecture is the Retention mechanism, which introduces a state matrix s to map the current timestep input to output, and a specially designed matrix a to propagate the state across the temporal dimension. This mechanism allows for efficient parallel training as all timestep outputs can be calculated simultaneously through matrix operations, enhancing training efficiency. In addition, the Retention mechanism can transform into recurrent neural network (RNN) sequential inference, reducing the inference cost per step to just a few matrix-vector dot operations. Another key feature of the Retention mechanism is the introduction of an exponential decay association factor γ, allowing for the control of 'memory' duration.

### 中文翻譯與解說

Encoder 模組讓 FisH 能夠利用波形嵌入之間的**非線性耦合資訊**，為 Decoder 生成具代表性的預測嵌入。

**為什麼選 RetNet？**

RetNet（Retentive Network）的核心是**Retention 機制**，具有以下特性：
- **狀態矩陣 s**：將當前時間步的輸入映射到輸出
- **傳播矩陣 a**：在時間維度上傳播狀態
- **指數衰減因子 γ**：控制「記憶」的持續時間（越遠的過去，影響越弱）

**RetNet 的三種等價計算模式：**

| 模式 | 計算方式 | 用途 | 成本 |
|------|----------|------|------|
| Parallel（平行） | 矩陣運算，所有時間步同時計算 | 訓練 | 高效 |
| Recurrent（遞迴） | RNN 逐步更新，每步 O(1) | 推論（即時） | 極低 |
| Chunked（分塊） | 介於兩者之間 | 長序列推論 | 中等 |

**數學表示：**

平行模式：`O_i^c = λ^(i-j) δ_{j≤i} q_i^a k_a^j v_j^c`

遞迴模式：`O_i^c = q_i^a H_{c,a}^i`，狀態更新：`H_{c,a}^{n+1} = λ H_{c,a}^n + k_a^{n+1} v_{n+1}^c`

這裡 λ = γ 就是指數衰減因子，控制長程記憶的強度。

---

### RetNet 完整架構

```
Retnet = Block_N ∘ Block_{N-1} ∘ ··· ∘ Block_2 ∘ Block_1
Block_i = FFN_i ∘ Norm_i ∘ (MSR_i ∘ Norm_i + Id)
```

每個 Block 包含：
- **MSR（Multi-scale Retention，多尺度保留）**：核心，含旋轉位置嵌入（Rotary Position Embedding, RoPE）和 Self Retention
- **FFN（Feed-Forward Network）**：兩層線性層夾正規化，非線性用 GELU，正規化用 RMSNorm
- **Norm**：RMSNorm，前後各一次，保證數值穩定
- **Resnet Wrapper**：殘差連接

Self Retention 的計算：
```
O = Norm(G) * Retention(Q, K, V)
O = A_o O + b_o
```
與 Transformer 類似，投影到 Q（Query）、K（Key）、V（Value）、G（Gate）。

**白話解說：**

RetNet 就是「記憶可控的 Transformer」。普通 Transformer 在推論時需要記住「所有過去的詞（時間步）」，計算量隨序列長度的平方增長（O(n²)），對即時串流來說太貴了。

RetNet 引入了指數衰減因子 γ：越久遠的過去，其影響力自動衰減。這讓 RetNet 在推論時可以像 RNN 一樣，只維護一個「壓縮記憶狀態」，每一步只需要做幾個矩陣向量乘法，計算成本是 O(1)——不管波形串流多長，每一步的計算量都一樣！

對 EEW 來說，這意味著 FisH 可以**永遠運行、永不停止**，地震來了就回應，平靜時也繼續監聽，完全不需要「切割」波形成固定長度的片段再批次處理。

---

## 2.3 Decoder（解碼器）

### 原文 (English)

> Within the Decoder module of our FISH model, the task-specific downstream outputs, including phase picking results, earthquake magnitudes, and location, are generated. The Decoder consists of three downstream heads, each responsible for producing the corresponding task outputs.
>
> For the phase picking head, a memory bank with a temporal size of T is utilized. At each time step, the prediction embedding is stored at the last position of the memory bank, and the prediction embedding at the first position is removed. This memory bank retains the prediction embeddings for the last T time steps. To generate the phase picking results, several convolutional layers are employed to localize the P/S arrival positions within this memory bank. If no P/S arrival is detected within the last T time steps, the output is set to 1, indicating the absence of P/S arrivals.
>
> For the location and magnitude estimation heads, they employ multiple linear layers to map the current prediction embedding to seismic quantities such as distances and magnitudes, enabling accurate estimation of earthquake locations and magnitudes.

### 中文翻譯與解說

Decoder 包含三個**下游任務頭（downstream heads）**，分別負責：

**1. 相位拾取頭（Phase Picking Head）**
- 維護一個大小為 T 的**記憶庫（memory bank）**（預設 T = 30 秒）
- 每個時間步：將新的預測嵌入加入記憶庫末端，移除最舊的嵌入
- 使用**卷積層**在記憶庫中定位 P/S 波到達位置
- 若偵測不到 P/S 波到達，輸出設為 1（表示無地震波到達）

**2. 位置估計頭（Location Estimation Head）**
- 使用多個線性層，將當前預測嵌入映射到震源相對台站的位置座標 (x_q, y_q)

**3. 規模估計頭（Magnitude Estimation Head）**
- 使用多個線性層，將當前預測嵌入映射到地震規模值

**白話解說：**

相位拾取頭用了一個「滑動視窗記憶庫」：想像一個長度 30 秒的捲動窗口，每秒更新，用卷積層掃描其中的 P/S 波特徵。這個設計讓模型能夠「回顧最近的歷史」來確認相位到達，而不是只看當前那一點。

位置和規模頭就簡單多了——直接從 Encoder 生成的「預測嵌入」線性映射出來。這說明 Encoder 已經把地震的位置和規模資訊「壓縮」進嵌入向量裡了，Decoder 只需要「解壓縮」即可。

---

## 2.4 訓練範式 / Training Paradigm

### 原文 (English)

> The raw data feed to the model is the waveform. We will only allow the normalization method that has online version. For example, shift or amplifier for a constant value.
>
> The main training framework is called "Sea" mode. The "Sea" mode is the advance version of the Recurrent mode.
>
> In Recurrent mode, given a waveform sequence containing earthquakes, our task is to obtain whole the seismic quantity for each timestamp. In this paradigm, model play the role that pack the history information into the latest embedding, and then transcribes to the seismic target.
>
> The "Sea" mode is no different from the Recurrent mode except a subtle controlling for the "hidden increment" of Retnet between the "quake" and "non-quake" states.

### 中文翻譯與解說

**資料正規化原則：**
只允許有**線上版本**的正規化方法（如位移或固定值放大），不允許需要完整資料才能計算的方法（如全域標準化）。

**資料增強策略：**
以 STEAD 為例，原始資料包含長度 L=6000 的波形序列和 P 波到達位置。訓練時在 [P-a, P] 範圍內隨機選取起始點，提取 L=6000 的子序列，缺失部分補零或加入雜訊。

**兩種訓練模式：**

**Recurrent 模式（遞迴模式）：**
- 輸入：Waveform(L,3)——長度 L 的三分量波形
- 輸出：每個時間步的地震物理量序列，如 Mag_pred(L,1)
- 模型角色：將歷史資訊壓縮到最新嵌入，再轉錄為目標地震物理量

**Sea 模式（核心訓練框架）：**
- Sea 模式是 Recurrent 模式的進階版
- 核心差異：控制 RetNet 在「地震」與「非地震」狀態之間的 **hidden increment（隱藏增量）**

**Hidden Increment 的定義：**

RetNet RNN 模式的更新公式：`H_{c,a}^{n+1} = λ H_{c,a}^n + k_a^{n+1} v_{n+1}^c`

其中 `k_a^{n+1} v_{n+1}^c` 就是每個時間步的 hidden increment。Sea 模式設定：

- **地震期間**：`Δ_{n+1} = ||k_a^{n+1} v_{n+1}^c|| ≠ 0`（允許更新）
- **雜訊期間**：`Δ_{n+1} = ||k_a^{n+1} v_{n+1}^c|| = 0`（禁止更新）

**Sea 模式帶來的三大特性：**
1. 模型可以從**任意雜訊時刻**開始訓練或推論，不受歷史狀態影響
2. 只要地震間隔大於記憶設定，在單一地震序列上訓練的模型能**自然應對多地震序列**
3. 模型的有效推論長度為**無限長**——真正的「immortal running（永生運行）」

**白話解說：**

Sea 模式是 FisH 最精妙的設計之一。普通的序列模型如果連續輸入長時間的雜訊，內部的記憶狀態會越來越「髒」，干擾後續的地震偵測。

Sea 模式的解法：**在非地震時段，強制讓 hidden increment 為零**，也就是「不讓雜訊污染記憶」。地震來了，才允許更新。這就像保全人員上班時打開監控記錄，下班後關閉——確保記憶只保存有用的地震資訊，雜訊時段的「空白時間」不會累積干擾。

這讓 FisH 能像一個**永不關機的感測器**：不管你什麼時候打開它，它都能立即正常工作；不管中間有多久沒有地震，它都不會被雜訊「毒化」。

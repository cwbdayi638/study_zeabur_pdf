# 第三章：實驗結果 / Results

## 資料集說明

本研究在以下資料集上驗證 FisH 模型：

| 資料集 | 說明 | 規模 |
|--------|------|------|
| **STEAD-Full** | 全球 AI 地震訊號資料集（Stanford Earthquake Dataset） | 1.2M |
| **STEAD-BDLEELSSO** | STEAD 的子集：震中距 <110 km、SNR ≥ 25 dB、NS/EW 分量方向正確 | 0.2M |
| **INSTANCE** | 義大利地震機器學習資料集 | 0.8M |

實驗結果主要以 STEAD-BDLEELSSO 作為分析範例。

---

## 3.1 相位拾取 / Phase Picking

### 原文 (English)

> To generate online phase picking results, for each timestep, we use the historical information of last T timesteps in the memory bank to generate the current picking results and continuously doing so as the time goes. Generally, we set T = 30s. Our FisH model demonstrates remarkable performance in promptly picking P/S arrival times upon the arrival of the corresponding waves. At the P/S arrival times, it achieves a precision and recall of 99% for P arrival picking. As for S arrival picking, our model achieves an accuracy rate of 96% and a recall rate of 90%.
>
> We also follow the evaluation setting in EQTransformer to evaluate the offline picking performance for our proposed model, where the complete seismic wave inputs are given and the information after P/S arrivals is used. Specifically, we use the first 30 seconds of each seismic wave sequence from the testing set of the STEAD-BDLEELSSO dataset and generate the picking results. In this scenario, our FisH model can achieve the performance of 99% in terms of both precision and recall on this dataset.

### 中文翻譯與解說

**即時（Online）相位拾取：**

每個時間步使用記憶庫中最後 T=30 秒的歷史資訊生成當前拾取結果。

P 波拾取性能：
- Precision（精確率）：99%
- Recall（召回率）：99%

S 波拾取性能：
- Precision：96%
- Recall：90%

**離線（Offline）相位拾取（與 EQTransformer 同等評估設定）：**

使用完整 30 秒波形輸入（含 P/S 波後資訊）：
- Precision：99%，Recall：99%（P 波）

**與其他方法的比較表：**

### P 波拾取比較

| 模型 | Precision | Recall | F1 | 資料集 |
|------|-----------|--------|-----|--------|
| **FisH（本研究）** | **0.99** | **0.99** | **0.99** | BDLEELSSO |
| FisH | 0.99 | 0.98 | 0.99 | STEAD-Full |
| FisH | 0.98 | 0.81 | 0.89 | INSTANCE |
| LPPN | 0.95 | 0.94 | 0.94 | STEAD-Full |
| EPick | 0.95 | 0.97 | 0.96 | STEAD-Full |
| CapsPhase | 0.94 | 0.99 | 0.97 | STEAD-Full |
| EQTransformer | 0.99 | 0.99 | 0.99 | STEAD-Full |

### S 波拾取比較

| 模型 | Precision | Recall | F1 | 資料集 |
|------|-----------|--------|-----|--------|
| **FisH（本研究）** | **0.96** | **0.95** | **0.96** | BDLEELSSO |
| FisH | 0.95 | 0.94 | 0.95 | STEAD-Full |
| FisH | 0.92 | 0.79 | 0.85 | INSTANCE |
| LPPN | 0.83 | 0.84 | 0.84 | STEAD-Full |
| EPick | 0.95 | 0.95 | 0.95 | STEAD-Full |
| CapsPhase | 0.88 | 0.99 | 0.93 | STEAD-Full |
| EQTransformer | 0.99 | 0.96 | 0.98 | STEAD-Full |

**白話解說：**

FisH 的相位拾取性能達到或超越目前最佳方法（EQTransformer），而且 FisH 是在**即時串流模式**下做到這一點的！其他方法通常需要完整的波形才能達到最佳性能，FisH 在「不看未來資料」的更困難條件下仍能達到 99% 的 F1。

---

## 3.2 規模估計 / Magnitude Estimation

### 原文 (English)

> In our FisH, our Decoder directly estimates earthquake magnitude from the prediction embedding at each timestep. We use the P-arrival time as the reference time, and evaluate the online magnitude estimation performance by using the mean absolute error (MAE) between the estimated magnitude and the target magnitude from 2 seconds before the P-arrival time to 70 seconds after the P-arrival time.
>
> It can be clearly seen that the error quickly drops after the P wave arrival and continues to decrease to the minimum (around 0.15) at around 7 seconds later. Notably, the FisH model achieves a very low magnitude estimation error (0.18) within 3 seconds after P wave arrives. This rapid convergence to the optimal magnitude estimate is particularly impressive and highlights the model's potential for applications in earthquake early warning systems.

### 中文翻譯與解說

FisH 的 Decoder 在每個時間步直接從預測嵌入估計地震規模。以 P 波到達時間為基準，評估 P 波前 2 秒到 P 波後 70 秒的規模估計 MAE。

**關鍵發現：**
- P 波到達後，誤差**快速下降**
- **P 波後 3 秒**：規模 MAE = **0.18**（極早期！）
- **P 波後 7 秒**：達到最小誤差約 **0.15**
- S 波到達後 6 秒：達到最佳性能

**白話解說：**

地震規模估計通常需要等 S 波到達才夠準確，因為 S 波的振幅資訊更豐富。FisH 能在 **P 波到達僅 3 秒後**給出 MAE=0.18 的規模估計，這在地震預警中是相當驚人的。以台灣的例子來說，震中距 20 km 的地震，S 波約在 P 波後 5 秒到達，FisH 可以在 S 波到達前就給出不錯的規模估計！

---

## 3.3 位置估計 / Location Estimation

### 原文 (English)

> Traditionally, the location of earthquake is determined by estimating its distance and azimuth-deg to the corresponding station. In this work, our FisH model estimate the relative location of the earthquake (x_q, y_q) to the corresponding station. We use the location error to evaluate the location estimation performance of our FisH model, which is defined as the Euclidean distance between the estimated location (x_e, y_e) and the target location e = sqrt((x_q - x_e)² + (y_q - y_e)²).
>
> The location error quickly decays after the P-arrival and reaches the minimum of around 6km after approximately 10 seconds. FisH can fast converge and achieve 8.06 kilometers within 3 seconds after P-arrival. Our model has a focus limit of around 60 seconds, after which the model resets back to its initial state.
>
> Interestingly, we find that the error of the location estimation has an obvious dependence on the interval between the P-arrival and S-arrival, which is approximately equivalent to the distance. The error of the location prediction follows a power law (linear in log-log plot) with respect to the P-S time (i.e., the source distance).

### 中文翻譯與解說

FisH 直接估計震源相對台站的位置座標 (x_q, y_q)，位置誤差定義為估計位置與真實位置的歐氏距離。

**時間演進表現：**
- **P 波後 3 秒**：位置誤差 **8.06 km**
- **P 波後 10 秒**：達到最小約 **6 km**
- S 波到達後 1 秒：快速響應，達到最佳性能

模型有約 60 秒的「焦點限制」，之後自動重置回初始狀態。

**重要發現：Power Law（冪律）分佈**

位置估計誤差與 P-S 時間（近似等於震中距）之間呈**冪律關係**（log-log 圖中為線性），即：

誤差 ∝ (P-S 時間)^α

這說明震源越近，位置估計越準確；且這種依賴關係是冪律而非線性，是一個新的發現。

### 與其他方法比較

| 模型 | 位置誤差 | 距離誤差 | 方位角誤差 | 規模誤差 | 資料集 |
|------|---------|---------|-----------|---------|--------|
| **FisH（本研究）** | **6.0 km** | **2.6 km** | **19°** | **0.14** | BDLEELSSO |
| FisH | 9.4 km | 3.9 km | 22° | 0.15 | STEAD-Full |
| FisH | 8.5 km | 3.6 km | 22° | 0.15 | INSTANCE |
| MagNet | - | - | - | 0.20 | BDLEELSSO |
| SeisPairNet | - | 8.85 km | 31° | 0.29 | BDLEELSSO |
| Mousavi (2020) | 7.3 km | 5.4 km | 33° | - | BDLEELSSO |
| ComplexCNN | - | 4.51 km | - | 0.26 | BDLEELSSO |
| FourierTransformer | - | 3.77 km | - | 0.19 | BDLEELSSO |

**FisH 在所有指標上均達到最佳或次佳：**
- 位置誤差 6.0 km（比 Mousavi 的 7.3 km 更好）
- 方位角誤差 19°（比 SeisPairNet 的 31° 大幅改善）
- 規模誤差 0.14（所有比較方法中最低）

**白話解說：**

FisH 最令人驚訝的結果之一：它能在 **S 波到達之前**就給出不錯的位置估計！傳統單站位置估計方法幾乎都需要 S 波到達才能計算，因為 P-S 時間差是距離估計的關鍵。FisH 能在 S 波前就有合理預測，顯示它從 P 波本身的波形特徵中提取了足夠的位置資訊。

---

## 3.4 真實世界應用 / Real World Monitoring

### 原文 (English)

> To assess the real-world applicability of our model, we applied the FisH model to actual earthquake data collected from the SAGE website. In this scenario, the warning system efficiently locates the earthquake's position approximately 1 second after detecting the P-wave signal and provides an estimation of the event's magnitude.
>
> We apply the FisH model to the recent earthquake in Taiwan on April 9th. In this example, the warning system quickly locates the earthquake's position approximately 1 second after receiving the P-wave signal and provides an estimation of the event's magnitude. In the figure above, green nodes mark the predicted locations at each timestamp based on the P-wave arrival. The blue arrow represents the prediction trajectory. The lower part of the figure displays the location error from just after the P-wave arrival up to 50 seconds later.

### 中文翻譯與解說

研究團隊將 FisH 應用於**2024 年 4 月 9 日台灣地震**的真實資料（來自 SAGE/IRIS 資料庫），結果顯示：

- P 波訊號偵測到後**約 1 秒**，即能定位震源位置
- 同時提供規模估計

圖中：
- 綠色節點：各時間步基於 P 波到達的預測位置
- 藍色箭頭：預測軌跡（位置估計隨時間收斂的路徑）
- 下方波形圖：顯示 P 波到達後 50 秒內的位置誤差演進

**對台灣 EEW 的意義：**

這個案例驗證了 FisH 在真實台灣地震資料上的可行性。台灣擁有全球最密集的地震觀測網路之一（CWASN，約 600 個台站），FisH 的單站即時能力可以讓**每個台站獨立運作**，大幅降低對網路通訊和多站聯合處理的依賴，特別適合山區或離島等通訊條件較差的地區。

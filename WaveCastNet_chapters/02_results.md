# 結果 / Results

## 原文 (English)

> We evaluated our methodology by forecasting particle velocity waveforms near the Hayward Fault, simulating earthquake scenarios in the San Francisco Bay Area. Our prediction domain spans approximately 120 km along the X direction (parallel to the fault) and 80 km along the Y direction (perpendicular to the fault).
>
> We train WaveCastNet using low-magnitude, point-source synthetic waveforms and evaluate its performance across multiple settings. These include both dense, regularly gridded sensor locations and sparse sensor configurations. We further assess the robustness to background noise and test generalization to challenging cases such as out-of-distribution source locations, high-magnitude events, and real earthquake data.

### 2.1 Point-Source Small Earthquakes

> WaveCastNet is trained to forecast the next 100 seconds of the wavefield evolution based on an initial observation window. Rather than generating the entire sequence in a single pass, we adopt an iterative forecasting strategy. Specifically, the model predicts overlapping subsequences of 15.6 seconds (J=60 time steps at Δt=0.26 s), which are recursively fed into subsequent iterations. On an NVIDIA A100 GPU, WaveCastNet generates the complete forecast in less than one second.
>
> Accurate forecasts can be achieved with as little as 5.7 seconds of post-rupture data.

### 2.2 Generalization

> WaveCastNet generalizes robustly to events up to M5.5, and achieves reasonable accuracy for M6 earthquakes when using a sliding-window inference strategy. WaveCastNet does not receive any of these parameters during inference.
>
> To assess WaveCastNet's performance on real-world observations, we evaluate the 2018 M4.4 Berkeley earthquake. WaveCastNet was trained exclusively on synthetic data, without any fine-tuning on real waveforms, making this a strict zero-shot generalization scenario.

### Performance Metrics (Table 1)

| 輸入方式 | ACC | RFNE |
|----------|-----|------|
| Dense regular sampling | 0.98 | 0.20 |
| Sparse irregular sampling | 0.93 | 0.36 |

| Mw | Fault size (km×km) | T_rup (s) | ACC | RFNE |
|----|-------------------|----------|-----|------|
| 4.5 | 1.8×1.8 | 3.5 | 0.95 | 0.35 |
| 5.0 | 3.4×3 | 3.7 | 0.95 | 0.37 |
| 5.5 | 8×4 | 6.0 | 0.95 | 0.42 |
| 6.0 | 12.5×8 | 9.6 | 0.88 | 0.52 |
| 6.5 | 26×12 | 13.2 | 0.66 | 0.84 |
| 7.0 | 66×15 | 26.6 | 0.53 | 0.86 |

## 中文翻譯與解說

### 研究設定

實驗在加州舊金山灣區 Hayward 斷層附近進行，預測域為 120 km（沿斷層方向）× 80 km（垂直斷層方向）的矩形區域。

訓練資料使用低震級（M < 4.5）點源合成波場，由開源軟體 **SW4**（四階有限差分黏彈性模型）生成，速度模型採用 USGS 舊金山灣區三維地震速度模型 v21.1。

### 預測策略：迭代式預測

- 每次預測 15.6 秒（60 個時間步，Δt=0.26 s）
- 遞迴式輸入至下一次迭代，直到覆蓋完整 100 秒
- **最短僅需 5.7 秒的破裂後資料即可開始預測**（對 EEW 非常重要！）
- A100 GPU 完成整個 100 秒預報僅需 < 1 秒

### 稠密輸入模式（Dense）

輸入為完整 3×344×224 的波場張量（3 個速度分量 × 空間格點）。

**結果：**
- 成功重建 P 波、S 波波前及散射尾波
- PGV 偏差通常在 5% 以內
- T_PGV（PGV 到時）誤差幾乎可忽略（除地質複雜邊界區域）
- ACC = 0.98，RFNE = 0.20

### 稀疏輸入模式（Sparse）

使用 101 個 ShakeAlert 台站的稀疏量測（3×101 張量）作為輸入，預測完整波場。

採用**隨機遮罩（MAE 策略）**訓練嵌入層：80% 台站隨機遮罩，讓模型學會從稀疏觀測中重建完整波場。

**結果：**
- 雖然預測誤差稍高，但仍能捕捉地震波傳播的主要特徵
- ACC = 0.93，RFNE = 0.36

### 不確定性估計（Ensemble）

訓練 50 個不同初始化的 WaveCastNet 實例，計算集成均值和標準差：
- 標準差通常低於均值的 1%，顯示高可靠性
- 在 Livermore 盆地（沉積盆地）附近偏差略高，反映複雜波場交互作用

### 泛化能力結果

**訓練外震源位置：** Hayward 斷層北側或聖安德烈斯斷層的震源仍可合理預測（Hayward 表現較佳）

**有限斷層大地震（M4.5–M7）：**
- M ≤ 5.5：準確度仍高（ACC ≥ 0.95）
- M 6.0：開始下降（ACC = 0.88），因為破裂持續時間接近輸入窗口
- M 6.5–7.0：顯著下降，振幅被低估（破裂持續時間超過輸入窗口）
- 延長輸入窗口可改善高震級預測

**2018 年 M4.4 Berkeley 真實地震（零樣本測試）：**
- 僅用合成資料訓練，完全無真實資料微調
- 成功重現 S 波、面波等主要特徵
- PGV 預測與觀測值高度一致
- T_PGV 散點較大，反映合成與真實資料的域差距

---

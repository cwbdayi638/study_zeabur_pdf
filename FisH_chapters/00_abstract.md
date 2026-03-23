# 摘要 / Abstract

## 原文 (English)

> Earthquake early warning (EEW) systems play a crucial role in mitigating the impacts of earthquakes by providing advance warnings to areas at risk. These systems analyze seismic data to estimate the location and magnitude of earthquakes, offering valuable time for appropriate responses. However, existing EEW approaches often treat phase picking, location estimation, and magnitude estimation as separate tasks, lacking a unified framework. Additionally, most deep learning models in seismology rely on full three-component waveforms and are not suitable for real-time streaming data. To address these limitations, we propose a novel unified seismic neural network called Fast Information Streaming Handler (FisH). FisH is designed to process real-time streaming seismic data and generate simultaneous results for phase picking, location estimation, and magnitude estimation in an end-to-end fashion. By integrating these tasks within a single model, FisH simplifies the overall process and leverages the nonlinear relationships between tasks for improved performance. The FisH model utilizes RetNet as its backbone, enabling parallel processing during training and recurrent handling during inference. This capability makes FisH suitable for real-time applications, reducing latency in EEW systems. Extensive experiments conducted on the STEAD benchmark dataset provide strong validation for the effectiveness of our proposed FisH model. The results demonstrate that FisH achieves impressive performance across multiple seismic event detection and characterization tasks. Specifically, it achieves an F1 score of 0.99/0.96. Also, FisH demonstrates precise earthquake location estimation, with a location error of only 6.0km, a distance error of 2.6km, and a back-azimuth error of 19°. The model also exhibits accurate earthquake magnitude estimation, with a magnitude error of just 0.14. Additionally, FisH is capable of generating real-time estimations, providing location and magnitude estimations with a location error of 8.06km and a magnitude error of 0.18 within a mere 3 seconds after the P-wave arrives. These findings highlight the model's ability to rapidly and accurately assess seismic events. The results demonstrate its potential to enhance EEW systems by providing accurate and timely information for earthquake monitoring and response.

## 中文翻譯與解說

地震預警（EEW）系統透過向危險地區提供預先警報，在減輕地震影響方面發揮了關鍵作用。這些系統分析地震資料以估計地震的位置和規模，為適當的應對措施提供寶貴的時間窗口。

**現有方法的三大問題：**
1. 現有 EEW 方法通常將相位拾取（Phase Picking）、位置估計（Location Estimation）和規模估計（Magnitude Estimation）視為**獨立的任務**，缺乏統一框架
2. 大多數地震學深度學習模型依賴完整的**三分量波形**（三個方向的加速度計資料），不適合即時串流資料
3. 高延遲問題：分開處理多項任務需要更多計算時間

**FisH 的解決方案：**

本文提出 **Fast Information Streaming Handler（FisH）**，一種全新的統一地震神經網路。FisH 的核心設計目標：
- 處理**即時串流地震資料**（非事後批次）
- 以**端到端（end-to-end）**方式同時輸出相位拾取、位置估計、規模估計三項結果
- 在單一模型中整合三項任務，利用任務間的**非線性關係**提升各項表現

**RetNet 骨幹的優勢：**

FisH 使用 RetNet（Retentive Network）作為核心架構：
- **訓練時**：平行處理長序列，效率高
- **推論時**：轉換為遞迴神經網路（RNN）模式，每步計算成本極低（O(1)）

**實驗結果摘要（STEAD 基準資料集）：**

| 指標 | 數值 |
|------|------|
| P 波拾取 F1 | 0.99 |
| S 波拾取 F1 | 0.96 |
| 位置誤差 | 6.0 km |
| 距離誤差 | 2.6 km |
| 方位角誤差 | 19° |
| 規模 MAE | 0.14 |
| P 波後 3 秒位置誤差 | 8.06 km |
| P 波後 3 秒規模誤差 | 0.18 |

---

**白話解說：**

想像一下，現有的 EEW 系統就像工廠的三條獨立生產線——一條只做「偵測地震波到達」，另一條只做「算震源在哪」，第三條只做「算地震多大」。這三條線各自需要獨立的模型和完整的資料，彼此不互通，自然效率低落。

FisH 的創新在於把這三條生產線合成一條，而且這條線不需要等一批貨（完整波形）到齊才能開工——它是「即時輸送帶」，波形資料一進來就一邊傳送一邊分析，P 波到達後短短 **3 秒**就能給出位置和規模估計。這對地震預警爭分奪秒的需求來說，意義非凡。

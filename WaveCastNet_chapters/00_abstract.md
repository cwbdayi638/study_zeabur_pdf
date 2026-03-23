# 摘要 / Abstract

## 原文 (English)

> We propose a new deep learning model, WaveCastNet, to forecast high-dimensional wavefields. WaveCastNet integrates a convolutional long expressive memory architecture into a sequence-to-sequence forecasting framework, enabling it to model long-term dependencies and multiscale patterns in both space and time. By sharing weights across spatial and temporal dimensions, WaveCastNet requires significantly fewer parameters than more resource-intensive models such as transformers, resulting in faster inference times. Crucially, WaveCastNet also generalizes better than transformers to rare and critical seismic scenarios, such as high-magnitude earthquakes. Here, we show the ability of the model to predict the intensity and timing of destructive ground motions in real time, using simulated data from the San Francisco Bay Area. Furthermore, we demonstrate its zero-shot capabilities by evaluating WaveCastNet on real earthquake data. Our approach does not require estimating earthquake magnitudes and epicenters, steps that are prone to error in conventional methods, nor does it rely on empirical ground-motion models, which often fail to capture strongly heterogeneous wave propagation effects.

## 中文翻譯與解說

本文提出一個新的深度學習模型 **WaveCastNet**，用於預測高維度地震波場。

WaveCastNet 將**卷積長表達記憶（ConvLEM）**架構整合進序列到序列（Seq2Seq）預測框架中，使其能夠捕捉空間與時間上的長期依賴關係以及多尺度模式。

由於在空間與時間維度上共享權重，WaveCastNet 所需的參數量遠少於 Transformer 等資源密集型模型，推論速度更快。

更關鍵的是，WaveCastNet 對罕見但高度危險的地震情境（例如高震級地震）的泛化能力優於 Transformer。

本文利用舊金山灣區的模擬資料，展示了模型即時預測破壞性地面運動的強度與到時的能力，並進一步透過真實地震資料評估 **zero-shot（零樣本）** 泛化能力。

### 🔑 重點白話解釋

- **不需要估算震源參數**（震央、規模），直接從波場資料預測，省去傳統 EEW 最容易出錯的步驟。
- **不依賴經驗性地動模型（GMPE）**，避免因模型假設不符合台灣複雜地質而失效。
- **zero-shot**：模型只用合成資料訓練，直接應用到真實地震錄製，無需重新訓練即可使用。

---

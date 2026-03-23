# FisH 論文導讀 / FisH Paper Guide

**論文全名：** Fast Information Streaming Handler (FisH): A Unified Seismic Neural Network for Single Station Real-Time Earthquake Early Warning

**作者：** Tianning Zhang, Feng Liu, Yuming Yuan, Rui Su, Wanli Ouyang, Lei Bai

**arXiv：** [2408.06629](https://arxiv.org/abs/2408.06629)

---

## 📚 章節目錄 / Table of Contents

| 檔案 | 內容 |
|------|------|
| [00_abstract.md](00_abstract.md) | 摘要 Abstract |
| [01_introduction.md](01_introduction.md) | 第1章 Introduction |
| [02_methods.md](02_methods.md) | 第2章 Methods（FisH 架構詳解） |
| [03_results.md](03_results.md) | 第3章 Results（實驗結果） |
| [04_discussion.md](04_discussion.md) | 第4章 Discussion（架構選擇與比較） |
| [05_conclusion.md](05_conclusion.md) | 結論 + 對台灣 EEW 的啟示 |

---

## 🔑 論文核心重點

### FisH 是什麼？
FisH（**F**ast **I**nformation **S**treaming **H**andler）是一個統一的地震神經網路，能從**單一地震站**的**即時串流資料**中，同時完成：
1. **相位拾取（Phase Picking）**：自動偵測 P 波與 S 波的到達時間
2. **位置估計（Location Estimation）**：估計震源相對觀測站的位置
3. **規模估計（Magnitude Estimation）**：估計地震規模

### 四大核心特點
1. **單站即時串流**：不需等待完整波形，隨時間步驟逐一處理，P 波到達後 3 秒內即可給出結果
2. **統一框架（Unified Framework）**：三項任務在同一神經網路中同時完成，而非三個獨立模型
3. **RetNet 骨幹架構**：訓練時用平行模式加速，推論時用遞迴（RNN）模式，計算成本 O(1)
4. **Sea 模式訓練**：控制「地震」與「非地震」狀態間的 hidden increment，讓模型可連續不間斷運行

---

## 📊 主要性能指標

| 任務 | 指標 | 數值 |
|------|------|------|
| P 波拾取（即時） | Precision / Recall | 99% / 99% |
| S 波拾取（即時） | F1 | 0.96 |
| 位置誤差（20秒後） | Location Error | 6.0 km |
| 距離誤差 | Distance Error | 2.6 km |
| 方位角誤差 | Back-azimuth Error | 19° |
| 規模誤差 | Magnitude MAE | 0.14 |
| **即時（P 波後 3 秒）** | Location / Magnitude | 8.06 km / 0.18 |

---

## 📖 術語對照表 / Glossary

| 英文術語 | 中文 | 說明 |
|----------|------|------|
| Earthquake Early Warning (EEW) | 地震預警 | 地震發生後、破壞性波到達前的預警系統 |
| Phase Picking | 相位拾取 | 自動偵測 P 波與 S 波到達時刻 |
| P-wave / S-wave | P 波 / S 波 | 縱波（初達波）/ 橫波（破壞性較強） |
| Location Estimation | 位置估計 | 估計震源位置（距離 + 方位） |
| Magnitude Estimation | 規模估計 | 估計地震規模（芮氏規模等） |
| Back-azimuth | 後方位角 | 震源相對台站的方位角 |
| RetNet | 保留網路 | Retentive Network，具平行訓練與遞迴推論雙重能力 |
| Retention Mechanism | 保留機制 | RetNet 核心，具指數衰減的記憶機制 |
| Embedder | 嵌入器 | 將波形轉換為高維嵌入向量的模組 |
| Encoder | 編碼器 | 提取嵌入向量間的相關性（RetNet） |
| Decoder | 解碼器 | 將預測嵌入解碼為任務輸出 |
| MSL (MultiScalerLayer) | 多尺度層 | Embedder 的基本單元，多尺度 CNN 特徵聚合 |
| MSR (Multi-scale Retention) | 多尺度保留 | Encoder 的核心，多頭保留機制 |
| Sea Mode | Sea 模式 | 進階訓練模式，控制「地震/非地震」狀態轉換 |
| Streaming Data | 串流資料 | 即時、連續傳入的資料流（非事後批次處理） |
| STEAD | Stanford 地震資料集 | Stanford Earthquake Dataset，全球 AI 地震訓練資料 |
| INSTANCE | 義大利地震資料集 | 用於驗證模型泛化能力 |
| STA/LTA | 短期/長期平均比 | 傳統相位拾取方法 |
| EWM | 指數加權移動 | Exponential Weighted Moving，用於即時波形分解 |
| MAE | 平均絕對誤差 | Mean Absolute Error |

---

## 🌏 對台灣 EEW 研究的意義

- FisH 論文中明確展示了其在**台灣 2024 年 4 月 9 日地震**的實際應用（圖 6），顯示對台灣地震場景的適用性
- 論文引用了台灣 EEW 系統相關文獻（陳達毅等人的 Earthworm-based Taiwan EEW）
- 台灣地震密集，單站即時 EEW 對資源有限的偏遠站點特別有價值
- RetNet 的 O(1) 推論成本讓模型可部署在站端小型晶片，無需雲端超算支援

# WaveCastNet: Rapid Wavefield Forecasting for Earthquake Early Warning

> **arXiv:** [2405.20516](https://arxiv.org/abs/2405.20516)  
> **最新版：** 2025年10月  
> **作者：** Dongwei Lyu, Rie Nakata, Pu Ren, Michael W. Mahoney, Arben Pitarka, Nori Nakata, N. Benjamin Erichson  
> **機構：** UC Berkeley / Lawrence Berkeley National Laboratory / Lawrence Livermore National Laboratory / 東京大學地震研究所

---

## 核心問題

傳統 EEW 流程（如 Earthworm）：

> **偵測 P 波 → 估計震央/規模 → 套用經驗地動模型 → 發預警**

**兩個致命弱點：**
1. 規模估計誤差大（初期只有幾秒資料），常造成漏報或誤報
2. 經驗模型假設均質地球，無法捕捉地質構造造成的複雜波場效應（如盆地放大）

---

## 核心創新：直接預測波場

WaveCastNet **完全跳過**震央估計和規模估計，直接從觀測波形預測未來波場演化：

```
觀測波形（5.7秒）→ WaveCastNet → 預測未來100秒波場
```

不需要：
- ❌ 震央位置估計
- ❌ 規模估計
- ❌ 經驗地動模型（GMM）

---

## 模型架構

### Seq2Seq + ConvLEM

```
輸入序列（觀測波形）
    ↓
Encoder（ConvLEM 堆疊）
    ↓ 壓縮成潛在狀態
Decoder（ConvLEM 堆疊）
    ↓ 迭代生成 15.6 秒子序列
輸出：完整 100 秒波場預測
```

### ConvLEM（卷積長表達記憶）— 核心創新

- 將 LEM（Long Expressive Memory）與卷積層整合
- 同時捕捉**時間多尺度**與**空間多尺度**結構
- 比 ConvLSTM 更能建模時間多尺度
- 比 Transformer **參數更少、推論更快**
- 在高規模地震的泛化能力**優於 Transformer**

### 迭代預測策略

- 每次預測 15.6 秒（60 time steps，Δt = 0.26s）
- 遞迴輸入前一段預測，直到覆蓋完整 100 秒
- 可從**僅 5.7 秒**的破裂後資料開始預測

---

## 實驗結果

| 能力 | 結果 |
|------|------|
| PGV 預測誤差 | 通常 < 5% |
| 推論速度 | NVIDIA A100 上 **< 1 秒**完成 100 秒預測 |
| 最短輸入 | 僅需 **5.7 秒**後震波形即可開始預測 |
| 稀疏感測器 | 僅 101 個站點仍可重建全場波形 |
| 零樣本測試 | 直接用真實地震資料，未見過仍能預測 |
| 不確定性估計 | 集成 50 個模型，lnPGV 標準差 < 1% |

### 測試場景
- **舊金山灣區 Hayward 斷層**模擬資料（合成）
- 點源地震（M < 4.5）→ 訓練
- 訓練區外點源（Hayward / San Andreas 斷層）→ 泛化測試
- 有限斷層大地震（M4.5–M7）→ 零樣本泛化
- **2018 Berkeley 真實地震** → 零樣本真實資料測試

---

## 與 Earthworm (pick_eew) 對比

| 特性 | Earthworm (pick_eew) | WaveCastNet |
|------|---------------------|-------------|
| 核心方法 | P 波拾取 + 規模估計 | 波場序列預測（Seq2Seq） |
| 需要震央估計 | ✅ 是 | ❌ 不需要 |
| 需要規模估計 | ✅ 是 | ❌ 不需要 |
| 預測內容 | 震度等級 | 完整波場（PGV、波形） |
| 輸入資料 | 各站 P 波到時 | 各站速度波形時序 |
| 推論速度 | 秒級（規則式） | < 1 秒（GPU） |
| 可解釋性 | 高（物理參數） | 中（深度學習黑箱） |
| 訓練資料需求 | 低 | 高（需大量模擬波場） |
| 地質異質性處理 | 依賴 GMM，較差 | 直接學習，較佳 |

---

## 模型優勢

1. **預測精度高**：優於 ConvLSTM、GRU、Transformer 等基準模型
2. **彈性輸入**：同時支援密集波場輸入與稀疏感測器輸入
3. **泛化能力強**：可推廣至高規模事件，具備零樣本能力
4. **抗噪聲**：在背景噪聲和資料延遲下仍維持效能
5. **不確定性估計**：集成預測提供可靠的不確定性量化
6. **免參數估計**：不需事件偵測、震源參數估計或 GMM

---

## 限制

1. **大規模有限斷層地震（M7+）**：有限斷層幾何複雜，訓練資料多樣性不足，振幅縮放不穩定
2. **頻率限制**：目前僅處理 < 0.5 Hz，工程應用需更高頻段
3. **合成→真實資料 Domain Gap**：需更多觀測資料、資料增強與物理約束正則化

---

## 對台灣 EEW 研究的意義

WaveCastNet 代表 EEW 的**下一代範式**——不再依賴物理參數估計，直接學習波場動力學。

台灣具有：
- 密集的 CWASN + TSMIP 雙網觀測系統（與本研究稀疏感測器配置契合）
- 豐富的歷史地震記錄（可用於訓練）
- 複雜的地質構造（盆地效應顯著，正是 GMM 失效的場景）

**建議整合方向：**
- 以台灣地震波場模擬（如 SW4 + TSMIP 地殼速度模型）產生訓練資料
- 使用現有 CWASN/TSMIP 稀疏站網設定訓練 WaveCastNet
- 與 Earthworm pick_eew 並行運作，作為互補驗證

---

## 引用

```bibtex
@article{lyu2024wavecastnet,
  title={WaveCastNet: Rapid Wavefield Forecasting for Earthquake Early Warning via Deep Sequence to Sequence Learning},
  author={Lyu, Dongwei and Nakata, Rie and Ren, Pu and Mahoney, Michael W. and Pitarka, Arben and Nakata, Nori and Erichson, N. Benjamin},
  journal={arXiv preprint arXiv:2405.20516},
  year={2024}
}
```

---

*摘要整理日期：2026-03-23*

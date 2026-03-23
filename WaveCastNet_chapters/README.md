# WaveCastNet 論文中英對照筆記 / WaveCastNet Bilingual Chapter Notes

**論文全名：** WaveCastNet: Rapid Wavefield Forecasting for Earthquake Early Warning via Deep Sequence to Sequence Learning

**作者：** Dongwei Lyu, Rie Nakata, Pu Ren, Michael W. Mahoney, Arben Pitarka, Nori Nakata, N. Benjamin Erichson

**arXiv：** [2405.20516](https://arxiv.org/abs/2405.20516)

**整理日期：** 2026-03-23

---

## 章節索引 / Chapter Index

| 檔案 | 中文章節 | English Section |
|------|----------|-----------------|
| [00_abstract.md](./00_abstract.md) | 摘要 | Abstract |
| [01_introduction.md](./01_introduction.md) | 引言 | Introduction |
| [02_results.md](./02_results.md) | 結果 | Results |
| [03_discussion.md](./03_discussion.md) | 討論 | Discussion |
| [04_methods.md](./04_methods.md) | 方法（含 ConvLEM 數學推導） | Methods (with ConvLEM derivation) |
| [05_conclusion.md](./05_conclusion.md) | 結論與台灣 EEW 啟示 | Conclusion & Taiwan EEW Implications |

---

## 論文核心貢獻摘要

WaveCastNet 是一個基於深度序列到序列（Seq2Seq）學習的地震波場預測框架，核心創新在於引入 **ConvLEM（卷積長表達記憶）** 架構，能在不需要估計震源參數（震央、規模）的情況下，直接從波場時序預測未來 100 秒的地面運動，且僅需 0.56 秒完成推論（NVIDIA A100 GPU），適合整合至地震預警系統（EEW）。

---

## 對台灣 EEW 的啟示

台灣位於歐亞板塊與菲律賓海板塊交界，地震活動頻繁，現行 EEW 系統（如 P-alert）主要依賴到時偵測與震源參數估算。WaveCastNet 的「無需震源參數」設計，理論上可大幅減少因震源估算誤差導致的漏報或誤報，值得台灣地震研究圈深入關注與移植。

# 討論 / Discussion

## 原文 (English)

> Our experiments demonstrate that WaveCastNet holds substantial promise for forecasting seismic wavefields derived from both point-source and finite-fault simulations of large-magnitude earthquakes, as well as from real observational data, such as the 2018 M4.4 Berkeley event. Across both dense and irregularly sparse sensor configurations, WaveCastNet reliably predicts seismic wave propagation and captures PGV values and their timing with high fidelity.
>
> We attribute this predictive performance in part to WaveCastNet's ability to internalize the Huygens principle: each spatial point acts as a secondary source of wavefronts. This principle helps explain the model's ability to generalize to out-of-distribution events.
>
> WaveCastNet generates 100-second forecasts in just 0.56 seconds on a single NVIDIA A100 GPU, demonstrating its suitability for real-time applications.
>
> However, applying a model trained on point-source simulations to large-magnitude events remains a significant challenge. As earthquakes increase in size, their physical representation transitions from point sources to extended rupture planes governed by complex kinematic models. Waveform amplitudes can vary by factors exceeding 80 between M4.5 and M7 events, and spatial decay rates of ground-motion intensity are magnitude-dependent.
>
> **Comparative study:** WaveCastNet outperforms Seq2Seq frameworks using ConvLSTM and ConvGRU backbones, as well as state-of-the-art transformers (Swin Transformer, Time-S-Former). While larger vision transformers may achieve higher accuracy on in-domain tasks, WaveCastNet generalizes best to domain-shifted settings with fewer parameters.

### Future Directions

> 1. Extending the Magnitude and Frequency Range — incorporate high-magnitude finite-fault simulations
> 2. Generative Modeling for Uncertainty-Aware Forecasting — explore diffusion models
> 3. Self-Supervised Pretraining for Foundation Models — reduce dependence on labeled data
> 4. Fine-Tuning and Domain Adaptation with Real Seismic Data
> 5. Accelerating Inference for Real-Time Forecasting — mixed-precision inference, pruning, quantization

## 中文翻譯與解說

### 模型性能的物理詮釋

WaveCastNet 的優異預測性能，部分歸因於模型成功內化了**惠更斯原理（Huygens' principle）**：空間中每個點都作為二次波前的波源。這解釋了模型能夠將波場動力學推廣到訓練域外事件的能力——包括聖安德烈斯斷層的點震源和有限斷層破裂。

### 與基準模型的比較

| 模型 | 參數量（百萬） | 點源 RFNE | 泛化能力（高震級） |
|------|---------------|-----------|-------------------|
| ConvGRU | 小 | ~0.34 | 差 |
| ConvLSTM | 小 | ~0.32 | 差 |
| Swin Transformer | 13.72M | ~0.26 | 較差（泛化差距大） |
| Swin Transformer* | 24.27M | ~0.24 | 差 |
| Time-S-Former | 10.21M | ~0.28 | 差 |
| Time-S-Former* | 33.82M | ~0.22 | 差 |
| **WaveCastNet** | **少** | **最佳均衡** | **最佳** |

**關鍵發現：** 大型 Transformer 雖在訓練域內表現略好，但泛化能力最差；WaveCastNet 以更少的參數實現最佳的泛化，說明其信息瓶頸（information bottleneck）帶來了隱式正則化效果。

### 大震級推廣的挑戰

- M4.5 至 M7 之間，地動振幅可相差超過 80 倍
- 大震的破裂持續時間（如 M6.5 為 13.2 秒，M7 為 26.6 秒）可能超過 5.7 秒的輸入窗口
- 解決方法：延長輸入窗口、訓練資料納入有限斷層模擬

### 未來發展方向

1. **擴展震級與頻率範圍**：目前僅訓練到 0.5 Hz，需擴展到更高頻率以支援工程應用
2. **生成模型（擴散模型）**：用於更完整的不確定性量化，產生波場的概率預報
3. **自監督預訓練（SSL）**：利用大量未標記波形資料進行預訓練，減少對合成標記資料的依賴
4. **遷移學習與域適應**：將合成訓練模型微調到真實地震觀測
5. **加速推論**：混合精度、模型剪枝、量化，進一步降低計算延遲

---

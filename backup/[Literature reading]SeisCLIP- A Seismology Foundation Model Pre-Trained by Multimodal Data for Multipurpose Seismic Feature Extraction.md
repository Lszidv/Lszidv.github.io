# Abstract 
### **SeisCLIP: 基于对比学习的地震学基础模型**

- **核心目标**：解决地震学研究中标注数据稀缺和模型区域泛化能力不足的问题。
- **方法**：
  - 1.采用 **对比学习** 进行预训练；2.结合 **Transformer 频谱编码器** 和 **MLP 事件信息编码器**。
- **优势**：
  - 1.适用于**多个地震学任务**（分类、定位、震源机制分析）；2.在**不同区域数据集**上表现优于基线方法；3.仅需少量数据即可微调。
---
# 1.Introduction
### ** 研究背景**
- **机器学习（ML）与深度学习（DL）** 在多个地震学任务中优于传统方法：
  - **去噪、震相拾取、地震检测、震源机制分析、地震预测** 等。
- **现有方法局限性**：
  - **单任务模型** 无法适应数据不足的任务。
  - **迁移学习（TL）** 受限于特定子领域。
  - **基础模型（Foundation Model）** 能提升泛化能力。

- **定义**：先 **预训练**，再 **微调（Fine-tuning）**，适应多个下游任务。


### **预训练策略**
- **生成式学习**（如 BERT、MAE）：   通过 **遮蔽部分数据** 进行预测。
- **对比式学习**（如 CLIP）：让匹配样本对靠近，错误样本远离。

### **地震学基础模型**
- **StorSeismic（BERT-based）** 和 **SFM（MAE-based）** 主要用于 **地震勘探**（2D 数据）。
- **SeisCLIP（对比学习）**：
  - 结合 **地震频谱**（Transformer 编码）和 **事件信息**（MLP 编码）。
  - 采用 **STEAD 数据集** 进行多模态训练。
### **结论**
- **挑战**：
  - 2D 地震模型难适用于 1D 波形。
  - 任务间数据不均衡。
- **解决方案**：
  - **SeisCLIP** 采用 **对比学习** 进行 **多模态预训练**。
  - 结合 **Transformer+MLP** 提取 **地震特征**。
---

# 2.Methods
### A. 地震学基础模型的架构
#### **1. 研究目标**
- 提出 **SeisCLIP**，采用 **对比学习（Contrastive Learning）** 进行 **预训练**。
- 解决 **长序列地震波形处理的计算开销问题**，提高模型在 **地震学任务** 中的适用性。

#### **2. 网络架构**
- **双分支编码器**
  - **频谱分支（Spectrum Branch）**：
    - **STFT 转换地震波形为频谱**，采用 **VIT-small** （图像识别）进行编码。
  - **事件信息分支（Information Branch）**：
    - 处理 **P/S 波到时、震源距离等 8 种参数**，并使用 **MLP 进行编码**。

#### **3. 训练策略**
- **采用频谱表示**，(波形太长)减少计算开销，提高 Transformer 处理地震数据的适用性。
- **对比学习（Contrastive Learning）**：
  - 让模型学习 **匹配正确的频谱与事件信息**，提高泛化能力。
- **训练样本结构**
  - **三通道频谱数据 + 八种事件信息**，分别输入到编码器进行联合训练。

## **B. 预训练和验证的数据准备**
- 采用 **STEAD 数据集**，包含 **1,030,000 条地震信号** 和 **8 种事件信息**。
- 训练集：**1,000,000 条**，验证集：**30,000 条**。

 **下游任务数据集**
### **（1）地震定位（Localization）**
- **数据来源**：日本地震数据，**M > 2**，**包含 2011 年东日本大地震**。
- **地震站**：NIED Hi-net，**提供高分辨率三分量地震数据**。
- **数据划分**：
  - 训练集：**10,000**
  - 验证集：**1,000**
  - 测试集：**1,000**

### **（2）震源机制分析（Focal Mechanism Analysis）**
- **数据来源**：日本地震数据，**M > 3**，2011-2016 年，JMA 提供震源机制解。
- **数据划分**：训练集：**随机划分**，具体信息见 **表 II**。

 **数据预处理**
- **所有波形数据** 统一处理：**截取 60 秒片段**；**100 Hz 采样率**；**STFT 转换为时频谱**； **均值-标准差归一化**

## **C. 通过对比学习（Contrastive Learning）预训练 SeisCLIP**

**预训练策略**
- **目标**：
  - **联合训练** 频谱编码器和信息编码器，使其学习 **匹配的地震频谱与事件信息**。
- **对比学习（Contrastive Learning）**：
  - **最大化真实样本对的余弦相似度**。
  - **最小化错误匹配样本对的余弦相似度**（共 N² - N 个错误组合）。
- **优化策略**：采用 **对称交叉熵损失** 确保训练稳定。

**训练细节**
- **超参数设置**： **学习率（Learning Rate）** = 1e-4； **批量大小（Batch Size）** = 192； **训练轮数（Epochs）** = 100
- **模型选择**： **第 55 轮的模型在验证集上表现最佳**，被选为最终预训练模型。
- **模型部署**
- 训练完成后，**SeisCLIP 频谱编码器可直接用于下游任务**：**事件分类**； **地震定位**；**震源机制分析**
## **D. SeisCLIP 在下游任务中的应用**

**下游任务**: **事件分类（Event Classification）**;**地震定位（Earthquake Localization）**;**震源机制分析（Focal Mechanism Analysis）**

**训练策略**
- **Fine-Tune**：微调 **预训练的 VIT-small 频谱编码器**。
- **Frozen**：冻结编码器，只训练任务层。
- **Scratch**：不使用 SeisCLIP 预训练权重，从零训练。

**基线模型对比**
- **波形模型（Waveform-Based）**
- **频谱模型（Spectrum-Based）**
- **对比 SeisCLIP 频谱编码器与传统方法**
![Image](https://github.com/user-attachments/assets/8c95c553-55f9-49ea-81b1-e43ec9a9d090)
---
# 3.RESULTS
 ##**A. 事件分类（Event Classification）**

**评估方法**
- 采用 **一对多（One-vs-All）** 方式评估分类任务。
- 计算 **ROC 曲线和 AUC 值** 评估分类能力。

**结果分析**
- **Fine-Tune 模型的 AUC 值最高，优于 Frozen 和 Scratch**。
- **混淆矩阵显示 Fine-Tune 模型分类准确率更高**。
- **由于 STEAD 预训练数据集中仅有地震数据，Frozen 模型受限，而 Scratch 模型表现更稳定**。

**泛化能力**
- **在 SCSN 二分类任务（地震 vs. 爆炸）中，Fine-Tune 模型的 AUC 最高**，展现了 **SeisCLIP 的优越泛化能力**。

## **B. 事件定位（Event Location）**

**震中距离预测**
- **Fine-Tune 模型的震中定位误差更低，优于基线模型**。
- **图 6(a), (c) 中 SeisCLIP 预测的震中点颜色更亮，表示误差更小**。
- **Fine-Tune 模型的误差分布更接近零，整体精度更高**。

**震源深度预测**
- **Fine-Tune 模型在震源深度预测中也优于 Frozen 和 Baseline**。
- **图 6 第三行：Fine-Tune 模型的高亮区域颜色更亮，误差更小**。
- **Fine-Tune 模型的残差分布最窄，MAE 误差最低**（图 6 第四行）。

**其他定位因素**
- **除了震中距离和深度，SeisCLIP 还优化了经度、纬度和震级估计**（图 7）。
- **Fine-Tune 模型的误差分布更集中，显示其优越的震级预测能力**。

**结论**
- **SeisCLIP 在地震事件定位任务中的表现显著优于传统基线方法**。
- **Fine-Tune 版本的泛化能力更强，可用于更广泛的地震学任务**。

## **C. 震源机制分析（Focal Mechanism Analysis）**

**任务定义**
- **目标：** 预测地震的震源机制（正断层、逆断层、走滑断层）。
- **挑战：** 震源机制原本是回归问题（预测 Strike, Dip, Rake），但数据有限，因此转化为分类任务。
- **输入：** **多台站地震数据**，用于震源机制分类。
- **输出：** 预测 **正断层（Normal）、逆断层（Reverse）、走滑断层（Strike-Slip）**（见 **图 1(d)**）。

**评估方法**
- **使用 PNW 数据集的分类评估标准**：
  - **ROC 曲线（Receiver Operating Characteristic Curve）**
  - **AUC 值（Area Under the Curve）**
  - **混淆矩阵（Confusion Matrix）**

## **3. 结果分析**
- **ROC 曲线（图 8 第一行）**：
  - **SeisCLIP 在所有断层类型上表现最佳**。
  - **Fine-Tune 模型的 AUC 最高**，Frozen 也明显优于 Baseline。
- **混淆矩阵分析**：
  - **Fine-Tune 分类正确率最高**。
  - **Frozen 仍然比 Baseline 强，说明预训练特征有效**。
---
# 4. DISCUSSION
### **1. SeisCLIP 预训练特征分析与可视化**
SeisCLIP 采用 **预训练（Pretraining）+ 微调（Fine-Tuning）** 方式，在多个地震学任务中展现优异性能。  
通过 **t-SNE 降维分析**，可视化 **Frozen、Scratch、Fine-Tuned** 三种训练策略的特征分布，以理解模型学习到的特征。

**预训练与微调**
- **预训练阶段**：
  - 在 **STEAD 数据集** 上训练，学习 **地震频谱的基本特征**。
  - 主要学习 **地震事件的基本频谱模式**，但无法精准区分 **爆炸 vs. 地震**。
- **微调阶段**：
  - 在 **PNW、SCSN、日本数据集** 上进一步优化，提高任务适应性和泛化能力。
  - Fine-Tune 使模型能够更清晰地区分 **地震、爆炸和地表事件**。

**t-SNE 降维分析**
- **t-SNE 主要用于高维特征投影到 2D 空间，便于观察特征聚类情况**。
- **不同训练策略的特征聚类情况（图 9）**：
  - **Frozen 模型**：仅能区分 **地震 vs. 非地震**，但无法区分 **爆炸 vs. 地震**。
  - **Scratch 模型**：特征分布较散乱，类别边界不清晰，未能有效学习地震信号特征。
  - **Fine-Tune 模型**：特征聚类最清晰，类别边界明确，能够精确区分 **地震、爆炸和地表事件**。

### **2. STFT 参数对 SeisCLIP 模型的影响与计算成本分析**
SeisCLIP 预训练时采用 **短时傅里叶变换（STFT）** 进行地震波形到频谱的转换，不同 STFT 参数对模型分类与定位任务的影响显著。

**STFT 设定**
- **频率点数**：50
- **频率范围**：0-50 Hz
- **时间采样窗口**：
  - **120 采样点（适用于分类任务）**
  - **600 采样点（适用于定位任务）**

**STFT 参数对任务表现的影响**
- **短频谱窗口（120 采样点）**：
  - 适用于 **事件分类任务**，因分类任务主要依赖频谱特征而非时间信息。
  - 计算成本低，训练更快。
- **长频谱窗口（600 采样点）**：
  - 适用于 **震中定位任务**，因震中定位依赖 **P-S 波到时信息**。
  - 更长的时间窗口可提供更完整的震相信息，提高定位精度。

### **2.3 计算成本分析**
- **计算时间对比**：
  - **50 × 600 模型训练 100 轮需要 5 天**。
  - **50 × 120 模型训练 100 轮仅需 2 天**。

### **3. 多台站任务的计算效率与 SeisCLIP 适用性分析**
SeisCLIP 在 **多台站任务** 中计算效率和适用性成为关键因素。

**计算效率分析**
..
**SeisCLIP 适用性**
- **Fine-Tune 模型可扩展至其他任务**：
  - **极性判断（Polarity Determination）**
  - **事件检测（Event Detection）**
  - **峰值地面运动估计（Peak Ground Motion Estimation）**。
- **泛化能力的挑战**：
  - **不同频谱尺寸的任务可能无法直接适配固定频谱尺寸的预训练模型**，影响泛化能力。
  - 例如：
    - **分类任务适用小频谱（120 采样点）**。
    - **定位任务适用大频谱（600 采样点）**。
**解决方案**
- **发布三种不同频谱尺寸的预训练模型**，提高适应性，使不同任务可选择合适的模型进行微调。
---
# 5.CONCLUSION
**SeisCLIP 作为地震学基础模型**
- **我们提出了一种基础模型（Foundation Model），用于事件分类、地震定位和震源机制分析。**
- **不同于传统深度学习方法，我们的方法先进行预训练，再进行微调，提高泛化能力。**

**预训练 + 微调的优势**
- **预训练阶段**：学习通用的地震信号特征，提高数据利用率。
- **微调阶段**：在不同任务上优化，使模型适应分类、定位、震源机制分析等任务。
- **最终效果**：模型在多个任务中均表现优异，超越基线模型。

**多区域数据集测试**
- **通过在 PNW、SCSN、日本等数据集上测试，验证 SeisCLIP 的泛化能力。**
- **结果表明，SeisCLIP 在多个数据集上表现优于基线模型，证明了其适用性。**

**SeisCLIP 的未来潜力**
- **SeisCLIP 作为通用模型，可扩展到更多地震学任务，如极性判断、事件检测、峰值地面运动估计等。**
- **为下一代地震学深度学习模型的发展提供了蓝图，推动地震学智能分析的发展。**
---
## **附录 A：评估指标（Evaluation Metrics）**

### **1. 二分类任务的评估方法**
对于 **二分类任务（如地震 vs. 爆炸）**，我们采用 **ROC 曲线（Receiver Operating Characteristic Curve）** 和 **AUC（Area Under the Curve）** 作为主要评估指标。

#### **1.1 预测分类方式**
模型输出 **每个类别的概率**（地震或爆炸），通过设定 **阈值（Threshold）**，决定测试集中的事件属于 **地震（Earthquake）** 还是 **爆炸（Explosion）**。计算 **混淆矩阵（Confusion Matrix）** 的关键指标：

- **真正例（$TP$，True Positive）**：正确预测的爆炸事件。
- **真负例（$TN$，True Negative）**：正确预测的地震事件。
- **假正例（$FP$，False Positive）**：将地震错误分类为爆炸。
- **假负例（$FN$，False Negative）**：将爆炸错误分类为地震。

#### **1.2 评估指标计算公式**
- **精确率（Precision）**：

$$
  Precision = \frac{TP}{TP + FP}
  $$

- **召回率（Recall 或 $TPR$，True Positive Rate）**：

$$
  TPR = Recall = \frac{TP}{TP + FN}
  $$
- **假正率（$FPR$，False Positive Rate）**：

$$
  FPR = \frac{FP}{FP + TN}
  $$
- **$F1$ 分数（F1-Score）**：

$$
  F1 = 2 \times \frac{Precision \times Recall}{Precision + Recall}
  $$

#### **1.3 ROC 曲线与 AUC**
通过 **改变阈值**，计算不同点的 **$TPR$ 和 $FPR$**，绘制 **ROC 曲线**。  
**AUC（曲线下面积）** 代表分类器在不同阈值下的整体性能：
- **$AUC$ 值范围：$0 - 1$**，值越大表示分类器性能越优。
- **$AUC = 1$** 表示完美分类，**$AUC = 0.5$** 表示随机分类。

---

### **2. 多分类任务的评估方法**
对于 **多分类任务（如地震、爆炸、地表事件）**，我们采用 **一对多（One-vs-Rest）** 方法，将其转换为多个 **二分类问题**。

#### **2.1 One-vs-Rest 方法**
我们对 **每个类别单独计算 ROC 曲线和 AUC**，具体方式如下：
- 在 **图 2** 的计算过程中：
  - 视 **地震** 为 **正样本（Positive）**，爆炸 & 地表事件为 **负样本（Negative）**。
  - 视 **爆炸** 为 **正样本**，地震 & 地表事件为 **负样本**。
  - 视 **地表事件** 为 **正样本**，地震 & 爆炸为 **负样本**。
- **分别计算 $TP, TN, FP, FN$ 指标**，生成多个二分类 ROC 曲线。

#### **2.2 宏平均（Macro Average）**
我们计算多个类别的 **$AUC$ 值的算术平均值**，作为整体分类性能的衡量指标：

$$
Macro\ Average = \frac{AUC_1 + AUC_2 + AUC_3}{3}
$$
宏平均（Macro Average）能够衡量模型在所有类别上的平均分类能力，确保其不偏向某一特定类别。

---

### **3. 结论**
- **二分类任务** 采用 **ROC 曲线和 AUC 评估模型整体性能**，并计算 **Precision、Recall、F1-Score** 等关键指标。
- **多分类任务** 采用 **One-vs-Rest 方法，将多分类问题转化为多个二分类问题**，分别计算 ROC 曲线和 AUC。
- **使用宏平均（Macro Average）评估多分类任务的整体性能**，确保模型在所有类别上的分类效果均衡。

## **附录 B：训练细节（Training Details）**

### **1. 训练优化策略**
在训练过程中，**SeisCLIP 采用 ADAM 优化方法（ADAM Optimizer）**，并针对 **不同的训练过程** 采用不同的 **损失函数（Loss Function）、学习率（Learning Rate）、训练轮数（Epochs）和批量大小（Batch Size）**（详见 **表 IV**）。

### **2. 训练损失曲线分析**
**图 11** 展示了 **预训练（Pretraining）和各下游任务（Downstream Tasks）** 训练集和验证集的 **损失曲线（Loss Curves）**。  
所有模型均基于 **验证集（Validation Dataset）上的最佳表现** 进行选择：
- **预训练过程中，模型在第 55 轮（55th Epoch）达到最佳性能**：
  - 此时 **损失函数值最低**，模型的 **训练集和验证集精度最高**。
  - 因此，该轮次的模型被选定为 **最终的预训练模型（Final Pre-Trained Model）**。

### **3. SeisCLIP 性能的关键因素**
SeisCLIP 的 **优异表现（Superior Performance）** 主要归因于其 **预训练编码器（Pre-Trained Encoder）** 具有 **大规模参数量（Substantial Parameter Volume）**：
- 由于编码器已经在 **大量数据上预训练**，它可以 **提取丰富的特征**，从而在下游任务中提供更好的泛化能力。
- 在具体的 **下游任务（Downstream Tasks）** 中，**解码器（Decoder）部分的参数量明显减少**，降低计算成本，同时保持高性能。


### **4. 可训练参数对比**
为了评估模型的计算开销，我们对 **不同模型的可训练参数数量（Trainable Parameters）** 进行了比较（详见 **表 V**）。  
该分析有助于理解：
- **SeisCLIP 预训练模型的计算量相对较大**，但**下游任务微调时，可训练参数显著减少**。
- **相比其他基线模型，SeisCLIP 由于预训练的特性，在实际任务中更具计算效率**。

### **结论**
- **SeisCLIP 采用 ADAM 优化，并针对不同任务调整超参数**（损失函数、学习率、批量大小等）。
- **训练曲线表明，模型在 55 轮时达到最佳状态，因此该轮模型被选定为最终预训练模型**。
- **SeisCLIP 预训练编码器的庞大参数量赋予其强大的特征提取能力，使得下游任务可采用更轻量的解码器，提高计算效率**。
- **不同模型的可训练参数对比（表 V）进一步说明，SeisCLIP 预训练模型可在微调阶段减少参数量，同时保持优异性能**。

![Image](https://github.com/user-attachments/assets/002465be-d75d-4c69-9d67-df99e1686dc9)













[Paper Link](https://ieeexplore.ieee.org/document/10400506)
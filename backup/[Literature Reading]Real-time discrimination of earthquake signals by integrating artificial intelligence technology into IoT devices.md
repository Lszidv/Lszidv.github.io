# Abstract
- 本文提出了一种基于进化深度学习方法构建的地震事件分类模型 MCU-Quake，该模型结构极其轻量（仅 2693 个参数），将事件类别判别编码为单一数值 $R(t)$。
- 训练数据来自美国犹他州的原始地震波形，模型在全球自然地震数据上展现出良好泛化能力，并能识别乌克兰战争中的爆炸事件。
- 判别输出 $R(t)$ 的分布如下：背景噪声 $\approx -5.01$，爆炸 $\approx 1.96$，自然地震 $\approx 1.01$。
- 模型设计高度轻量，适合部署于资源受限的 IoT 微控制器中，为就地人工智能传感器的部署提供可行性方案。
---
# Introduction 
 - 传统地震检测方法难以兼顾实时性、抗噪性与低资源部署，尤其在 IoT 场景下受限于 MCU 的存储与算力瓶颈。
- 当前主流方法包括：
  经典机器学习（如 E3WS、2S-ML-EIOS，依赖手工特征与 CPU 并行）；
  深度学习方法（高参数、需 GPU、适用于离线回顾）。
- MCU-Quake 采用进化策略设计，仅需 2693 个参数，可在 IoT 微控制器上运行，利用秒级波形实现地震源实时分类。与如 LEQNet 等主流模型相比，MCU-Quake 推理能耗低 2700 倍，且具备低成本、低延迟与高准确性等优势。
- 该研究推进了 AI 与 IoT 融合（AIoT）在地震早期预警与非法爆破监测等关键任务中的实际应用潜力。
---
# Results
## 模型演化过程与轻量设计
- 通过多目标进化策略（优化模型大小、推理时间与准确率），从256个初始结构中进化出多个高性能模型。Pareto最优集 包含多个非支配结构，其中第5代 ID20 的个体最终被命名为 MCU-Quake。
- MCU-Quake 架构简单，仅由 1D卷积层 + 前馈网络 组成，参数仅 2693 个，内存需求约为 10 KB，可部署于绝大多数微控制器。

![Image](https://github.com/user-attachments/assets/102ca29a-0675-4225-918b-e5180cb2d9d7)

## 多数据集对比验证
- 在 UUSS 数据集上：
  信号源识别精度（爆破/地震）：87.34% / 84.18%，召回率：87.49% / 84.74%
  地震检测精度/召回：99.37% / 98.96%在 STEAD 数据集上：
- 地震检测精度/召回：98.02% / 96.44%
  STA/LTA 与 E3WS 精度受噪声影响显著，LEQNet 性能接近 MCU-Quake，但参数多近 15 倍。
## 资源与能耗优势
- 输入对比：
  MCU-Quake：7 秒 单通道原始波形
  LEQNet：60 秒 三通道滤波波形
  E3WS：10 秒 三通道提取 140 特征
- 参数与内存：MCU-Quake 仅 6.8% 的 LEQNet 参数量，~10 KB 内存
- 成本对比：
  MCU（如 Arduino Nano 33）：约 $6 美元
  GPU部署模型：约 $600 美元
- 推理能耗：
  LEQNet（Raspberry Pi）：334.7 mJ
  MCU-Quake：0.121 mJ
- 只有 STA/LTA 与 MCU-Quake 可部署于 MHz 级 MCU 上，MCU-Quake 在 64MHz 微控器上 推理时间仅 128ms。

![Image](https://github.com/user-attachments/assets/2253d77c-d704-4c9c-992b-a0535a2b65bc)

## MCU-Quake 的判别知识与泛化能力
- 模型学习得到的三类嵌入代表值为：背景噪声：−5.01;爆炸：1.96;自然地震：1.01
- MCU-Quake 的输出嵌入值 $R(t)$ 可视为单一数值，用于与三类嵌入模态值进行距离比较，从而完成信号源判别。
- 密度函数分布比较法：通过概率密度函数（PDF）拟合各类训练集嵌入分布，实现输入值的分类判断。
- 泛化能力分析：虽未在 STEAD（全球数据） 上训练，但其背景噪声与多数地震事件嵌入值仍落入训练集（UUSS）学习的分布区间。对俄罗斯-乌克兰战争信号表现出良好区分能力（爆炸 vs 地震），部分误判样本为爆炸假阴性。
- 波形长度影响分析：判别性能随输入波形时长提升而显著增强。至少 2 秒 波形才能较好分离信号类别，7 秒输入 时表现最优。
- 实时滑动窗口推理能力：支持滚动窗口检测地震波，并准确在后期判断其来源。能在地震波到达后的数秒内准确识别其为地震信号，满足实用预警需求。

![Image](https://github.com/user-attachments/assets/c004b094-06c0-41ae-bd86-dd0717b3c618)

---
# Discussion
## 模型的可解释性与对比分析
- MCU-Quake 使用 秒级原始垂直波形分量 即可进行信号源判别，显著优于同类模型对复杂输入的依赖。以往研究（如 Linville 2019 和 Lara 2023）也表明垂直分量在短时窗内就具有高判别性。
- 与“黑箱”式概率模型（如 LEQNet、EQTransformer）相比，MCU-Quake 输出为实值嵌入，可通过 概率密度函数进行似然判断，具备高度可解释性。
- 使用 对比学习（contrastive learning） 强化不同信号源的嵌入区分，避免人为阈值设定误差。
- 对低信噪比 事件的误判主要集中于爆炸与地震分布的重叠区域，为系统性误差提供合理解释。
## 俄乌战争数据集验证与实地复现能力
- 利用来自 Dando 等人发布的 乌克兰单通道地震阵列数据，MCU-Quake 成功识别出 1276 起地震事件，其中 371 起为爆炸，尽管模型训练时只用美国爆破数据。对 2022 年 3 月 7 日和 3 月 31 日的爆炸活动也能进行部分复现，与实际战争节点对齐。虽存在误检与漏检问题，但主要受信噪比、耦合方式（地面 vs 地下）与传播路径差异影响。在未改动模型结构与参数的前提下，模型展示出良好的跨地区泛化能力，验证其实地部署与长期监测的实用性。
## MCU-Quake 的轻量建模与边缘部署优势
- 传统 RNN/LSTM 模型难以在资源受限设备上部署，原因在于：状态变量多，复杂度高；内存与计算资源需求大；注意力机制进一步提升计算开销。
- MCU-Quake 通过进化型 NAS 搜索出极小型网络结构，使用 CPU 并行训练约 2000 个候选结构，每个训练时间仅约 1.4 分钟。
- 相比于 GPU/TPU 依赖的大模型，MCU-Quake：仅需 ~10 KB 内存；可在 64 MHz Cortex-M4 上实现 ~128ms 推理；在 ESP32 S3 上进一步降低至 ~68ms；推理功耗 0.1–2.3W，适用于 太阳能野外部署。
## 简约建模策略与判别优化设计
- 不使用 LSTM/RNN，不建模长时间依赖关系；
- 基于时间-深度方向的轻量特征融合方法（复杂度 $\mathcal{O}(n + d)$）；
- 相比传统 CNN 时间序列特征提取，推理精度提升 11%。
## 实用性与推广性
- 地震事件出现频率极低（<1%），MCU-Quake 可作为长时间运行的节能触发器，级联调用大型模型；可部署于旧版观测台站、低成本 MCU 设备，提升其智能能力；与 GPS、环境传感器联合使用，构建更广泛的 地球系统感知平台。
---
# Methods
## 数据集构建与预处理
- 使用三类数据集：UUSS（美国本地）：建模与测试；STEAD（全球）：泛化验证；俄乌战争：实战场景验证。所有数据经人工标注，仅使用垂直分量原始波形（未滤波/未校正）。
- 构造方式：每个样本包含 参考/正/负/噪声 四段波形，P 波后 9s 内归一化，100Hz 采样。
- 训练集 / 验证集数量：65065 / 7165 条。

![Image](https://github.com/user-attachments/assets/3b736819-7080-4f69-a9a1-c8757d256d58)

## 进化式微型神经网络搜索（NAS）
- 模型结构由 五个变尺寸卷积核 + 嵌入全连接层 构成，支持最大池化与拼接操作。
- 结构由 11 个变量控制（染色体编码），包括：卷积步长、核是否启用；池化尺寸与步长；三层全连接网络的大小。
- 训练采用 对比学习 + Wasserstein 距离：区分正样本 / 负样本 / 背景噪声；
- 目标：最大化嵌入间差异。
-演化过程：初始种群 256；最大 8 代；保留适应度最优的前16；使用 多核 CPU 云平台并行训练（Python + DEAP + TensorFlow）。

![Image](https://github.com/user-attachments/assets/c1db4bd5-a7ee-49b0-8eb2-a323b1e4b30e)

## MCU 实时部署验证
- 实验中使用 两种 MCU 连接 TFT 显示器，每秒接收波形窗口，实时输出信号源类别概率。
- 推理时间通过串口反馈，支持低延迟、可视化显示与低功耗运行。

[Paper Link](https://www.nature.com/articles/s43247-025-02003-y)
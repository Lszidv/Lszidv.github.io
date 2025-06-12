# Abstract
- 提出了一种基于深度神经网络的滤波模型 FilterNet，结合了 CNN 与 RNN 架构；
- 实现了原始隧道衬砌 GPR 数据的端到端滤波处理，包括去除空气反射波、去噪与自动增益；
- 性能方面，FilterNet 的 SSIM 达到 0.997，PSNR 高达 19.41，参数量为 UNet 的 48%，GFLOPS 仅为其 1/3；
- 支持实时处理，在抑制随机噪声方面具有显著优势，是一种高效、轻量、实用的GPR数据处理方案。
---
# 1. Introduction
- 地质雷达（GPR）因其高效、无损、高分辨率的特点，在铁路路基等浅层结构检测中应用广泛；
- GPR 数据采集过程易受噪声与电磁干扰影响，影响信噪比与分析效果；
- 传统滤波算法（傅里叶、小波、曲波等）与浅层机器学习方法（EMD、VMD、FastICA）虽具一定效果，但难以实现复杂数据的端到端处理；
- 深度神经网络（DNN）滤波方法（如 UNet、GPRNet、卷积自编码器）在信噪提升与特征提取方面表现优越，但存在参数量大、处理速度慢等问题；
- 为实现更高效的 GPR 数据处理，本文提出一种融合 CNN 与 RNN 的深度滤波模型 FilterNet，具备小参数量、高实时性的优势，适用于实际工程场景中对原始 GPR 数据的快速处理。
---
# 2. Method
## 2.1 The architecture of FilterNet
- FilterNet 结构结合 CNN 与 GRU（RNN 子类），用于 GPR B-Scan 数据端到端滤波,模型设计目标：低参数量、高效率、支持实时性、小样本可训练性；
- FilterNet 架构包含：
    Encoder： 基于多层 1D CNN，输入为长度为 512 的一维通道波形；
    GRU 层： 提取时序相关性，支持处理二维剖面；
    Decoder： 将波形特征向量还原为滤波结果；
- FilterNet1D 为对照模型，移除 RNN 并引入 skip connection，用于处理单通道数据,两种模型的输出均通过 Sigmoid 函数归一化；
- FilterNet 具有更短特征向量与更少参数量，适合实际工程中的实时场景；FilterNet1D 收敛速度更快。

![Image](https://github.com/user-attachments/assets/26a20c43-9bec-42ac-83f5-d519168e8804)
## 2.2. Training data
- 为提升模型的实用性与泛化能力，不使用合成数据，而是采用实际采集的 GPR 隧道衬砌检测数据进行训练与测试。
- 数据来源： 采集自宜漳铁路宜娄段共 15 座隧道（如刘家隧道、苏油冲隧道），覆盖湖南省益阳、长沙、湘潭、娄底等区域。线路北起益阳东站，南至娄底东站，地理坐标范围大致为 北纬 27°40′ 至 28°30′，东经 111°50′ 至 112°20′。
- 模型目标： FilterNet 实现 端到端处理，直接以原始波形为输入，输出已完成 去除空气反射波、噪声抑制和自动增益处理的滤波结果（见图 2）。
- 预处理步骤： 模型输入数据仅进行归一化处理，不采用其他手动预处理操作。
- 标签数据构建： 滤波后的标签波形由 GPR 专业后处理软件 Reflexw 生成。该软件广泛用于 GPR 数据处理，支持数据导入、增益调整、滤波与编辑等功能。
- 优势： 通过使用原始波形数据与专业标注输出构建训练样本，可大幅简化后处理流程，提高 GPR 数据处理的自动化程度与工程可部署性。
##  2.3 Real-time inference process 
![Image](https://github.com/user-attachments/assets/533d85de-cad6-4133-8cf9-d4f055428f2a)
- FilterNet1D：用于处理单通道一维波形，结构简洁，只需实时输入当前通道数据即可完成滤波；
- FilterNet：用于处理二维 B-Scan 数据，依赖状态信息传递与序列建模，能捕捉通道之间的上下文信息。
- 推理步骤如下：
    1. 归一化输入
    2. 卷积编码器提取特征向量
    3. GRU 接收前一状态与当前特征，输出当前状态向量
    4. 解码器生成滤波输出波形
- 状态向量 $mathbf{h}_i$具有记忆性，融合了当前通道与历史通道的信息，实现了对 B-Scan 数据的二维建模。整个流程支持 端到端、实时滤波输出，适用于隧道衬砌检测等现场实时 GPR 处理任务。
---
# 3. Performance analysis of FilterNet
## 3.1 Single data analysis

<!-- Failed to upload "image.png" -->
- 测试集：长度为 2048 行 的 GPR 波形数据，每行包含 512 点的一维信号；
- 原始数据（图 4a） 能量集中于空气反射波，地下回波几乎被掩盖；
- 标签数据（图 4b） 仍存在随机噪声，且反射波比模拟数据更强；
- FilterNet 输出（图 4c, 4d）：能有效滤除标签中残留的随机高频噪声；网络输出趋于平均，天然抑制正负对称的高频随机扰动；网络不仅保留反射信号，还显著增强其可辨识性；
- 模型差异对比：
    1D 模型：噪声残留主要为横向结构；
    2D 模型：更好建模通道间上下文信息，表现出更强的噪声抑制能力；
📌 神经网络在保留目标反射波的同时抑制随机噪声，表现出超越传统滤波算法的优势。

## 3.2. The accuracy of FilterNet
- 比较了三个模型的滤波精度：FilterNet1D；FilterNet；UNet（Liu et al., 2024）
-评估指标包括：峰值信噪比（PSNR, Peak Signal-to-Noise Ratio）；结构相似性指数（SSIM, Structural Similarity Index）

$$
\mathrm{PSNR} = 20 \cdot \log_{10} \left( \frac{I_{\max}}{\mathrm{RMSE}(x, d)} \right)
$$

$x$ 表示模型输出波形；
$d$ 表示标签波形；
$\mathrm{RMSE}(x, d)$ 为均方根误差；
$I_{\max}$ 为最大值（通常归一化为 1）；
- PSNR 值越高，表示滤波后信噪比越高

$$
\mathrm{SSIM}(x, d) = \frac{(2 \mu_x \mu_d + C_1)(2 \sigma_{xd} + C_2)}{(\mu_x^2 + \mu_d^2 + C_1)(\sigma_x^2 + \sigma_d^2 + C_2)}
$$

$\mu_x, \mu_d$ 为均值；
$\sigma_x^2, \sigma_d^2$ 为方差；
$\sigma_{xd}$ 为协方差；
$C_1, C_2$ 为常数项（用于稳定计算）；
- SSIM 值越接近 1，说明结构越相似。

## 3.3. Computation Efficiency Test
- 评估了模型的 理论计算复杂度（以 GFLOPS 表示）和 可训练参数数量；
- FilterNet 推理仅需 3.21 GFLOPS，为 UNet 的 1/3，更适合资源受限设备；
- 模型参数量方面，FilterNet 仅为 107.1K，而 UNet 高达 222.2K，多出 115.1K 参数；
- FilterNet 每次只需处理一帧数据，相比 UNet 同时处理多帧的机制，显著节省内存；
- 由于 RNN 不易并行，FilterNet 在多核 CPU 上处理速度略慢，但在低核数处理器（如 ARM）上仍具优势。

## 3.4. Model Generalization Ability Test
- FilterNet 作为 $ \text{DNN} $ 方法，依赖训练数据，对相似数据具有高精度，但对异构数据泛化性较弱；
- 使用两类数据测试模型泛化能力：
    1.香山隧道 GPR 实测数据（背景噪声更强）；
    2.合成管道反射数据（加入高斯噪声）；
- 相比之下，UNet 与 FilterNet 拟合能力更强但泛化能力更弱；
- 测试表明，若要适应更多任务，需使用多样化数据进行 迁移学习 训练提升泛化能力
---
# 4. Disscussion
![Image](https://github.com/user-attachments/assets/6cf402f7-5f3e-42dc-b802-bf4dca80c7f6)
- 深度学习模型可直接处理原始 GPR 数据，替代传统的直达波去除、滤波与增益预处理流程；
- FilterNet 可有效抑制深部高频噪声，但：1D 模型对横向噪声抑制较弱；2D 模型能更好学习横向纹理，具备结构性滤波能力；
- 为测试泛化能力，引入两种人工噪声：类直达波背景噪声；幅值为 1% 的高斯噪声；
测试结果见图 8：UNet 在类直达波噪声下完全失效；FilterNet 输出弱，但有信号；FilterNet1D 表现最好，可正常输出并抑制部分干扰；
- 对于高斯噪声：所有模型性能下降；UNet 过拟合严重，泛化性最弱；FilterNet 因含 RNN，具时序建模能力，表现次优；FilterNet1D 虽为 CNN，但单通道信息密度高，泛化能力最强。
---
# 5. Conclusion
- 提出了一种可直接处理原始 GPR 数据的滤波模型 —— FilterNet，包含两种结构：
    FilterNet1D：基于 CNN，处理单通道数据；
    FilterNet：结合 CNN 和 RNN，可建模多通道数据间的时序信息。
- 该模型具备以下三项优势：
   1.端到端输出，无需传统的直达波去除、滤波、增益调整等流程，提升自动化效率；
    2.更强的噪声抑制能力与泛化能力，可在不同噪声环境下稳定输出，效果接近人工处理；
   3.实时性强、计算资源占用低，适合现场嵌入式部署或边缘计算平台。
[Paper Link](https://www.sciencedirect.com/science/article/pii/S277246702500017X#fig1)
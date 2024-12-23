# MFAST

## Abstract
自动化流程：仅需人工选择S波到达时间，其他步骤完全自动化。
核心算法：基于特征值最小化、多时间窗口优化，以及聚类分析确定最佳解。
质量控制：通过质量分级标准自动筛选结果，减少了人工干预的主观性。(可能会保留部分低质量测量值)

高质量数据：在高质量数据集上，自动技术与人工分析结果高度一致。
低质量数据：在低质量数据集上，自动技术能够提取出新信息，可能比人工方法更全面。

## 1. Introduction

地震各向异性是研究地球内部方向性特征的重要工具，常通过剪切波分裂（SWS）来进行分析。
剪切波分裂的两个关键参数（ϕ：快波偏振方向，通常与最大水平应力方向平行。δt：快慢波时间差，与裂隙密度成正比。）
SC91网格搜索技术是经典的剪切波分裂分析方法，通过特征值最小化优化分裂参数。

```
局限性：
依赖滤波器和时间窗口：SC91算法高度依赖滤波器的选择和时间窗口的设置，而这些参数可能因数据特性变化而显著影响测量结果。
结果的不确定性：不同时间窗口或滤波器的选择可能导致剪切波分裂参数（ϕ 和 δt）的不一致性。
数据剔除率高：在许多应用中，算法会剔除大部分数据，这表明其对数据质量要求较高，可能忽略一些具有潜在信息的数据。

Teanby等[2004a]结合SC91方法与聚类分析，通过误差最小和支持度最大的标准，确定最佳时间窗口。
成功应用于北海数据集，验证了局部地震事件时间变化和远震事件静态特性的适用性。
尽管仍需手动分级剔除低质量数据，但为剪切波分裂分析提供了新的思路

改进后的全自动方法
滤波器优化：首次结合多滤波器分析，扩大了适用数据范围，增强了对不同频率地震波的适应性。
频率特性分析：根据地震波形频率动态调整时间窗口长度，使测量更加灵活。
自动分级：基于聚类特性，而非简单选择最佳聚类，自动评估测量结果的可靠性。
成功应用于日本、阿拉斯加、蒙特塞拉特等火山地震数据，揭示了与岩浆活动相关的各向异性时间变化。
```
## 2. Method 

SC91基础分析：
在多个时间窗口上执行SC91算法。
计算每个窗口的分裂参数（ϕ和δt），并统计误差范围。
聚类分析：
基于时间窗口的测量结果进行聚类分析。
选择方差最小的聚类作为最佳聚类，并进一步从中挑选误差最小的时间窗口作为最终测量结果。

方法改进
自动选择时间窗口范围：根据地震事件的周期特性动态选择时间窗口范围，避免了人为设置的不灵活性。
多滤波器优化：利用信噪比与滤波器带宽的乘积评估信号质量，确保适用不同频率特性的地震信号。

**2.1 滤波**
```
地震数据常因噪声污染而难以直接处理，信号增强和噪声抑制成为剪切波分裂测量的关键。
滤波器的选择直接影响信噪比，进而影响分裂参数（快波偏振方向 
ϕ 和时间延迟 δt）的测量质量。

提出了14个预定义的带通滤波器组（表1），针对每个测量自动测试滤波器性能，选择信噪比与带宽乘积最佳的滤波器。
自动化的滤波器选择方法替代了统一应用宽带滤波器或人工选择滤波器的传统方法

多滤波器方法的优越性:
避免了窄带滤波器可能导致的循环跳跃问题，同时保留了信号的完整性。
多滤波器方法适用于局部地震波（如S波）和远震波（如SKS波），拓展了方法的应用范围：
局部地震波：用于微地震研究和局部剪切波分裂分析。
远震波：适配SKS波，研究地幔各向异性。
```
**2.2 信噪比计算**
```
1.计算方法
基于快速傅里叶变换（FFT），计算信号窗口与噪声窗口幅值的比值，并取其平均值作为信噪比。
信号窗口选择在S波到达之后，噪声窗口选择在S波到达之前，包含P波尾波噪声对测量的干扰。
设置信噪比最低门槛（如3），剔除噪声干扰过大的测量值，提升数据质量。

2. 多滤波器优化策略
对符合信噪比要求的波形，选择带宽（以倍频程表示）与信噪比乘积最高的三种滤波器进行分析。
通过多个滤波器处理和分析波形，能够有效检查剪切波分裂参数的频率依赖性，筛选最稳定的测量值。
```
**2.3 基本分裂技术**
&emsp;核心测量方法
采用反分裂算子通过网格搜索确定剪切波分裂参数（ϕ,δt）。
参数优化目标是最小化协方差矩阵的第二特征值 𝜆2 ，实现粒子运动的最大线性化。
 &emsp;时间窗口选择的自动化
通过主频 𝑓𝑑动态调整窗口长度：
最小窗口为1个周期，最大窗口为2.5个周期。
自动生成自定义配置文件，每个测量具有独立的窗口设置。
&emsp;聚类分析与参数优化
聚类分析在多个窗口配置下搜索所有测量值：总方差最小的聚类为最佳聚类 ，聚类中方差最小的测量值作为最终结果。
![SC91 流程](https://github.com/user-attachments/assets/56d08650-7ad0-4ff6-9aed-54bda37a65b1)
(a) ENZ方向的滤波波形  (b) 波形旋转后的结果[根据SC91算法旋转至快波偏振方向（𝑝）和垂直于快波的方向（𝑝⊥）]
(c) 随窗口变化的分裂参数[不同测量窗口下快波偏振方向(ϕ)和时间延迟(δt)的变化。每个窗口的误差条显示了参数的不确定性。]
(d) 聚类结果[聚类分析结果，展示所有窗口测量值在 𝜙-δt 参数空间中的分布。选择误差最小的聚类作为最终结果(大交叉点,图c)]
(e) 粒子运动轨迹[原始波形(左)和校正后波形(右)的粒子运动轨迹。校正后波形的轨迹呈线性，表明剪切波分裂已成功校正。]
(f) 协方差矩阵的特征值等值线[参数空间中协方差矩阵第二特征值(𝜆2​)的等值线。等值线最小值对应于最优分裂参数(灰色框标示)]

**2.4 评估标准**
问题：
```
循环跳跃：快波和慢波的匹配出现了整数个半周期的偏差，导致分裂参数失真。[滤波器过窄或错误匹配]
多解问题：不同时间窗口可能产生多个参数解，难以确定唯一的结果。[信号质量不稳定或存在噪声干扰。
分裂现象本身的复杂性（如局地应力场变化）]
```
自动评分方法：
```
(1) 基于聚类分析的自动评分
方差var(k)： 每个聚类内的分裂参数稳定性，方差越小，聚类质量越高。
快波偏振方向（ϕ(k)）： 快波的方向，用于判断分裂特性。
延迟时间（δt(k)）： 慢波相对快波的到达时间延迟，反映分裂强度。
测量数量（Nmeas(k)）： 每个聚类内的测量值数量，数量越多表明聚类可信度越高。

方差最小的聚类（𝑘best ）定义为最佳聚类。
其他方差小于5⋅var(k best) 的“合格聚类”用于辅助验证最佳聚类结果。

(2) 过滤、剔除
某些次级聚类与最佳聚类的分裂参数（快波方向𝜙和延迟时间𝛿𝑡）差异过大，剔除。
零分裂数据是指由于没有显著各向异性或初始波形与介质方向一致，剪切波未发生分裂。
（如果 ∣ϕ−a∣ 不满足 20°≤∣𝜙−𝑎∣≤70°，则判定为零分裂。）
![0测量](https://github.com/user-attachments/assets/c2470110-f881-40ca-af9f-a53f75a58a7a)

局地事件的延迟时间一般在 0.1 秒至 0.6 秒之间，异常值可能由循环跳跃或噪声引起。
设置最大延迟时间𝑡lagmax=1.0s，并剔除所有 δt>0.8⋅tlagmax的测量。
```
综合评分准则：
```
(1) 基于聚类的评分
比较所有“合格聚类”与最佳聚类的参数差异。
如果次级聚类的参数差异较大，则判定为低质量解并剔除。
(2) 基于等值线的评分
特征值等值线范围越小，表明最佳解与其他解的差异越小，可信度越低。
使用误差等值线的最大值作为评分参数，剔除误差过大的测量。
改进点：
不仅关注等值线的最小值区域，还考虑整个误差分布，提高了评分的全面性。
```
**2.5 均值**
```
在多次地震事件中测量同一各向异性的剪切波分裂参数时，需要计算参数平均值，以描述整体的各向异性特征。

1.延迟时间的统计处理：使用传统的高斯分布统计方法，适合延迟时间这种连续变量。

2.快波偏振方向的处理：由于偏振方向是周期性变量（角度0°到360°），采用Von Mises分布，这是一种适合方向性数据的统计模型。（通过单位矢量法处理）：
    将每个偏振方向转换为单位矢量。
    对所有矢量进行求和并归一化，得到均值方向。
    解决180°不确定性的方法是将角度加倍后相加，最后减半。

3.一致性度量：
通过结果矢量的长度 R（0到1之间）量化方向分布的一致性：
    R=1：完全一致，所有偏振方向对齐。
    R=0：完全随机，无显著方向性。
```
示例：快波偏振方向的处理

##### **假设数据**
给定 4 个快波偏振方向（单位：度）：

$\phi = [30^\circ, 32^\circ, 35^\circ, 210^\circ]$

##### **步骤 1：转换为单位矢量**
每个方向对应的单位矢量：

- $\mathbf{v}_1 = (\cos(30^\circ), \sin(30^\circ)) = (0.866, 0.500)$
- $\mathbf{v}_2 = (\cos(32^\circ), \sin(32^\circ)) = (0.848, 0.530)$
- $\mathbf{v}_3 = (\cos(35^\circ), \sin(35^\circ)) = (0.819, 0.574)$
- $\mathbf{v}_4 = (\cos(210^\circ), \sin(210^\circ)) = (-0.866, -0.500)$

##### **步骤 2：矢量求和**
将这些单位矢量相加：

$$
\mathbf{R} = \sum \mathbf{v}_i = (0.866 + 0.848 + 0.819 - 0.866, 0.500 + 0.530 + 0.574 - 0.500)
$$
$$
\mathbf{R} = (1.667, 1.104)
$$

##### **步骤 3：计算均值方向**
计算矢量的方向（均值偏振方向）：

$$
\phi_{\text{mean}} = \arctan\left(\frac{1.104}{1.667}\right) = \arctan(0.662) \approx 33^\circ
$$

##### **步骤 4：解决 180° 不确定性**
为解决对称性问题，将角度加倍再求平均：

$$
\phi' = [60^\circ, 64^\circ, 70^\circ, 420^\circ]
$$
求平均方向后，再将结果减半：
$$
\phi_{\text{mean}} = \frac{1}{2} \cdot \text{mean}(\phi') = \frac{1}{2} \cdot 153.5^\circ = 33^\circ
$$

##### **步骤 5：一致性度量**
计算矢量长度 $R$：

$$
R = \sqrt{(1.667)^2 + (1.104)^2} = \sqrt{2.779 + 1.219} = \sqrt{3.998} \approx 2.0
$$

将其归一化到 [0, 1]。

---

##### **解释结果**
- **均值偏振方向：** $33^\circ$，反映了所有快波偏振方向的集中趋势。
- **一致性度量：** $R \approx 0.8$，表明偏振方向具有较高一致性，但仍存在一定分散。

**2.6 使用多滤波器**
##### 背景
```
优点：
不同滤波器可以捕捉信号的不同频率特征，有助于提高剪切波分裂参数的鲁棒性。
如果不同滤波器对同一事件的结果一致，可以增加结果权重，提升统计可信度。

问题：
不同滤波器的结果并非完全独立，可能存在相关性（例如信号特征重叠），导致高斯统计假设失效。
不同滤波器的参数范围和性能差异使结果比较变得复杂。
```
##### 评分方法改进
```
时间差判断： 最大时间差不得超过允许值的1/8，以确保时间延迟结果的稳定性。
角度差判断： 偏振方向的角度差不得超过π/8（22.5°），确保快波方向结果的一致性。
误差条选择： 在结果相似的情况下，优先选择误差条最小的测量结果。
```
## 3.Data
1. 方法的应用与适用性
提出的方法适用于任何三分量地震数据，只要 S 波信号清晰并高于噪声水平。
多窗口聚类分析减轻了 S 波到达时间误差对测量的影响，增强了方法的鲁棒性。
2. 开发与测试
方法的开发基于新西兰 Ruapehu 火山和日本浅间火山的高质量数据，通过多次迭代和优化选择最佳参数。
在不同火山环境中的应用和评分技术开发进一步提升了结果的稳定性。
3. 数据来源与对比
测试数据包括 1994 年、1998 年和 2002 年的临时台站记录。
自动化方法与手动测量进行了全面对比，重点分析浅层和深层事件的分裂特性。

## 4. Results
4.1 与人工的对比
```
自动化方法依赖于聚类分析选择最佳测量窗口，大幅减少了手动干预，提高了效率和客观性。
手动方法则需人工测试多个时间窗口，结果可能受主观偏差影响，耗时较长。

在不应用质量准则的情况下，75% 的结果在偏振方向 𝜙 差异 25° 以内，85% 的结果在时间延迟 𝛿𝑡差异 0.2 秒以内，表现出较高的一致性。
质量标准的逐步应用（如 AB 等级或 Eng8 准则）进一步提高了测量结果的精度和稳定性。
Table 2 描述了一系列评分准则，从基础的 Null 判断到高质量的 A、B 等级，并包括基于能量等值线比的 Eng8 准则。
更高的质量标准（如 A 和 Eng8 准则）显著提高了测量结果的可靠性，但导致数据覆盖范围大幅缩小。例如，Eng8 筛选后数据量从 212 条减少到 75 条。

自动化程序基于信噪比与带宽乘积准则选择滤波器，减少了手动干预对结果的影响。
当自动选择滤波器时，异常值数量减少，评分准则间的差异减小，显示出自动化方法对结果一致性的增强作用。

单一滤波器分析减少了计算复杂度，但可能会略微降低结果的鲁棒性。
多滤波器分析尽管计算量增加，但能更全面地反映数据特性，尤其是提高了偏振方向和时间延迟结果的稳定性。

从全数据集（包含零分裂测量）进行分析，进一步验证了自动化方法的适用性。
使用 ABeng8 准则筛选后，自动化结果与历史手动方法在偏振方向分布上高度一致。
```
![Mfast 质量准则](https://github.com/user-attachments/assets/95a80574-a798-4506-80f4-0925356001e6)

4.2 平均值的比较
```
1.自动与手动结果的一致性:
自动化方法的结果整体一致性（R 值）略低于手动方法，表明自动化方法在某些情况下对数据分布的捕捉不如手动方法精确。
时间延迟（δt）的均值在手动和自动方法间的差异小于 0.04 秒，大部分情况下处于统计置信区间内。
2. 时间延迟与深度的关系:
深层组的时间延迟显著高于浅层组，分别约为 0.25 秒和 0.14 秒。
这表明地震波在深层传播过程中受到的剪切波分裂效应更强，与地壳结构特性有关。
3. 分组数据的分散性:
1998 年浅层事件的测量结果最分散（R 值最小），可能受到数据质量、信号噪声或事件位置特征的影响。
4. 年份和深度的差异
1998 年和 2002 年的自动化结果与手动结果一致性较高，均在 95% 置信区间内。
1994 年深层事件的自动化结果在偏振方向上偏向北，与手动结果存在显著差异，表明自动方法在该年份和深度组上的适用性有限。
5. 台站的个体差异
2002 年数据集中的 LHUT2 和 FWVZ 台站的深层事件显示了手动与自动方法的最大差异，提示特定台站可能存在数据采集或传播路径的特殊性。

```

## 5. Discussion
5.1 测量值比较：散点图
```
1. 数据分散性与物理过程
剪切波分裂测量的时间延迟分散性高达 80%，与本地事件中复杂的地震波传播路径有关。
可能的物理机制包括：
传播路径中强速度对比界面的相位散射（如岩浆囊）。
波路径的轻微变化对分裂参数的影响。
2. 自动化方法的表现
自动化方法在选择最佳滤波器时结果分散性略高，但平均值与手动方法一致，且异常值更少。
结果一致性表明自动化方法对周期性错误（如循环跳跃）具有较好的鲁棒性。
3. 筛选与质量控制
对三种最佳滤波器进行测量并比较结果，可能是剔除低质量数据的有效方法。
自动化方法通过提前剔除低质量测量值，减少了最终结果中的误差。

结合自动化和手动分析的结果，能够更准确地揭示地震波分裂的物理机制，为区域应力场研究提供可靠依据。
```
![7](https://github.com/user-attachments/assets/5cacfc69-eea9-4fec-9563-dee4d10e21f7)
![8](https://github.com/user-attachments/assets/ad320a1a-c6e4-4401-bc71-fab7f5603a72)
![9](https://github.com/user-attachments/assets/53cfd87c-194e-40a3-b2d4-e1bb2bc56069)
自动化方法在严格质量准则下，与手动测量的一致性显著提高。
质量准则（如 AB, A, AB eng8）能有效减少异常值，但筛选越严格，数据覆盖范围越小。

5.2 个别台站比较：玫瑰图
```
一致性与分散性
高质量波形一致性：
自动化方法在分析已被手动方法判定为高质量的波形时，与手动方法结果一致性较高。
快波偏振角（ϕ）和延迟时间（𝛿𝑡）的散布范围在自动与手动方法之间基本吻合，表现出良好的相似性。

全数据处理的分散性差异：
当所有数据被纳入处理时，自动化方法在某些测站表现出更大的分散性，特别是在偏振角（𝜙）分布上。
某些测站（如 LHOR 和 FWVZ 的浅层事件）显示自动化方法的结果比手动方法更集中，但在其他测站（如 LHUT2 的深层事件）中，自动化方法的分布更为分散，甚至出现双峰现象。

通过假设验证发现：
零分裂剔除率（假设 1）对测量结果差异的影响较小。
手动分析在剔除无效数据方面的高准确性（假设 2）和人为偏向（假设 3）共同导致手动方法结果更集中。
假设 2 --手动分析的技术优势（准确识别无效数据）；假设 3 --人为主观偏向可能进一步缩小结果的分散性。
手动方法可能因历史经验倾向于剔除与预期不一致的结果，而自动方法无此偏向，导致分布更分散。
```
![台站对比_玫瑰图](https://github.com/user-attachments/assets/3e0001a4-fe52-48ad-ab6a-3c68e311f114)
![所有台站平均值对比](https://github.com/user-attachments/assets/d9b25048-e67c-4407-8f72-501b31826be5)
![具体台站对比](https://github.com/user-attachments/assets/e5ba5873-21e7-43a6-b5ad-5c50fcde6fdf)
自动方法虽然在分散性上略逊一筹，但通过多滤波器评分的优化，展现了很高的潜力，尤其是在复杂环境和大规模测量中。
手动方法则继续在高质量数据的细致分析中占据优势。

5.3 平均结果和时间变化
```
自动与手动方法在2002年浅层事件的平均偏差小于10°，表明两者具有较高一致性。
1998年浅层事件的自动与手动方法结果偏差较大（52°），这与数据本身的分散性特性有关，而非方法缺陷。

自动方法确认了1994年与2002年深层事件之间快波偏振方向的显著差异（59 ± 19），表明时间上的地震各向异性变化。
Gerst和Savage（2004）发现的2002年浅层与深层事件之间的显著差异（37 ± 12）在自动方法中得到了部分验证。
火山山顶测站散射：

结合时间和空间变化分析，自动方法可为火山活动的预测与监测提供更可靠的依据（2002年中测站分布的散射性最明显的位置集中在鲁阿佩胡火山的山顶附近。）
```
5.4  建议
```
方法适用性：
该方法适用于各种三分量地震数据集，无论是火山区域还是非火山区域。
已经在多个火山（如浅间山、奥克莫克火山）和非火山区域成功测试，显示了良好的适应性和稳定性。
通过调整滤波器，该方法还能扩展至远震事件的各向异性分析。

实时与非实时分析建议：
对于非实时分析，可以选择多个滤波器以获得更多测量，但需要更多计算时间。
实时分析中，为了提高效率，可以使用单一最佳滤波器（如“fb1”），减少处理时间，同时保持测量数量的合理性。
更严格的能量标准（如“能量 >8”）能减少异常值，但会显著降低测量数量，适合在数据量大的情况下使用。

区域特定调整：
根据研究区域的特性调整滤波器，以适应不同的地质环境和数据特点。
```
## 6. Conclusions
```
自动化与手动方法的优劣势：
自动化技术在相同高质量数据集上的表现与手动技术高度一致，说明其可靠性。
手动评分在排除不适合波形拟合方面具有优势，但可能因无意识偏见导致低估某些数据质量。

自动化方法的适用性：
自动化技术对大规模数据处理尤为适用，尤其是需要快速分析或客观评估时。
在质量较低的数据上，自动化方法的分散性(离散，变异)较高，但仍能为研究提供有效的参考。

科学发现与验证：
自动化技术有效验证了1994年和2002年鲁阿佩胡火山深部事件之间的快波偏振方向变化。
进一步确认了浅部和深部事件在各向异性特征上的显著差异，展现了自动化方法在时空变化研究中的重要价值。
```
[Paper Link](https://agupubs.onlinelibrary.wiley.com/doi/10.1029/2010JB007722)


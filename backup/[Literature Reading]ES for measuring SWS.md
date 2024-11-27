## Abstract
  专家系统和模块化设计

## 1.Introduction
 ```
人工神经网络（Artificial Neural Networks, 1995）：
训练神经网络识别剪切波分裂，能够处理复杂的波形。
局限性：由于剪切波波形的复杂性，这些方法只能部分成功。

相关函数（Correlation Functions, 1995 & 1998）：
利用信号的相关性计算剪切波的极化方向和时间延迟
局限性：剪切波的复杂波形使得这种方法不能完全适用。

自动化第一次到达拾取（Murat and Rudman, 1992）：
提出了成功的自动化技术，专门用于第一次到达的拾取。
局限性：只能处理第一次到达，无法适用于剪切波分裂等复杂波形。

正交变换估计（Orthogonal Transformations, Shieh, 1997）：
通过正交变换进行剪切波分裂的估计。
局限性：受限于交叉相关技术的局限性，可能不适用于所有情形。

集群分析和最优窗口法（Cluster Analysis and Optimum Window, Teanby et al., 2004）：
通过集群分析对剪切波数据进行分类，并优化时间窗口以自动识别剪切波分裂。
局限性：为了确保准确性，舍弃了大量数据，可能导致信息丢失。
```
  ES专家系统，第一阶段，输入三分量数据，得到初步结果（分裂参数）；第二阶段，调整计算，使结果趋于稳定；最终阶段，系统对结果进行评估。

## 2. Initial identification of shear-wave splitting
  考虑剪切波在水平面上的二维运动，定义两个基本量：
```
nmax:剪切波到达前(BTW)噪声的最大振幅
smax:剪切波到达后(ATW)信号的最大振幅

对每个地震，定义阈值参数veq = max(cbef * nmax, smax / caft)
根据经验，对不同的台站有不同的系数Cbef和Caft
```

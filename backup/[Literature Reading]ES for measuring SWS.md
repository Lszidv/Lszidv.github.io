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
2.1 初始规则：快波极化方向的近似
  考虑剪切波在水平面上的二维运动，定义两个基本量：
```
nmax:剪切波到达前(BTW)噪声的最大振幅
smax:剪切波到达后(ATW)信号的最大振幅

对每个地震，定义阈值参数veq = max(Cbef * nmax, smax / Caft),根据经验，对不同的台站有不同的系数Cbef和Caft

信号强度超过veq，认为剪切波到达
定义B点为信号首次超过Veq的点，A点为B点前，最接近BTW窗口内平均值的点
快剪切波极化方向近似值j0为A→B的方向
```
2.2 初始规则：快慢波到达的近似
```
将水平分量旋转，与j0平行或正交
在BTW和ATW窗口内,对每个半周期进行编号（Kbef,Kaft）与幅值测量（a）
Abef和Aaft分别为BTW和ATW窗口内最大值
```
2.2.1 快慢波近似到达的ES规则
```
当幅值a超过阈值H时，认为快剪切波到达（初始为H0, H0 = ci*Aaft  , 0<ci<1 , i=1,2 ，分别代表快、慢剪切波  ，不同台站ci不同）
```
2.2.2 快慢波到达评估的ES规则
```
更新阈值的规则
γ = Ht/Abef  （Ht是当前的阈值，Ht＋1是下一次更新后的）
定义四个常数γ1，γ2（0<γ1<γ2），γm，γt
（1）如果γ<γ1，Ht＋1  = γ1*Abef ； 如果γ >= γ2 , Ht＋1 = γ2 * (Abef + Z * η*Ht)  (0 < η ≤ 1)
（2）如果Ht>Aaft , Ht = γm*Aaft (0 < γm< 1)
（3）如果 awj ≥ Ht（w = bef 或 w = aft；j = 1, 2, k），并且 aw(j-1) ≥ γt * Ht  (0 < γt ≤ 1) ，并且阈值已经更新
    快剪切波的初始到达发生在半周期 j-1 内，慢剪切波也是一样的规则（γt,γm通常大于1.0）
精确的到时估计：
（4）定义数据点i在第j个半周期的绝对值Xj,i ,如果awj > Ht ， 并且xj,1 ≥ Cspe * nmax；(cspe > 1)
    [当前幅度 awi 大于阈值 Ht，并且第一个半周期的幅度 xj,1 大于系数 Cspe 与 nmax 的乘积，就更新快剪切波的到达时刻为最后一个周期j-1, 慢剪切波规则类似]
（5）如果xj,1 ≤ nmax ，前移到点i ,该点 xj,i ≤ nmax，但 xj,i+1 ≥ nmax ,定义该点为C ，定义窗口NBTW ,其与BTW一样长，但是以C作为结束点
    分析NBTW 开始到点 C 的每个点的极化方向，确定最终剪切波到时点
```
$p = \sum_{i=start}^{end} \text{Zamp}(i) \cdot f(β \cdot \left| φ(i) - φ0 \right|)$
```
start 是分析窗口的起始点（窗口NBTW 中的第一个数据点） , end是点C
Zamp(i) 是连续两个点绝对值差
β是极化方向的误差容忍度，保持在±2.5°
φ(i)是每个点振动方向，φ(0)是快剪切波极化方向的初始值
具有最大p值点定义为D，即为快剪切波的到达时刻，慢剪切波具有类似的规则
```
## 3. Accurate identification of shear-wave splitting
  第二阶段，重复迭代，至结果趋于稳定
```
（1）如果慢波到时晚于快波到时，继续执行；不然，则将两个水平分量旋转90°（矫正极化方向），若重新计算后，慢波到时仍晚于快波到时，则拒绝此数据
（2）任何评估步骤中，如果信噪比低于某个值（例如Crsn2 = 3.0），则认为数据质量不够，进行排除
（3）ES规则第二次应用结束时，若慢波到时晚于快波到时，认为此数据不可接受
    复杂的波形难以辨别，出现误判，如下图（m为系统识别，k为正确到时）
（4）额外规则来提高精度：
    定义系数ck
```
$$k1(i) = \frac{x(i) - x(m + 1)}{i - m - 1}, \quad k2(i) = \frac{x(n - 1) - x(i)}{n - 1 - i}$$

$$
ck(i) = \left| \frac{k1(i)}{k2(i)} \right|
$$

对于 \( i = m + 2, m + 3, ... , n - 2 \)，

$$
\text{max}[ck(m + 2), ck(m + 3), \dots, ck(n - 2)] \geq cH
$$

如果 **ck(i)** 的最大值大于某个特定值 **cH**，则认为该样本点为快（或慢）剪切波的到达点。
```
k1(i) 和 k2(i) 分别表示点 i 与点 m + 1 和点 n - 1 之间幅度差与距离的比值（局部变化率，离散点--->有限差分代替导数计算）
计算 k1(i) 和 k2(i) 的比值的绝对值来得到 ck(i)，目的是衡量幅度变化的相对速度。
具有最大 ck(i) 值的样本点即为快（或慢）剪切波的到达点。若ck(i)最大值未超过阈值cH，到达点即为m(第二阶段计算的初步到达点)
```
![#2 fig 2](https://github.com/user-attachments/assets/9af10e35-6d09-447a-b526-8171dc053fde)
第二阶段完成后，评估ES系统产生的最佳估计

## 4. Identifying quality of shear-wave splitting
  定义 Qp,Qt （1，2，3 --->Good,Acceptable,Unacceptable）评估快剪切波极化方向，慢剪切波到时以及时间延迟 
  定义 Crsn1 和 2rsn2，分别表示Good和Acceptable的信噪比
```
（1）如果信噪比rsn大于等于参考值  crsn1（rsn ≥ crsn1），则Qp = 1
    crsn2 ≤ rsn < crsn1 ， 则Qp = 2  （本文在冰岛的研究中， crsn1 = 5.0 和 crsn2 = 3.0 ）
（2）定义 daft , dbef 分别表示快剪切波到达后，到达前10个点窗口内最大幅值与最小幅值之差
    daft / dbef 表示的是快剪切波到达前后幅度变化的强度比率，比率越大，表示剪切波到达时的变化越显著
    如果 daft / dbef ≥ rd，则 Qt = 1；如果 daft / dbef < rd，则 Qt = 2，其中 rd 由经验确定（在本研究中为2.8）
    对于慢剪切波，daft 和 dbef 同样被用来计算幅度变化，如果快慢剪切波Qt = 1 ，则时间延迟质量也为1
```
## 5. Calculation of polarisation of fast shear-wave
  快剪切波极化方向j通过加权极化图中每个时间延迟点的振动方向来计算，权重是幅值

$$
Φ = \frac{\sum_i Zamp(i) \cdot φ(i)}{\sum_i Zamp(i)}
$$

## 6. Evaluation of ES identification of shear-wave splitting
  使用冰岛西南部BJA站记录的8个月SIL数据进行测试。
处理前  
![#2 fig 3](https://github.com/user-attachments/assets/87784dcc-7326-445f-a648-72b6d0f2c13f)
处理后
![#2 fig4](https://github.com/user-attachments/assets/9a73f3bb-11e4-4b72-a6ba-339af1cf246c)
极化图
![#2 fig 5](https://github.com/user-attachments/assets/1df322ed-f929-4c64-9910-850e83504b12)

ES分析成功地选择了粒子运动方向发生突变的点，显示了快慢剪切波到达时大约90°的方向变化。












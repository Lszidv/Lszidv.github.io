# Classification of Teleseismic Shear Wave Splitting Measurements: A Convolutional Neural Network Approach
# Abstract
  剪切波分裂
  问题：需要可靠的分裂测量数据，目视效率低
  方法：CNN 人工识别数据训练，合成数据测试，实际数据对比
  应用：Boardband seismic data recorded in south central Alaska

# 1.Introduction
  XKS波在各向异性介质中会分裂成两个正交极化的快波和慢波。
  ```
  XKS：地震波在核幔边界（CDB）由P转换为S波[SKS,PKS,SKKS]
  各向异性介质:不同方向物理性质不同的材料（密度、电导率、折射率、拉伸强度等）
      上地壳各向异性：浅地表裂隙填充物质和中下地壳矿物（黑云母、角闪石）晶格优势排列
      上地幔各向异性：橄榄石中的晶格优势取向（LPO）
 
  ```

  分裂参数：快波的极化方向（φ） -->  各向异性方向
                    时间延迟（δt）-->  分裂幅度
   
![splitswithcredit](https://github.com/user-attachments/assets/b8761590-9308-4025-bb9f-e1e1ed4694f2)

  测量方法：“横向能量最小化（SC91特征值法）”方法（Silver & Chan, 1991）
  #1
  结果需人工验证，极为耗时。
  设计CNN，用于验证分裂参数是否有效（“good”,“fair” ,“unacceptable” and “null” ）
  
# 2. Training Data Set and Preprocessing
![台站](https://github.com/user-attachments/assets/0ded0b27-4840-42a7-9c7a-d7faf384e17d)

  数据：美国本土及其周边区域的1,108个观测台站86,903个已发表的、经过人工标注的XKS剪切波分裂（SWS）测量数据，人工测量并分级（ https://doi.org/10.1002/2014GC005267）
  #1
  标准：
  ```
  1.快慢波在矫正过程中的拟合程度[波形叠加和频谱匹配（分裂后的快慢波频谱应相似，即频谱形状和峰值频率一致）]
  2.质点运动的线性度[分析地震波的偏振图，理想情况应是线性偏振]
  3.能量等高线图的稳健性和唯一性[稳定的分裂结果会在能量图中呈现一个稳定、唯一的最小点]
  4.XKS到达强度[检查Original R 、Original T分量上的XKS波形]
  ```
  若不符合标准，调整时窗、带通滤波频率，以排除XKS时窗中的非XKS到达，提高信噪比
  数据覆盖不同构造区域（azimuthally invariant）（azimuthally varying）
  寻找最优的分裂参数（使得修正过后的横向分量具有最小的XKS能量）：将原始横向分量和径向分量作为输入，通过网格搜索得到最优参数组合，进而得到修正（消除剪切波分裂影响）后的横向和径向分量
  将四个地震记录作为输入(每个波形包含1000个节点，50s的时窗，20Hz的采样率)，输出分类标签（分裂参数组合“可接受”和“不可接受”）
  对四种地震记录时窗长度和开始时间的不同组合进行测试，得到了以理论到时为中心50s的时窗（IASP01）<?>
  训练集“可接受”类别多于“不可接受”类别，为消除过拟合影响，设置不同权重（9：1）以抵消数量差异，shuffle = 0.8

# 3.Structure and Training of the CNN
![cnn](https://github.com/user-attachments/assets/c61f8ee5-c4e0-415d-8956-89f6b1a71be7)
  （Length ; Depth）步长为2，数据尺寸逐步减半，特征32维
  模型设计：8个一维卷积层和一个全连接层。（Following Perol et al. (2018)  [DOI: 10.1126/sciadv.1700578](https://doi.org/10.1126/sciadv.1700578)）
  层间使用Relu激活函数，缓解梯度消失
  输出参数组合能否被接受的概率
  交叉熵函数作为损失函数 （#..）
  batch_size = 100, epochs = 64  --> “In each iteration, 100 measurements are randomly selected to train the CNN and each measurement in the training data set is used for 64 times.”
  
   The training history of accuracy and loss:   
 
![training history](https://github.com/user-attachments/assets/e625fb4d-5eaf-48cc-b6bf-e92368d059dd)

#  4. Testing With Synthetic SWS Measurements
  使用合成数据了解CNN在不同参数下的表现（SWS合成数据集   Kong et al.  
https://doi.org/10.1785/0120140108）
 **·  定义快慢波：**
   定义未分裂的XKS波在时间t上的径向分量：

  $$ R(t) = A_0 \sin(2 \pi f t) e^{-\alpha t} $$

  A ：这是未分裂的XKS波的振幅，给定为5000。
  f ：频率，取值为0.125 Hz。
  α：衰减因子，随机在0.1到0.5之间变化。

  快波分量，即未分裂径向分量R(t)旋转角度𝜃后得到：

  $$ S_f(t) = R(t) \cos(\theta) $$

  慢波分量，即时间延迟后未分裂径向分量R(t-δt)旋转角度𝜃后得到：

  $$ S_s(t) = -R(t - \delta t) \sin(\theta) $$

  ```
   分裂的两个波相互正交，使用sin和cos表示，相差90°的相位
   𝜃是快波极化方向 𝜙和后方位角（台站到地震事件与地理北向的夹角）的夹角
  ```
 **·震中距离和震源深度的随机设定：**
  震中距随机设定到90°到120°，对应远震
  震源深度随机设定到20到50km，对应浅源地震
 **·分量投影与随机噪声的添加：**
  将快慢波分量投影到N-S和E-W方向上，添加随机噪声，模拟真实地震信号
  信噪比（SNR) = maxR(t) / maxN(t)
  **·角度𝜃与测量准确性之间的关系：**
  数学上证明原始横向分量上的能量和分裂参数的可靠性都依赖于角度𝜃(Silver & Chan, 1991 https://doi.org/10.1029/91JB00899 )
  以90°为周期，当𝜃小于15°或者大于75°时，快慢波分裂微弱
  **·通过合成数据，进一步量化角度𝜃与测量可靠性之间的关系**
   生成72组合成地震记录，SNR在4到10之间变化，每组包含1000条测量数据，每组的baz为(n-1)×5，确保从0°开始，包含了全范围。
  快波极化方向设定为0，baz等同于角度𝜃
  当baz与快或慢波角度相差15°以内（即接近0°或者90°）时，CNN识别为可接受的分类迅速减少
![baz,snr](https://github.com/user-attachments/assets/f34dd2d2-c719-4e70-8602-07d748564bf2)
    当信噪比≥2.5时，CNN的准确率在90%以上，当信噪比≥6.5时，在97%以上。

# 5. Application to SWS Measurements in South Central Alaska
  将训练好的CNN应用于阿拉斯加中南部地区的宽频地震数据
  数据来自IRIS,预处理流程（#）
  从127个台站的地震数据获得19960对剪切波分裂参数（图a）
  根据信噪比自动排序方法（Liu et al. (2008)）筛选出6314对“potentially acceptable“参数（图b）
  使用CNN进行分类标记，CNN 判定 124 个台站中的 2,668 对测量为“可接受”（图c）
  **·对CNN结果进行筛选**
      （1）排除BAZ与快波极化方向小于15°的测量（图d）
      （2）排除快波极化方向和延迟时间标准差较大的测量(图e)
      （3）排除分裂时间大于2s的测量（图f）
   最后剩余115个台站的1530对测量数据，经过筛选，测量数据逐渐减少，但是同一台站的测量结果一致性得到提升。
![筛选](https://github.com/user-attachments/assets/93944b5f-beef-4865-9c65-ab7a995a79f8)
  
# 6. Discussion
  **·与人工测量相对比**
    在图3b步骤后，人工进行验证，得到106 个台站中得971 条确认的有效测量，其中，98.1%被CNN识别为可接受。因此，可以将人工验证的步骤置于CNN识别之后。
    对人工验证的测量采用相同(1)(2)(3)筛选步骤,相比之下CNN仅遗漏5.7%，遗漏的测量数据集中在区域 B，“where the interaction of two flow systems with nearly orthogonal directions”(Yang et al., 2021).

![结果](https://github.com/user-attachments/assets/d240cc06-2771-435e-84a5-b381cc61bdec)

    图 (b) 和图 (c) 表明，CNN 和人工操作员的结果在台站平均快波方向和分裂时间上高度相似，但快波方向的相似性更高
    图 (d) 和图 (e) 显示了 CNN 可能误判了一些质量不佳的测量为“可接受”，特别是在 B 区域，这些误判导致了 CNN 和人工结果在分裂时间上的相关性下降。

  **· CNN 方法与非机器学习的全自动剪切波分裂测量方法（SplitRacerAUTO）的对比**
  将CNN结果与非机器学习的自动化方法 SplitRacerAUTO（https://doi.org/10.1016/j.cageo.2021.104961)对比
  相同数据集与预处理步骤，CNN效果更好 [ 绘制与图3相同样式的SWS测量参数图 ，置于补充信息中]
  
  **· CNN 的局限与进一步工作**
    CNN无法自动选择时窗的开始与结束时间，带通滤波频率范围
    当震中距（D = 𝜃×R）小于90°时，S波到达台站较早，可能会混入XKS时窗
    解决思路：①在预处理阶段设计一个CNN，用于拾取XKS波到时，从而准确设置XKS波时窗起点；②非 CNN 方法确定最佳 XKS 窗口，例如 Link 等人（2022）提出的时频谱技术。

# 7.Conclusions    
  研究基于已发表的人工标记数据集建立的 CNN 模型在自动分类剪切波分裂测量中的应用，强调了其在高信噪比条件下的优异性能，以及在阿拉斯加数据上的成功应用，表明基于 CNN 的方法在提高测量效率方面的潜力。

# 7.Data Availability Statement 
  程序和模型： https://github.com/YW-Zhang94/CNN-SWS.git
  地震数据（SAC）：https://figshare.com/articles/dataset/CNN_SWS_data/19904833.（2.1G）
  波形数据：(https://ds.iris.edu/ds/nodes/dmc/, last accessed March 2019)，从IRIS获取
    under the main network codes of 
    AK (https://doi.org/10.7914/SN/AK),
    AT (https://doi.org/10.7914/SN/AT), 
    DW (https://doi.org/10.7914/SN/DW), 
    IM (International Miscellaneous Stations),
    IU (https://doi.org/10.7914/SN/IU),
    TA (https://doi.org/10.7914/SN/TA), 
    XE (https://doi.org/10.7914/SN/XE_1999), 
    XR (https://doi.org/10.7914/SN/XR_2004),
    XV (https://doi.org/10.7914/SN/XV_2014),
     YE (https://doi.org/10.7914/SN/YE_2011), 
    YV (https://doi.org/10.7914/SN/YV_2006).


[Geophysical Research Letters - 2022 - Zhang - Classification of Teleseismic Shear Wave Splitting Measurements  A.pdf](https://github.com/user-attachments/files/17540440/Geophysical.Research.Letters.-.2022.-.Zhang.-.Classification.of.Teleseismic.Shear.Wave.Splitting.Measurements.A.pdf)

### Classification of Teleseismic Shear Wave Splitting Measurements: A Convolutional Neural Network Approach
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

  数据：美国本土及其周边区域的1,108个观测台站86,903个已发表的、经过人工标注的XKS剪切波分裂（SWS）测量数据，人工测量并分级
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
  模型设计：8个一维卷积层和一个全连接层。（Following Perol et al. (2018)）
  层间使用Relu激活函数，缓解梯度消失
  输出参数组合能否被接受的概率
  交叉熵函数作为损失函数 （#..）
  batch_size = 100, epochs = 64  --> “In each iteration, 100 measurements are randomly selected to train the CNN and each measurement in the training data set is used for 64 times.”
  
   The training history of accuracy and loss:   
 
![training history](https://github.com/user-attachments/assets/e625fb4d-5eaf-48cc-b6bf-e92368d059dd)

#  4. Testing With Synthetic SWS Measurements
  使用合成数据了解CNN在不同参数下的表现（SWS合成数据集   Kong et al.  
https://doi.org/10.1785/0120140108）
   
   ![公式](https://latex.codecogs.com/png.latex?R(t)%20%3D%20A_0%20%5Csin(2%5Cpi%20f%20t)%20e%5E%7B-%5Calpha%20t%7D)




  

# 5. Application to SWS Measurements in South Central Alaska

[Geophysical Research Letters - 2022 - Zhang - Classification of Teleseismic Shear Wave Splitting Measurements  A.pdf](https://github.com/user-attachments/files/17540440/Geophysical.Research.Letters.-.2022.-.Zhang.-.Classification.of.Teleseismic.Shear.Wave.Splitting.Measurements.A.pdf)

# SWAS: A shear-wave analysis system for semi-automatic measurement of shear-wave splitting above small earthquakes
##  Abstract
   问题：小震剪切波到达复杂，剪切波测量困难
   方法：开发SAWS专家系统，自动估计快波极化方向和剪切波到时，人工辅助调整（在原始地震图、旋转地震图和极化图之间进行调整）
   数据：冰岛SIL地震网络数据

 ```
  里氏震级：基于地震波振幅，在这个标度中，为了使结果不为负数，里克特定义在距离震中100千米处之观测点地 
  震仪记录到的最大水平位移为1微米（这也是伍德-安德森扭力式地震仪的最大精度）的地震作为0级地震，当地震 
  仪记录到的最大水平位移小于1微米，震级便为负。

  矩震级：基于震源区面积，断层滑移，岩石刚度等参数，适用于大范围地震
 ```
$$  M_L = \log_{10}(A) + 3 \log_{10}(8T) - 2.92 $$
$$ M_W = \frac{2}{3} \log_{10}(M_0) - 6.033 $$
![震级](https://github.com/user-attachments/assets/d3b61cbd-40a3-4983-bffe-e233856f3e17)

## 1.Introduction
  由于信号微弱，并且收到散射和各向异性的影响，准确测量小震的剪切波分裂十分困难，但其对于理解震前微裂隙的变型过程至关重要。
  在数据稀疏或者质量不高的情况下，过度依赖筛选会导致数据偏差。
  现有的技术（SAM系统：通过计算快慢波互相关函数来确定它们的时间延迟，快波极化方向通过极化图来帮助识别）存在局限性（需要高信噪比数据，主观性强，耗时耗力，不适合大规模自动化应用）
  
## 2.An expert system for automatic identification and measurementr
  专家系统利用地震目录中的地震定位参数以及P波和S波到时作为输入，在剪切波窗口（入射角 ≤ 45° ）内初步计算剪切波分裂参数，随后经过第二套专家规则进行评估和优化，专家规则详见 #
  
![屏幕截图 2024-11-15 194555](https://github.com/user-attachments/assets/4a53052b-bf65-4e80-8aae-40a80cf62c14)

## 3.Accessing quality of measurements
  定义剪切波分裂测量质量的评估方法，按信噪比来评估Qp<1,良好；2：可接受；3：不可接受 >(极化方向质量)，按快波到达前一个时窗内振幅大小比值来评估Qt(延迟时间质量)
  
## 4. Automaticyinli picks and visual adjustments
  SWAS的界面设计
  fig2和fig3，两个使用案例
![2a](https://github.com/user-attachments/assets/4af082f3-35db-471e-b682-4491615c2955)
![2b](https://github.com/user-attachments/assets/e4e3a91f-db7c-4416-8955-a7fcb0aac612)
![2c](https://github.com/user-attachments/assets/1888e068-826b-4322-a088-c24e240650be)
![2d](https://github.com/user-attachments/assets/b1b01470-e002-478e-b153-db15bf89fbaa)
![2e](https://github.com/user-attachments/assets/c2a28aca-d94f-4c84-8224-9637f65efe26)
![3a](https://github.com/user-attachments/assets/c1bedb58-f7c5-4ed1-b504-4cb66564847e)
![3b](https://github.com/user-attachments/assets/712ad825-322f-41dd-9f4a-b7595758feaf)
  ```
1.初始三分量波形：原始波形，地震仪在三个方向上的记录
    Z方向为垂直记录，主要反映P波信号，剪切波信号弱
    E、N分量为水平记录，包含剪切波信号
2.从 E、N 到 R、T 分量
    Radial(径向分量)，沿震源传播方向的振动
    Transverse(横向分量)，垂直传播方向的振动，主要反映剪切波的横向振动
    通过旋转矩阵（震源到台站之间的方位角）将 E、N 分量投影到 R、T 分量, 其包含快慢波的混合信号
3.从R、T到快慢波分量
    Fast(S1)分量，沿快波极化方向的振动分量，主要反应快波特征
    Slow(S2)分量，垂直快波极化方向的分量，主要反应慢波特征
    通过旋转矩阵（快波极化方向）将 E、N 分量投影到 R、T 分量，将快慢波分离
  ```
[rotation](https://service.iris.edu/irisws/rotation/docs/1/help/)

```
   极化图：直观展示剪切波分裂过程中粒子在水平面的运动轨迹 ，在每个时间窗口内计算粒子运动轨迹，叠加为一个大图，用于展示剪切波极化行为和快慢波的分离。
    快波极化方向通常与介质裂隙方向一致
    快慢波极化方向垂直
    轨迹从快波到慢波阶段表现出方向上的显著变化
    快波阶段的运动轨迹较为线性，而慢波阶段的轨迹可能呈现更复杂的形状
```
## 5. Accessing effectiveness of SWAS/ES techniques
  研究范围：8个月内8个台站的855次地震事件，数据来源为震级 𝑀≥−1.0的小震记录。
  525条记录（61%）被评估为质量良好（𝑄𝑝,𝑄𝑡=1,2）
  准确率见下图：
![准确率](https://github.com/user-attachments/assets/51398786-47ee-4626-b432-b9c124ed9b37)

  剪切波分裂在小震上方的观测数据中，时间延迟有明显随机散布，这种效应无法用叠加技术来去除（叠加只会掩盖这一显著特征。这种散布不能用传统的读数误差、定位误差或结构解释误差来解释 Volti和Crampin, 2003a,b），而在人工源地震中，虽然也会在界面处出现大幅变化，但不会像小震一样在远离断层带的区域出现如此显著的随机散布，解释为断层带上高孔隙流体压力引起的，这种压力使得微裂隙的几何形状远离区域应力方向，导致剪切波极化方向发生90°的翻转（Crampin和Zatsepin, 1997）。这种现象已在模型模拟和在油气藏、断层带的观测中得到证实。
  
## 6. Comparing SWAS/ES and visual measurements  
  对时间延迟按照射线路径进行分组，射线路径方向与平均裂隙平面（Average Crack Plane）夹角在 15°-45° 之间，其对裂隙形状（即纵横比）更敏感，夹角在 15°以内对裂隙密度更敏感。

## 7. Conclusions
  自动化初步结果和便捷的人工调整功能

## Appendix A. An expert system for measuring shear-wave splitting
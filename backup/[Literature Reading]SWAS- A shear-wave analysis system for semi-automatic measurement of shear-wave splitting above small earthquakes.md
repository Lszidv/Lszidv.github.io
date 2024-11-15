# SWAS: A shear-wave analysis system for semi-automatic measurement of shear-wave splitting above small earthquakes
#  Abstract
   问题：小震剪切波到达复杂，剪切波测量困难
   方法：开发SAWS专家系统，自动估计快波极化方向和剪切波到时，人工辅助调整（在原始地震图、旋转地震图和极化图之间进行调整）
   数据：冰岛SIL地震网络数据

 ```
  里氏震级：基于地震波振幅，在这个标度中，为了使结果不为负数，里克特定义在距离震中100千米处之观测点地 
  震仪记录到的最大水平位移为1微米（这也是伍德-安德森扭力式地震仪的最大精度）的地震作为0级地震，当地震 
  仪记录到的最大水平位移小于1微米，震级便为负

  矩震级：基于震源区面积，断层滑移，岩石刚度等参数，适用于大范围地震
 ```
$$  M_L = \log_{10}(A) + 3 \log_{10}(8T) - 2.92 $$
$$ M_W = \frac{2}{3} \log_{10}(M_0) - 6.033 $$
![震级](https://github.com/user-attachments/assets/d3b61cbd-40a3-4983-bffe-e233856f3e17)

# Introduction
  由于信号微弱，并且收到散射和各向异性的影响，准确测量小震的剪切波分裂十分困难，但其对于理解震前微裂隙的变型过程至关重要。
  在数据稀疏或者质量不高的情况下，过度依赖筛选会导致数据偏差。
  现有的技术（SAM系统：通过计算快慢波互相关函数来确定它们的时间延迟，快波极化方向通过极化图来帮助识别）存在局限性（需要高信噪比数据，主观性强，耗时耗力，不适合大规模自动化应用）
  


# An expert system for automatic identification and measurement
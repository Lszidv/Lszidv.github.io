# SWAS: A shear-wave analysis system for semi-automatic measurement of shear-wave splitting above small earthquakes
#  Abstract
   问题：小震剪切波到达复杂，剪切波测量困难
   方法：开发SAWS专家系统，自动估计快波极化方向和剪切波到时，人工辅助调整
   数据：冰岛SIL地震网络数据

 ```
  里氏震级：基于地震波振幅，在这个标度中，为了使结果不为负数，里克特定义在距离震中100千米处之观测点地 
  震仪记录到的最大水平位移为1微米（这也是伍德-安德森扭力式地震仪的最大精度）的地震作为0级地震，当地震 
  仪记录到的最大水平位移小于1微米，震级便为负

  矩震级：基于震源区面积，断层滑移，岩石刚度等参数，适用于大范围地震
 ```
$$  M_L = \log_{10}(A) + 3 \log_{10}(8T) - 2.92 $$
$$ M_W = \frac{2}{3} \log_{10}(M_0) - 6.033 $$

# Introduction

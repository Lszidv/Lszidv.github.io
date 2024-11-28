## Summary
相位检测、识别和初至时间是分析地震数据的基础且重要的常规工作。台站，地震记录，仪器设备的增多和普及使得 人工识别和挑选地震相位的工作不可行，自动化处理更为重要。

  first stage --- CNN(detects the phases)；second stage --- RNN (pick both phases)
  检测在三分量上进行，垂向分量用来拾取P波，水平分量用来拾取S波
  数据集：北智利的手动挑选的地震波形记录。检测和挑选的记录为39,000条（detection）和36,000条（picking）
DeepPhasePick方法通过蒙特卡洛丢弃技术来计算起始时间的不确定性，该技术模拟了贝叶斯推断的过程
```

```
## Introduction
  

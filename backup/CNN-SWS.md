## 论文“Classification of Teleseismic Shear Wave Splitting Measurements: A Convolutional Neural Network Approach”代码部分

### 文件结构
```
CNN-SWS-main/
├── 1_data/                                # 数据文件夹
│   ├── Out_Bin/                           # 存储 XKS.out 文件，.out文件包含三列，台站和网络名称（stname_NW）、事件名称（EQ123456789）、测量质量（A 和 B 表示可接受，其余表示不可接受）      
│   │   ├── PKS.out                       
│   │   ├── SKK.out
│   │   └── SKS.out
│   └── PKSOut/                            # 存储不同台站和事件的波形数据
│       ├── 109Cxx_TA/                     # 台站文件夹
│       │   ├── EQ140250514/               # 事件文件夹
│       │   │   ├── 109Cxx_TA.rl           # 校正径向分量
│       │   │   ├── 109Cxx_TA.ro           # 原始径向分量
│       │   │   ├── 109Cxx_TA.tl           # 校正横向分量
│       │   │   └── 109Cxx_TA.to           # 原始横向分量
│       │   ├── EQ********/               #其他事件
│       ├── **************                   # 其他台站文件夹
│   ├── PKS.list                               # Out_Bin/PKS.out
│   ├── SKK.list                               # Out_Bin/SKK.out
│   └── SKS.list                               # Out_Bin/SKS.out
│
├── load/                                   # 数据加载和预测文件夹
│   ├── 2_load/                             # 加载输出文件夹
│   │   └── Outp/                           # 存放加载预测结果
│   ├── load.py                             # 数据加载和预测脚本
│   └── parameter.list                      # 加载过程参数
│
├── model/                                  # 模型文件夹
│   └── CNN_XKS.h5                          # 已训练好的模型权重
│
├── train/                                  # 模型训练文件夹
│   ├── 2_train/                            # 训练输出文件夹
│   │   ├── CNN_XKS.h5                      # 训练后的模型权重
│   │   ├── parameters.list                 # 训练过程参数(单独写出来，方便改动)
│   │   ├── train_64.acc                    # 训练精度记录
│   │   ├── train_64.loss                   # 训练损失记录
│   │   ├── train_64.val_acc                # 验证精度记录
│   │   └── train_64.val_loss               # 验证损失记录
│   └── train.py                            # 模型训练脚本
│
├── test/                                   # 测试文件夹
│   └── test.py                             # 模型可视化和测试脚本
│
├── Do_load.cmd                             # 加载命令脚本
├── Do_train.cmd                            # 训练命令脚本
└── README.txt                              # 项目说明文件


```

### load.py
  ```
import os
import numpy as np
from obspy import read
from keras.models import Sequential
from keras.layers import Conv1D, ZeroPadding1D, Flatten, Dense

X = []                                                                        # 数据列表
Y = []                                                                        # 标签列表
X_nst, Y_nev = [], []                                                       # 台站名和事件名
input_length = 1000

# 数据和模型加载
nrt = os.path.normpath('C:/Users/~/S-wave spliting/CNN-SWS-main/1_data')
nmodel = os.path.normpath('C:/Users/~/S-wave spliting/CNN-SWS-main/model/CNN_XKS.h5')
# 路径检查
if not os.path.exists(nrt):
    raise FileNotFoundError(f"The data root path {nrt} does not exist.")
if not os.path.exists(nmodel):
    raise FileNotFoundError(f"The model path {nmodel} does not exist.")
```

```
# 读取SAC数据
XKS = ['PKS', 'SKS', 'SKK']

for k in range(3):
    XKS_rout = os.path.join(nrt, f'{XKS[k]}.list')               # C:/Users/~/main/1_data/Out_Bin/*.out
    print(f"Reading {XKS_rout}")

    if not os.path.exists(XKS_rout):
        raise FileNotFoundError(f"The file {XKS_rout} does not exist.")

# 逐行读取数据
with open(XKS_rout, 'r') as Pl:
    for line_Pl in Pl:
        vals = line_Pl.split()
        P_rout = os.path.join(nrt, vals[0])                       # 数据根目录nrt ＋ .out文件第一列
        print(f'Doing: {XKS[k]} {vals[0]}')

        if not os.path.exists(P_rout):
            raise FileNotFoundError(f"The file {P_rout} does not exist.")
```
```
PKS, y = [], []                                                              # PKS用于储存波形数据，y储存分类标签
with open(P_rout, 'r') as P:
    for line in P:
        vals = line.split()
        nst = vals[0]                                                       # station name
        nev = vals[1]                                                      # event name

        # 处理分类标签
        if vals[2] in ['A', 'B']:
            y.append(1)
            y.append(0)
        else:
            y.append(0)
            y.append(1)

```
```

ncom = ['.ro', '.to', '.rl', '.tl']                                    # 4分量列表    
components = []
for i in range(4):
    ro_rout = os.path.join(nrt, f'{XKS[k]}Out', nst, nev, f'{nst}{ncom[i]}')
    print(f'Reading file: {ro_rout}')

    if os.path.exists(ro_rout):
        st = read(ro_rout)
        components.append(st[0].data[:input_length])   # 截取前input_length个数据
    else:
        raise FileNotFoundError(f"The file {ro_rout} does not exist.")

for i in range(input_length):
    PKS.append(np.array([comp[i] for comp in components]))

X.append(np.array(PKS))                                            # 将4个分量组成的PKS保存到列表X中
Y.append(np.array(y))                                                # 分类标签保存到Y中
X_nst.append(f"{nst}_{XKS[k]}_")                                # 台站信息
Y_nev.append(nev)                                                    # 事件信息

```

```
# 定义模型
input_shape = (input_length, 4)
model = Sequential()
   # 添加卷积层
model.add(Conv1D(kernel_size=3, filters=32, input_shape=input_shape, strides=2, activation='relu'))
model.add(ZeroPadding1D(padding=1))
model.add(Conv1D(kernel_size=3, filters=32, strides=2, activation='relu'))
model.add(ZeroPadding1D(padding=1))
model.add(Conv1D(kernel_size=3, filters=32, strides=2, activation='relu'))
model.add(ZeroPadding1D(padding=1))
model.add(Conv1D(kernel_size=3, filters=32, strides=2, activation='relu'))
model.add(ZeroPadding1D(padding=1))
model.add(Conv1D(kernel_size=3, filters=32, strides=2, activation='relu'))
model.add(Flatten())
model.add(Dense(units=2, activation='softmax'))

# 加载模型权重并进行预测
model.load_weights(nmodel)
result = model.predict(np.array(X))

# 保存预测结果
output_dir = os.path.join('C:/Users/~/S-wave spliting/CNN-SWS-main/load/2_load/Outp')
os.makedirs(output_dir, exist_ok=True)
for i in range(len(result)):
    nst = X_nst[i]                                                                        
    nev = Y_nev[i]                                                                      
    res_name = os.path.join(output_dir, f'{nst}_{nev}.res')         
    y_name = os.path.join(output_dir, f'{nst}_{nev}.y')

    np.savetxt(res_name, result[i])
    np.savetxt(y_name, Y[i])

print('finish')
```



### train.py

```
import numpy as np
import matplotlib.pyplot as plt
import obspy
import csv
from obspy import read
from obspy.taup import TauPyModel
import os
from pathlib import Path
import random
import keras
from keras import regularizers
from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten, Conv2D, Conv1D, MaxPooling1D, UpSampling1D, ZeroPadding1D

# 初始化
X_good, Y_good = [], []
X_bad, Y_bad = [], []
X, Y = [], []
X_rand, Y_rand = [], []
nst_good, nev_good = [], []
nst_bad, nev_bad = [], []
X_nst, Y_nev = [], []
nst_rand, nev_rand = [], []
```

```
# 读取参数文件
n=0
with open('parameters.list') as p:
    for line in p:
        n += 1
        vals = line.split()
        if n == 1: nrt = str(vals[0])
        if n == 2: batch_size = int(vals[0])
        if n == 3: epochs = int(vals[0])
        if n == 4: byn = int(vals[0])
        if n == 5:
            ac = int(vals[0])
            uc = int(vals[1])

```

```
# 读取SAC数据
input_length = 1000
XKS = ['PKS', 'SKS', 'SKK']

for k in range(3):
    XKS_rout = nrt + str(XKS[k]) + '.list'
    with open(XKS_rout) as Pl:
        for line_Pl in Pl:
            vals = line_Pl.split()
            P_rout = nrt + str(vals[0])
            with open(P_rout) as P:
                for line in P:
                    PKS, y = [], []
                    vals = line.split()
                    nst = vals[0]  # station name
                    nev = vals[1]  # event name
                    y = [1, 0] if vals[2] in ['A', 'B'] else [0, 1]
                    
                    ncom = ['.ro', '.to', '.rl', '.tl']
                    for i in range(4):
                        ro_rout = f"{nrt}{XKS[k]}Out/{nst}/{nev}/{nst}{ncom[i]}"
                        st = read(ro_rout)
                        if i == 0: ro = st[0].data
                        if i == 1: to = st[0].data
                        if i == 2: rl = st[0].data
                        if i == 3: tl = st[0].data
                        
                    for i in range(input_length):
                        PKS.append(np.array([ro[i], to[i], rl[i], tl[i]]))
                    if y[0] == 1:
                        X_good.append(PKS)
                        Y_good.append(y)
                        nst_good.append(f"{nst}_{XKS[k]}_")
                        nev_good.append(nev)
                    else:
                        X_bad.append(PKS)
                        Y_bad.append(y)
                        nst_bad.append(f"{nst}_{XKS[k]}_")
                        nev_bad.append(nev)
```
```
# 数据处理
npts = int(len(X_bad) / len(X_good))
class_weight = {0: ac, 1: uc}
if byn == 0: npts = 1

for i in range(npts):
    for ii in range(len(X_good)):
        X.append(X_good[ii])
        Y.append(Y_good[ii])
        X_nst.append(nst_good[ii])
        Y_nev.append(nev_good[ii])

for i in range(len(X_bad)):
    X.append(X_bad[i])
    Y.append(Y_bad[i])
    X_nst.append(nst_bad[i])
    Y_nev.append(nev_bad[i])

rann0 = random.sample(range(len(X)), len(X))
X_rand = [X[i] for i in rann0]
Y_rand = [Y[i] for i in rann0]

x_train, y_train = np.array(X_rand[:int(len(X) * 0.8)]), np.array(Y_rand[:int(len(Y) * 0.8)])
x_test, y_test = np.array(X_rand[int(len(X) * 0.8):]), np.array(Y_rand[int(len(Y) * 0.8):])

```
```
model = Sequential()
model.add(Conv1D(32, kernel_size=3, strides=2, activation='relu', input_shape=(input_length, 4)))
model.add(ZeroPadding1D(1))
# 添加多个卷积层，最终展平成向量并连接到 softmax 输出层
model.add(Flatten())
model.add(Dense(2, activation='softmax'))

model.compile(loss='categorical_crossentropy', optimizer=keras.optimizers.Adam(lr=0.001), metrics=['accuracy'])
H = model.fit(x_train, y_train, batch_size=batch_size, epochs=epochs, class_weight=class_weight, validation_data=(x_test, y_test))

fig, ax = plt.subplots()
plt.plot(H.history['acc'], label='train_acc')
plt.plot(H.history['val_acc'], label='val_acc')
plt.legend()
plt.show()

model.save_weights('CNN_XKS.h5')
print('Finish')

```


## **CNN_Local_SWS: S 波到时计算**

本文件的主要任务是 **计算 S 波的到达时间**，并将其用于后续的剪切波分裂分析。该计算过程涉及 **理论到时计算**、**S 波实际到时计算**、**时间扰动引入** 以及 **波形数据读取**，确保时间窗口的选取足够精确。

---

### **1. 计算 S 波的理论到时**
```python
if S_arrival == -1:
    S_arrival = model_s.get_travel_times(
        source_depth_in_km=st[0].stats.sac['evdp'],
        distance_in_degree=st[0].stats.sac['gcarc'],
        phase_list=["s"]
    )[0].time
```
</code></pre>
<ul>
<li><strong>使用 TauPyModel 计算 S 波传播时间</strong>，输入震源深度 <code inline="">evdp</code> 和震中距 <code inline="">gcarc</code>，返回 <strong>S 波的理论到达时间</strong> <code inline="">S_arrival</code>（单位：秒）。</li>
</ul>
<hr>
<h3><strong>2. 计算 S 波的实际到时</strong></h3>
<pre><code class="language-python">S_arr = int((st[0].stats.sac['o'] + S_arrival - st[0].stats.sac['b']) * 100) + shift
</code></pre>
<ul>
<li><strong>基于理论到时计算实际到时</strong>：
<ul>
<li><code inline="">o</code>：地震事件发生时间。</li>
<li><code inline="">S_arrival</code>：S 波传播时间。</li>
<li><code inline="">b</code>：数据记录开始时间。</li>
<li><strong>转换为 1/100 秒单位</strong> 并加入随机偏移 <code inline="">shift</code>。</li>
</ul>
</li>
</ul>
<hr>
<h3><strong>3. 引入时间扰动</strong></h3>
<pre><code class="language-python">for ishift in range(22):
    shift = int((ishift - 10) * 2 - np.random.choice([0,1], size=1))
    if ishift == 10:
        shift = 0
</code></pre>
<ul>
<li><strong>在 <code inline="">±20</code> 采样点范围内加入随机偏移</strong>，模拟测量误差，提高数据鲁棒性。</li>
</ul>
<hr>
<h3><strong>4. 读取 SAC 头文件时间信息</strong></h3>
<pre><code class="language-python">f = st[0].stats.sac['f']
f_tr = int((f - st[0].stats.sac['b']) * 100)
</code></pre>
<ul>
<li><strong>获取时间窗口信息</strong>：
<ul>
<li><code inline="">f</code>：事件时间窗口结束时间。</li>
<li><code inline="">b</code>：记录开始时间。</li>
<li>计算 <strong>时间窗口长度</strong> <code inline="">f_tr</code>（单位：采样点）。</li>
</ul>
</li>
<li><strong>讨论：<code inline="">b</code> vs `a</strong>：
<ul>
<li><code inline="">b</code> 代表 <strong>文件记录的开始时间</strong>，用于计算 S 波到达时间的相对时刻。</li>
<li><code inline="">a</code> 代表 <strong>事件初动时间</strong>，通常指 <strong>P 波初动</strong>，在 S 波计算中可能并不适用。</li>
</ul>
</li>
</ul>
<hr>
<h3><strong>5. 读取波形数据</strong></h3>
<pre><code class="language-python">nz = '../data/SWS_trace/' + nst + '/' + neq + '/' + nst + '.z'
nn = '../data/SWS_trace/' + nst + '/' + neq + '/' + nst + '.n'
ne = '../data/SWS_trace/' + nst + '/' + neq + '/' + nst + '.e'

st = read(nz)
</code></pre>
<ul>
<li><strong>读取三分量波形数据（Z、N、E）</strong>，用于剪切波分裂分析。</li>
</ul>
<hr>
<h2><strong>总结</strong></h2>

步骤 | 关键代码 | 作用
-- | -- | --
计算 S 波理论到时 | model_s.get_travel_times() | 计算 S 波的传播时间
计算 S 波实际到时 | (o + S_arrival - b) * 100 + shift | 确定 S 波到达时间
时间扰动 | shift = (ishift - 10) * 2 - np.random.choice([0,1]) | 使 S 波到时产生随机偏移
读取 SAC 头文件 | f = st[0].stats.sac['f'] | 计算时间窗口信息
读取波形数据 | st = read(nz) | 读取 Z、N、E 三分量数据


<p><strong>最终 <code inline="">S_arr</code> 以</strong> <strong>1/100 秒单位计算</strong>，用于剪切波分裂分析。</p>
<pre><code>

---
## Pytheas计算S波到时

## 函数: `arPicker`

该函数使用AR-AIC方法（Akazawa, 2004）来自动计算P波和S波的到达时间。此方法集成在Obspy中，使用`obspy.signal.trigger.ar_pick`来实现。

### 参数：
- `stream`: 包含波形数据的`Stream`对象（包含3个分量：垂直分量、北向分量和东向分量）。该数据流用于计算P波和S波到达时间。
- `pkCNF`: 配置类`parsePickerCnf`，包含S波和P波到达时间计算所需的设置参数。

### 返回值：
- `p`: P波到达时间，相对于记录开始时间的秒数。
- `s`: S波到达时间，相对于记录开始时间的秒数。

```
def arPicker(stream, pkCNF):
    df = stream[0].stats.sampling_rate  # 采样率
    z, n, e = stream  # 获取三分量（垂直、北向、东向）
    
    # 使用ar_pick计算P波和S波的到达时间
    p, s = ar_pick(
        z.data, n.data, e.data, df,
        pkCNF.arFMIN, pkCNF.arFMAX, pkCNF.arLTAP, pkCNF.arSTAP,
        pkCNF.arLTAS, pkCNF.arSTAS,
        pkCNF.arMP, pkCNF.arMS, pkCNF.arLP, pkCNF.arLS,
        s_pick=True  # 标记计算S波的到达时间
   )
    
    logging.info("AR-AIC arrivals (p,s) in s: " + str((p, s)))
    return p, s
```

### 关键步骤：
1. **参数初始化**：从`Stream`对象中获取采样率（`df`）和波形数据（`z`, `n`, `e`）。这些数据是用于计算P波和S波到达时间的输入。
2. **调用`ar_pick`**：使用`ar_pick`函数计算P波和S波的到达时间。该函数包含了多个参数，用于配置计算过程中的频率范围、滤波器参数等。
3. **返回结果**：返回P波和S波的到达时间（单位为秒），并记录日志输出。

### 参考文献：
- Akazawa, T., 2004. A technique for automatic detection of onset time of P-and S-Phases in strong motion records, 13th World Conference on Earthquake Engineering.

---

### 总结：
该函数利用AR-AIC方法自动计算S波到达时间，并返回相对于记录开始的时间。计算过程依赖于多个配置参数，确保可以在不同的数据条件下正确检测到P波和S波的到达时间。
```

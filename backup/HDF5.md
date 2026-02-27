
## 目录
- [1. HDF5 是什么](#1-hdf5-是什么)
- [2. 与 SAC / miniSEED 的区别：为什么要用 HDF5](#2-与-sac--miniseed-的区别为什么要用-hdf5)
- [3. 核心对象模型：Group / Dataset / Attribute](#3-核心对象模型group--dataset--attribute)
- [4. 设计你的地球物理数据集：两种 schema](#4-设计你的地球物理数据集两种-schema)
- [5. 性能核心：Chunk / Compression / Cache](#5-性能核心chunk--compression--cache)
- [6. 并行与数据规模：sharding、VDS、Parallel HDF5](#6-并行与数据规模shardingvdsparallel-hdf5)
- [7. 地球物理工程实践：从 SAC/miniSEED 到 HDF5](#7-地球物理工程实践从-sacminiseed-到-hdf5)

---

## 1. HDF5 是什么

HDF5（Hierarchical Data Format v5）是一个**二进制科学数据容器**，核心能力是把“**数据 + 元数据 + 组织结构 + 高性能 I/O 策略**”放进一个文件里。

- **文件里的小型文件系统**
- 里面有“文件夹”（Group）和“文件”（Dataset）
- 每个对象还能挂“属性”（Attribute）作为元数据

### 1.1 你会从 HDF5 得到什么
- **层级结构**：适合事件-台站-窗段（event/station/window）组织
- **自描述**：shape、dtype、单位、处理参数、版本号都能固化
- **高性能 I/O**：支持分块（chunk）与压缩（compression），按需切片读取
- **可扩展**：可增长维度（append 新样本），可虚拟拼接（VDS）

---

## 2. 与 SAC / miniSEED 的区别

| 维度 | SAC | miniSEED | HDF5 |
|---|---|---|---|
| 主要定位 | 分析与可视化友好（单道/单分量常见） | 台网采集/归档/交换标准（连续流强） | 科学数据容器（自定义 schema，训练与批处理强） |
| 组织方式 | 海量小文件 + header | 波形分片 + 生态配套（StationXML） | 少量大文件 + 内部层级/表 |
| 元数据 | 固定 header 字段 | 元数据常外置 | 自由扩展 attributes / meta 表 |
| 训练 I/O | 打开文件多，元数据开销大 | 同上（但连续流场景强） | 按索引随机读快，适合 `X[N,3,T]` |
| 适合场景 | 震相处理、研究常用工具链 | 台网数据分发、实时流 | 深度学习、批量实验、可追溯数据产品 |

### 2.1 为什么使用 HDF5 
当你做 SWS（例如标签 **$ (\phi,\Delta t) $**）或任何窗口化学习任务时，训练访问常是：

- 随机取样：`X[i,:,:]`
- 或读时间片段：`X[i,:,t0:t1]`

SAC/miniSEED 的痛点是“**文件系统元数据开销**”：打开/关闭/遍历海量小文件非常慢。  
HDF5 的优势是“**数组式访问**”：按索引切片读写，吞吐更稳定。

---

## 3. 核心对象模型：Group / Dataset / Attribute

### 3.1 Group（组）
- 类似目录
- 用路径定位：`/events/19960126.028/SJ01`

### 3.2 Dataset（数据集）
- 多维数组：`float32 [N,3,T]`
- 也可存表：compound dtype（类似结构体/表格）

### 3.3 Attribute（属性）
- 适合小规模元信息：单位、采样率、版本号、坐标系约定
- 不适合存大数组（大数组应作为 Dataset）

### 3.4 推荐的元数据分层策略
- 文件级 attributes：全局约定（`fs`、`component_order`、`bandpass`、`dataset_version`）
- meta 表：样本级字段（`event_id`、`station`、`baz`、`evdp`、`quality`、`t0/t1`）
- dataset attributes：对特定数组的单位/解释（`units="m/s"`，`axis_order="NCT"`）

---

## 4. 地球物理数据集：两种 schema

> 选择 schema 的关键：“可读性”还是“训练吞吐”？

### 4.1 Schema A：层级树（直观、友好）
适合：数据量中等、以研究分析与手工检查为主

优点：
- 样本索引访问极快（`X[i]`）
- 对象少，元数据开销低
- 更易分片（sharding）

缺点：
- 需要你维护索引与 meta

---

## 5. 性能核心：Chunk / Compression / Cache

> 一句话：HDF5 是否快，主要取决于 **chunk 是否匹配访问模式**。

### 5.1 Chunk（分块）决定切片读取成本
- contiguous：连续布局，整段顺序读快，子集切片不一定快
- chunked：分块布局，支持压缩、支持 resize、子集切片更灵活

### 5.2 Compression（压缩）不一定慢
对波形这种相关性强的数据，压缩后磁盘读取量降低，常常整体更快。  
但前提是：chunk 不能太大，且访问模式不要让同一 chunk 反复解压。

### 5.3 Cache（chunk cache）决定“是否重复解压”
如果 chunk 大于 cache，或访问模式跨很多 chunk，会造成：
- 重复读
- 重复解压
- 看起来“压缩后更慢”

### 5.4 地球物理训练数据的经验 chunk 配方
**访问模式 1：以样本为单位随机读 `X[i,:,:]`**
- `X` 形状：`[N,3,T]`
- chunk 建议：`[64,3,T]` 或 `[128,3,T]`（接近 batch）

**访问模式 2：以时间片段读 `X[i,:,t0:t1]`**
- chunk 建议：`[64,3,256]`（时间维也分块）
- 注意：chunk 太小会增加元数据/索引负担

---

## 6. 并行与数据规模：sharding、VDS、Parallel HDF5

### 6.1 推荐路线：sharding（分片）优先
当数据上到几十 GB / 上百 GB：
- 不建议单文件过大（备份、复制、损坏风险）
- 建议按 shard 切分：`train_000.h5 ... train_099.h5`

### 6.2 VDS：把多个 shard 映射成一个“虚拟大数据集”
优点：
- 训练侧像读单文件一样读
- 生产侧依旧分片易管理

### 6.3 Parallel HDF5：HPC/MPI 场景的正统方案
适合：
- 多进程共同写同一文件
- 强依赖 MPI-IO 与 collective 操作语义

工程侧常见折中：
- “写入阶段用并行或分布式生成 shard”
- “训练阶段多进程只读 shard”

---

## 7. 地球物理工程实践：从 SAC/miniSEED 到 HDF5

下面是一条适合 SWS / 三分量窗段学习的典型流水线：

1) 扫描目录与索引构建  
- event_id、station、分量文件、起止时间、可用性

2) 读取三分量并统一通道顺序  
- 固定为 `ZNE` 或 `ENZ`，并在文件级 attributes 记录 `component_order`

3) 预处理固化（建议全写入 attributes / meta）  
- demean / detrend / taper / bandpass / resample / normalize

4) 切窗与样本化  
- 得到 `X[i] = [3,Tw]`
- 保存 `t0/t1`、SNR、quality、pick 误差等到 meta

5) 写入 HDF5（扁平大数组 + meta 表）  
- `/X[N,3,Tw]`
- `/y[N,2]`（phi, dt）
- `/meta[N]`（event_id/station/baz/evdp/quality/…）






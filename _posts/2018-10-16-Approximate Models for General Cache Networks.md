---
title: Approximate Models for General Cache Networks
tags:	PaperRead
key: 20181016
---


## 综述

<!--more-->

## 基本信息
**Author:** Rosensweig, E. J. Kurose, J. Towsley, D.<br>
**Year:** 2010<br>
**Journal:** 2010 Proceedings Ieee Infocom<br>
**Url:** [click here](https://ieeexplore.ieee.org/document/5461936)

## 建模过程
### 模型定义与假设
$G=(V,E)$：缓存网络。其中$V=\{v_1,v_2,\dots,v_n\},E\subseteq{V\times{V}}$。

$F=\{f_1,f_2,\dots,f_N\}$：系统中的内容（文件）序列。

$S={s_1,s_2,\dots,s_m}$：服务器序列。每个内容存储在一个或多个服务器上；每个服务器连接一个或多个中间缓存节点。

$files(s)\subseteq{1,2,\dots,F},s\in{S}$：服务器中内容的序号，有：

$$|\bigcup_{s\in{S}}files(s)|=N$$

对于$\forall s \in S,v \in V$，有：

$$
\chi(s,v)=\begin{cases}
1,\quad s与v直接相连\\
0, \quad otherwise
\end{cases}
$$

简化起见，假设每个服务器仅连接一个缓存节点，定义为$v_s$。


$P=(v_{P_1},v_{P_2},\dots,v_{P_j})$：一系列有序节点组成的路径。

$P_i^v=(v_{P_1}=v,\dots,v_{P_j}=v_s)$：从节点v到存储有内容$f_i$的服务器$v_s$的最短路径。

假设外部对内容$f_i$请求到达符合泊松过程，速率为$\lambda_i$。

对于给定路径$P_i^v$，令$P_i^v[j]$表示路径上的第$j$个节点，给定两个节点$v,v^{\prime} \in V$，定义：

$$R(v,v^{\prime})=\{i:v^{\prime}=P_i^v[2]\}$$

表示$v$的下一跳是$v^{\prime}$、目的地是服务器$s$的请求内容序号$i$的集合。
>上句比较难翻译，附原文：This is the set of all request ids i for which v 0 is on the next hop from v along the shortest path to source s s.t. i ∈ files(s).

![](https://github.com/kanyuanzhi/kanyuanzhi.github.io/raw/master/assets/myimages/20181016/1.jpg)

$r_{i,v}$：节点$v$上对内容$f_i$的综合请求速率（直接请求$\lambda_{i,v}$+间接请求$m_{i,v}$），则：

$$r_{i,v}=\lambda_{i,v}+\sum_{v^{\prime}:i \in R(v^{\prime},v)}m_{i,v^{\prime}}$$

假设文内容从服务器下载至缓存节点的时间与相邻请求到达的间隔时间相比可忽略不计。

### 模型描述
$\vec{p_v}={p_{1,v},p_{2,v},\dots,p_{N,v}}$：稳态时在节点$v$处对内容请求的分布。

$\vec{q_v}={q_{1,v},q_{2,v},\dots,q_{N,v}}$：任意时刻各内容出现在节点$v$的概率。

[*a-LRU*](https://kanyuanzhi.github.io/2018/10/15/An-Approximate-Analysis-of-the-Lru-and-Fifo-Buffer-Replacement-Schemes.html)可看作是一个函数映射：

$$contents(\vec{p_v},|v|)=\vec{q_v}$$

假设请求流符遵循IRM模型，即$p_{i,v}$与之前的请求无关，则有：

$$m_{i,v}=r_{i,v} \cdot (1-q_{i,v})$$

综合以上，可得本文所提出的计算任意图节点丢失率的算法*a-NET*：

$$
\begin{align}
r_{i,v} &=\lambda_{i,v}+\sum_{v^{\prime}:i \in R(v^{\prime},v)}m_{i,v^{\prime}} \\
p_{i,v} &=\frac{r_{i,v}}{\sum_{j=1}^{N}r_{j,v}} \\
\vec{q_v} &=contents(\vec{p_v},|v|) \\
m_{i,v} &=r_{i,v} \cdot (1-q_{i,v})
\end{align}
$$

迭代计算上组式子，直到$m_{i,v}$的变化小于一个阈值。注意到迭代计算有可能不收敛，但实验中没有出现不收敛的情况。

### 模型分析




















---
title: Hierarchical Web Caching Systems Modeling Design and Experimental Results
tags:	PaperRead
key: 20181016
---


## 综述

<!--more-->

## 基本信息
**Author:** Hao Che, Ye Tung, and Zhijun Wang<br>
**Year:** 2002<br>
**Journal:** IEEE JOURNAL ON SELECTED AREAS IN COMMUNICATIONS<br>
**Url:** [click here](https://ieeexplore.ieee.org/document/1031903)

## 建模过程

### 模型假设
考虑两级缓存系统，根节点大小为$C_0$，$M$个叶节点的大小为$C_k(k=1,2,\dots,M)$，每个节点的缓存替换策略均为LRU。

假设：

1. 总请求到达叶节点$k$符合泊松过程，速率为$\lambda_k$；
2. 对节点$k$上的每个内容$i(i=1,2,\dots,N)$的请求是独立的，概率为$p_{ki}$，且$\sum_{i=1}^Np_{ki}=1$;
3. 每个内容的大小相同。

对节点$k$上单个内容$i$的请求速率为$\lambda_{ki}$，有：

$$\lambda_{ki}=p_{ki}\lambda_k$$

$p_{ki}$服从Zipf-like分布：

$$p_{ki}=K_k\frac{1}{[R_k(i)]^{z_k}}$$

其中$K_k$是归一化因子，$R_k(i)$是内容$i$在节点$k$出的流行度排名。

### 模型描述
缓存丢失率常用做评价缓存替换算法的性能。

$\lambda_{ki}$：对节点$k$上单个内容$i$的请求速率。

$\lambda_{ki}^0$：内容$i$从叶节点$k$到根节点的平均请求速率，即内容$i$在节点$k$的平均缓存丢失速率。

$\lambda_{0i}$：内容$i$在根节点的平均丢失速率。

缓存丢失率可描述如下：

$$\begin{align}
内容i在节点k的丢失率：\eta_{ki} &= \frac{\lambda_{ki}^0}{\lambda_{ki}} \\
内容i的在整个系统中的总丢失率：\eta_i &= \frac{\lambda_i^0}{\sum_{k=1}^M\lambda_{ki}} \\
节点k的总丢失率：\eta_k^0 &= \frac{\sum_{i=1}^N\lambda_{ki}^0}{\sum_{i=1}^N\lambda_{ki}} \\
全系统的总丢失率：\eta &= \frac{\sum_{i=1}^N\lambda_{0i}}{\sum_{k=1}^M\sum_{i=1}^N\lambda_{ki}}
\end{align}$$

可以发现，在计算缓存丢失率时，$\lambda_{ki}^0$与$\lambda_{0i}$是关键。

### 模型分析
$\lambda_{ki}^0$与$\lambda_{0i}$对应的平均丢失间隔为$T_{ki}=\dfrac{1}{\lambda_{ki}^0}$与$T_{0i}=\dfrac{1}{\lambda_{0i}}$































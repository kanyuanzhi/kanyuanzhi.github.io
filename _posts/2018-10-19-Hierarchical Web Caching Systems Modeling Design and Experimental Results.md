---
title: Hierarchical Web Caching Systems Modeling Design and Experimental Results
tags:	PaperRead
key: 20181019
---


## 综述

<!--more-->

## 基本信息
**Author:** Hao Che, Ye Tung, and Zhijun Wang<br>
**Year:** 2002<br>
**Journal:** IEEE JOURNAL ON SELECTED AREAS IN COMMUNICATIONS<br>
**Url:** [click here](https://ieeexplore.ieee.org/document/1031903)

## 建模过程

### 模型定义与假设
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

![image](https://github.com/kanyuanzhi/kanyuanzhi.github.io/raw/master/assets/myimages/20181019/1.jpg)

### 模型描述
缓存丢失率常用做评价缓存替换算法的性能。

$\lambda_{ki}$：对节点$k$上单个内容$i$的请求速率。

$\lambda_{ki}^0$：内容$i$从叶节点$k$到根节点的平均请求速率，即内容$i$在节点$k$的平均缓存丢失速率。

$\lambda_{0i}$：内容$i$在根节点的平均丢失速率。

缓存丢失率可描述如下：

$$\begin{align}
内容i在节点k的丢失率：\eta_{ki} &= \frac{\lambda_{ki}^0}{\lambda_{ki}} \\
内容i的在全系统中的总丢失率：\eta_i &= \frac{\lambda_i^0}{\sum_{k=1}^M\lambda_{ki}} \\
节点k的总丢失率：\eta_k^0 &= \frac{\sum_{i=1}^N\lambda_{ki}^0}{\sum_{i=1}^N\lambda_{ki}} \\
全系统的总丢失率：\eta &= \frac{\sum_{i=1}^N\lambda_{0i}}{\sum_{k=1}^M\sum_{i=1}^N\lambda_{ki}}
\end{align}$$

可以发现，在计算缓存丢失率时，$\lambda_{ki}^0$与$\lambda_{0i}$是关键。

### 模型分析
$\lambda_{ki}^0$与$\lambda_{0i}$对应的平均丢失间隔为$T_{ki}=\dfrac{1}{\lambda_{ki}^0}$与$T_{0i}=\dfrac{1}{\lambda_{0i}}$

#### 计算$T_{ki}$
注意到节点$k$处针对内容$i$的两个请求丢失的间隔时间由**一组独立同分布的随机变量序列$\{t_1,t_2,\dots,t_{n-1}\}$**加上**一个独立随机变量$t_n$**组成。

$t_i(i=1,2,\dots,n-1)$：针对内容$i$的两个命中请求的时间间隔；

$t_n$：最后一次缓存命中与下一次缓存丢失的时间间隔。

![image](https://github.com/kanyuanzhi/kanyuanzhi.github.io/raw/master/assets/myimages/20181019/2.jpg)

首先计算叶节点$k$处内容$i$的请求丢失间隔$t$的概率密度函数$f_{ki}^0(t)$。有：

$$t=\sum_{i=1}^{n-1}t_i+t_n$$

$t_i$的精确分布可以通过单个内容的组合泊松过程的分布得到，但计算量巨大。因此本文提出一个近似技术使得问题易于处理。

⭐️**定义$\tau_{ki}$为对内容$i$的两个相邻请求的最大间隔时间，且期间不包括缓存丢失。实际上$\tau_{ki}$是个随机变量，但在本文的近似中，假设：**

1. **对任意给定的$k$和$i$，$\tau_{ki}$是一个常数；**
2. **对任意给定的$k$，$\tau_{ki}$是与$i$有关的一个常数。**

根据以上近似，图二的过程可以简单描述为：

$$\begin{cases}
t_j\leq{\tau_{ki}},(j=1,2,\dots,n-1)\\
t_n\geq{\tau_{ki}}
\end{cases}$$

$\tau_{ki}$可以通过下式求得：

$$\sum_{j=1,j\neq{i}}^NP_{kj}(t<\tau_{ki})=C_k$$

其中$P_{kj}(t<\tau_{ki})$是叶节点$k$处对内容$i$请求间隔时间的累积分布。

现在$f_{ki}^0(t)$可以被表示为：

$$f_{ki}^0(t)=\sum_{n=1}^{\infty}f_{ki}(t|n)P_{ki}(t<\tau_{ki})^{n-1}(1-P_{ki}(t<\tau_{ki}))$$

其中：

$$P_{ki}(t<\tau_{ki})=1-e^{-\lambda_{ki}\tau_{ki}}$$

对$f_{ki}^0(t)$做Laplace变换，可得：

$$\phi_{ki}(s)=\frac{\lambda_{ki}e^{(-s-\lambda_{ki})\tau_{ki}}}{\lambda_{ki}e^{(-s-\lambda_{ki})\tau_{ki}}+s}$$

然后易得：

$$T_{ki}=-\frac{d\phi_{ki}(s)}{ds}{|_{s=0}}=\lambda_{ki}^{-1}e^{\lambda_{ki}\tau_{ki}}$$

$$\lambda_{ki}^0=T_{ki}^{-1}=\lambda_{ki}e^{-\lambda_{ki}\tau_{ki}}$$

⭐️**由此可以算得单个节点$k$内容$i$的缓存命中率：**

$$p_{hit}=1-\eta_{ki}=1-\frac{\lambda_{ki}^0}{\lambda_{ki}}=1-\frac{\lambda_{ki}e^{-\lambda_{ki}\tau_{ki}}}{\lambda_{ki}}=1-e^{-\lambda_{ki}\tau_{ki}}$$
























---
title: An Approximate Analysis of the Lru and Fifo Buffer Replacement Schemes
tags:	PaperRead
key: 20181015
---


## 综述
本文提出了一个近似模型，用以分析预测独立参考模型下在LRU和FIFO策略的缓存命中率，在LRU中，近似的命中率低于实际命中率。
<!--more-->

## 基本信息
**Author:** Dan, A. Towsley, D.<br>
**Year:** 1990<br>
**Journal:** 1990 Acm Sigmetrics Conference on Measurement and Modeling of Computer Systems<br>
**Url:** [click here](https://dl.acm.org/citation.cfm?id=98525)

## 建模过程
### 模型定义与假设
总大小为$D$的项目集合。

大小为$B$的缓存。

项目集合分割为$K$个分割，编号$k=1,2,\dots,K$，其中第$k$个分割包含$D_k$个项目，所以：
$D=\sum_{k=1}^K{D_k}$

任何请求的内容在第$k$个分割的概率为$\alpha_k,\sum_{k=1}^K{\alpha_k}=1$。

令$\{A_i\}_{i=1}^{\infty}$为一组独立同分布序列，其中$A_i$表示被请求的第$i$个项目来自的分割，根据假设，有：

$$Pr[a_i=k]=\alpha_k,k=1,2,\dots,K,i=1,2,\dots,\infty$$

$\underline{X_n}=(X_{1,n},\dots,X_{B,n})$是$n$个请求之后系统的状态。

$X_{i,n}$表示缓存中第$i$个位置被占用。

$X_{i,n}=k$当且仅当第$n$个请求之后来自$k$分割的一个项目占据了位置$i$。

定义$Y_{k,n}$为第$n$个请求之后在缓存中所有来自分割$k$的项目的总数，则：$Y_{k,n}=\sum_{i=1}^B{1(X_{i,n}=k)},k=1,\dots,K$

稳态时，$\underline{X}=\lim_{n\to\infty}{\underline{X_n}},Y_k=\lim_{n\to\infty}{Y_{k,n}},1\leq{k}\leq{K}$

$k$分割的命中率为：$h_k=\dfrac{E[Y_k]}{D_k}$

### 模型描述

缓存栈顶部为位置1，底部为位置$B$，第$j$个最常访问的项目在位置$j,j=1,\dots,B$。令$\pi(\underline{x})=Pr[\underline{X}=\underline{x}]$，其中$\underline{x}=(x_1,\dots,x_B)\in{S}$，$S$是缓存可能状态的序列集合，有：

$$S=\{\underline{x}:\sum_{i=1}^B1(x_i=k)\leq{D_k},1\leq{k}\leq{K},\sum_{k=1}^K\sum_{i=1}^B1(x_i=k)=B\}$$

每个状态的概率满足：

$$\begin{split}
\pi(\underline{x})={}&\sum_{j=1}^B\pi(x_2,\dots,x_j,x_1,x_{j+1},\dots,x_B)\cdot\alpha_{x_1}{}\\
&+\sum_{l\in{k:\sum_{i=2}^{B}1(x_i=k)<D_k}}\pi(x_2,\dots,x_B,l)\cdot\alpha_{x_1}{}\cdot(D_{x_1}-\sum_{i=2}^{B}1(x_i=x_1)-\delta_{l,x_1})/D_{x_1}
\end{split} \quad (1)$$

**⚠️此公式加号左边是$x_1$在前一状态中，且不在栈底部的概率；右侧是$x_1$不在前一状态中或在前一状态中的栈底部的概率。**

其中$\delta_{x,y}=1$如果$x=y$否则为0。

令$Y_k(j)=1(X_j=k)$表示来自分割k的项目是否在缓存的第$j$个位置，有$Y_k=\sum_{j=1}^BY_k(j)$。令$p_k(j)=Pr[X_k(j)=1]$，显然$p_k(1)$是最后请求的项目来自分割$k$的概率，且有$p_k(1)=\alpha_k$。


令$r_k(j-1)$表示当一个请求到来后缓存中有项目从位置$j-1$移到$j$的条件下，该项目是来自分割$k$的概率（条件概率）。我们**近似**稳态时候的概率$p_k(j)=r_k(j-1)$，令$b_k(j)$表示缓存中前$j$个位置中的项目来自分割$k$的平均数量，有：

$$b_k(j)=\sum_{l=1}^jE[X_l(j)]=\sum_{l=1}^jp_k(l),j=1,2,\dots,B \quad (2)$$

$$p_k(j)=r_k(j-1)=\dfrac{(\alpha_k[1-\frac{b_k(j-1)}{D_k}])^+}{r(j-1)},j=1,2,\dots,B-1;k=1,\dots,K \quad (3)$$

其中

$$r(j-1)=\sum_{i=1}^K(\alpha_i[1-\dfrac{b_i(j-1)}{D_i}])^+,j=1,2,\dots,B-1$$

$$(a)^+=\begin{cases}a,\quad a>0\\
0,\quad otherwise\end{cases}$$

迭代计算(2)(3)两式**（初始解$p_k(1)=\alpha_k$，一直计算到$b_k(B)$）**，可计算分割$k$的缓存命中率：

$$h_k=\dfrac{E[Y_k]}{D_k}=\dfrac{b_k(B)}{D_k}$$

### 模型分析

**⚠️由于公式(1)中的概率$\pi$并不容易求得，所以近似用$r_k(j-1)$表示，即用缓存前$j-1$个位置中分割来自$k$的平均项目数代替真实的项目分布。**

在近似计算过程中，某些分割$i$的命中率$h_i$甚至大于1，为避免这种情况，将$b_i(j)$的最大值限制为$D_i$，这造成近似命中率$p_i(j)$**小于**真实值（由公式(3)可以看出），且为保证总命中率$\sum_{l=1}^Kp_l(j)=1$，除$p_i(j)$以外的命中率会重新分布（因为强制固定$b_i(j)\leq{D_i}$）。

在迭代过程中，只要$b_i(j)=D_i$，随后计算步骤里，位置$l(l>j)$上来自分割$i$的项目不再考虑，即$p_i(l)=0$。







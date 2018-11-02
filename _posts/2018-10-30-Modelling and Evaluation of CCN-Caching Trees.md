---
title: Modelling and Evaluation of CCN-Caching Trees
tags:	PaperRead
key: 20181030
---


## 综述

<!--more-->

## 基本信息
**Author:** Ioannis Psaras, Richard G. Clegg, Raul Landa, Wei Koong Chai, and George Pavlou<br>
**Year:** 2011<br>
**Journal:** NETWORKING<br>
**Url:** [click here](https://link.springer.com/chapter/10.1007/978-3-642-20757-0_7)

## 单节点建模过程

### 模型定义与假设
$\lambda$：使得内容PoI来到缓存顶部的请求的到达速率（对内容PoI请求的到达速率）。

$\mu$：使得内容PoI下移的请求的到达速率。

$\pi=(\pi_1,\pi_2,\dots,\pi_{N+1})$：链的均衡概率，或者PoI在该位置花费的时间比例，有：

$$\pi_i=\frac{\lambda}{\mu}\big[\frac{\mu}{\mu+\lambda}\big]^i$$

$$\pi_{N+1}=\big[\frac{\mu}{\mu+\lambda}\big]^N$$

$\pi_{N+1}$表示PoI不在缓存中的时间所占比重。













---
title: Modeling Data Transfer in Content-Centric Networking
tags:	PaperRead
key: 20180128
---


## 综述

<!--more-->

## 基本信息
**Author:** Giovanna Caroﬁglio, Massimo Gallo, Luca Muscariello and Diego Perino.<br>
**Year:** 2011<br>
**Journal:** Proceedings of the 23rd international teletraffic congress<br>
**Url:** [click here](https://dl.acm.org/citation.cfm?id=2043487)

## 模型描述

### 假设与记号

![](https://github.com/kanyuanzhi/kanyuanzhi.github.io/raw/master/assets/myimages/20190128/1.jpg)

- $M$个不同内容平均分成$K$个不同流行度。流行度为$k$的内容被请求的概率是$q_k, k \geq 1$，流行度服从Zipf分布。
- 内容分成不同大小的块，用$\sigma$表示内容的平均大小（以块的数量记，块的大小相同）。
- 网络中每个节点拥有一个可以存储$x$个块的缓存。
- 定义流行度为$k$的内容的虚拟往返时延$VRTT_k$,即在稳定状态下发送块请求到接收块回复的平均时间：

$$VRTT_k=\sum^{N}_{i=1}R_i(1-p_k(i))\prod^{i-1}_{j=1}p_k(j),\quad k=1,\dots,K$$





































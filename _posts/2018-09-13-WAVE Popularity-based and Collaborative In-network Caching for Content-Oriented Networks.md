---
title: WAVE Popularity-based and Collaborative In-network Caching for Content-Oriented Networks
tags:	PaperRead
key: 20180913
---


## 综述
本文对近年来国际上在该领域的主要研究成果进行了回顾与总结，分析了ICN缓存网络的新特征带来的挑战，综述了ICN缓存网络中若干主要问题的研究现状，包括缓存大小规划、应用无关的缓存空间共享机制、缓存决策策略、网络内置缓存对象的可用性以及缓存网络的理论模型等，并进行了深入的分析和对比，同时指出可能的设计选择空间。
<!--more-->


## 基本信息
**Author:** Kideok Cho, Munyoung Lee, Kunwoo Park, Ted Taekyoung Kwon, Yanghee Choi, Sangheon Pack<br>
**Year:** 2012<br>
**Journal:** 2012 Ieee Conference on Computer Communications Workshops<br>
**Url:** [click here](https://ieeexplore.ieee.org/abstract/document/6193512/)

## WAVE特征总结
1. 基于流行度（Popularity-based）：WAVE依据内容的流行度决定缓存块的数量。当访问量增加时，WAVE指数式增加缓存快的数量并更广泛地传播它们。
2. 简单（Simple）：不需要先验信息，缓存决策只依赖每个文件的两个计数器；可用于所有路由器架构，因为只需要知道请求从哪里来。
3. 分布式（Decentralized）：每个路由器独立做出缓存决定。
4. 可逐步部署（Incrementally deployable）

## 假设说明
上游路由器可向下游路由器建议是否缓存内容，通过在块中添加一个**缓存建议标志位**（位于数据包的头部）。如果下游路由器缓存空间已满或者有其独立的缓存策略，它将忽略缓存建议并将块继续转发至下游路由器。

## 具体操作过程
![image](https://github.com/kanyuanzhi/kanyuanzhi.github.io/raw/master/assets/myimages/20180913/1.jpg)

1. H1向源服务器请求1个文件（包括7个块）；
2. 源服务器向H1返回7个块，由于是对文件的第一次请求，所以源服务器标记1号块为**需缓存块**（缓存建议标志位设置为2）。路由器A仅缓存1号块，同时缓存标志位重新设置为0（为避免下游路由器B，C缓存1号块）。
3. 

## 块缓存算法

### 缓存什么（如何分发块）

### 替换什么

### 在哪缓存






















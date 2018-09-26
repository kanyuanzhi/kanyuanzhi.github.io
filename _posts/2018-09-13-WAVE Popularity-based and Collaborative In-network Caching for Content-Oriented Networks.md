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
1. 基于流行度（Popularity-based）：WAVE依据内容的流行度决定缓存块的数量。当访问量增加时，WAVE**指数式**增加缓存快的数量并更广泛地传播它们。
2. 简单（Simple）：不需要先验信息，缓存决策只依赖每个文件的两个计数器；可用于所有路由器架构，因为只需要知道请求从哪里来。
3. 分布式（Decentralized）：每个路由器独立做出缓存决定。
4. 可逐步部署（Incrementally deployable）

## 假设说明
上游路由器可向下游路由器建议是否缓存内容，通过在块中添加一个**缓存建议标志位**（位于数据包的头部）。如果下游路由器缓存空间已满或者有其独立的缓存策略，它将忽略缓存建议并将块继续转发至下游路由器。

## 具体缓存过程⭐️
![image](https://github.com/kanyuanzhi/kanyuanzhi.github.io/raw/master/assets/myimages/20180913/1.jpg)

以下序号分别与上图中序号一一对应：

1. H1向源服务器请求1个文件（包括7个块）；
2. 源服务器向H1返回7个块，由于是对文件的第一次请求，所以源服务器标记1号块为**需缓存块**（缓存建议标志位设置为2）。路由器A仅缓存1号块，同时缓存标志位重新设置为0（为避免下游路由器B，C缓存1号块）；
3. 剩余块(2-7号块)也将转发至H1，且不在中间路由器中缓存；
4. H2向源服务器请求相同的文件（包括7个块），注意到1号块将由路由器A响应，2-7号块由源服务器响应；
5. 路由器A中已缓存1号块，所以1号块直接由路由器A转发，并将其缓存建议标志位设置为1，1号块到达路由器B时，B缓存1号块，并将其缓存建议标志位重新设置为0；
6. 源服务器将接下来两个块（2，3号块）的缓存建议位设置为1，在路由器A中缓存，然后路由器A将2，3号块的缓存建议位重新设置为0；
7. 其余块（4-7号块）将转发至H2，且不在中间路由器中缓存；
8. H3向源服务器请求相同的文件，将由路由器A，B和源服务器分别响应，并重复上述过程；
9. 1号块在路由器C中；
10. 2，3号块在路由器B中；
11. 4，5，6，7号块在路由器A中。

## 块缓存算法
WAVE依据文件流行度动态调整缓存块的数量，同时为了减少缓存更新的产生的负载，WAVE采取了保守的方法来选择要被缓存的块。

### 缓存什么（如何分发块）

#### 参数说明
缓存块的数目由**块标记窗口(chunk marking window CMW）**决定，其随着文件请求数的增加而指数型增加。

CMW的大小由以下两个参数决定：

- 基准$x$，决定块缓存速度；
- 当前状态$n$，也叫做CMW状态。

因此，对于给定$x,n$，当前CMW的范围从$\sum_{j=0}^{n-1}x^j+1$到$\sum_{j=0}^{n}x^j$，如果所有的缓存块发生转移，则$n$加1。对于不流行的内容，其$n$很小，从而避免分发；对于流行的内容，由于CMW呈指数型增长，WAVE可以分发得很快。

对于以上操作，路由器对每个接口的每个文件维护两个计数器：CMW状态$n$和最近一个缓存建议标志位被设置的块的序号$cached$。

#### 算法过程
![Alt Image Text](https://github.com/kanyuanzhi/kanyuanzhi.github.io/raw/master/assets/myimages/20180913/2.png "algorithm")

### 替换什么

### 在哪缓存






















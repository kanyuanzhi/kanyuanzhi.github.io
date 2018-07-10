---
title: A survey of Web cache replacement strategies
tags:	PaperRead
key: 20180710
---


# 综述
本文全面介绍了已有的（2003年）缓存替换策略,并提出一种新的分类方法将这些缓存替换策略归类，在归类的基础上分析比较各类之间的优缺点。进一步，本文讨论了缓存策略对于代理缓存服务器的重要性，最后展望了未来的研究方向（在2018看这些还是很有前瞻性的）。
<!--more-->

# 基本信息
**Author:** Podlipnig, S., Boszormenyi, L.<br>
**Year:** 2003<br>
**Journal:** Acm Computing Surveys<br>
**Url:** [click here](http://apps.webofknowledge.com/InboundService.do?customersID=ResearchSoft&mode=FullRecord&IsProductCode=Yes&product=WOS&Init=Yes&Func=Frame&DestFail=http%3A%2F%2Fwww.webofknowledge.com&action=retrieve&SrcApp=EndNote&SrcAuth=ResearchSoft&SID=6E2nE3hfIggQQhgCdB9&UT=WOS%3A000187181700002)

# 缓存替换策略归类

## Recency-Based Strategies
使用**近因**（recency）作为划分点。最典型代表是LRU策略，基于请求流中可以观察到的**引用局部性**(locality of reference，表征了基于过去的到达流来预测未来的到达流的能力)。有两种局部性：

1. 时间局部性：是指短时间内对同一内容的重复请求。意味着最近被访问的内容更有可能在在将来被访问。

2. 空间局部性：是指访问模式（access patterns）。对一些内容的访问意味着将来会对其他某些内容访问。

Recency-Based Strategies主要是针对时间局部性。

### 实例
- LRU
- LRU-Threshold
- Pitkow/Recker's strategy
- SIZE
- LRU-Min
- EXP1
- Value-Aging
- HLRU
- PSS
- LRU-LSC
- Partitioned Caching

> 具体算法可查阅相关论文

### 优点
1. 网络的请求流经常表现出时间局部性的特点，所以这些策略比较适合；
2. 可以很好适应工作负载的变化（比如出现新的热点内容）；
3. 易于部署，执行快速。

### 缺点
1. 简单的LRU及其变体没有很好地将recency与内容大小结合起来。SIZE策略有考虑内容大小，但替换算法过于激进。PSS结合得不错。
2. 没有考虑频率（frequency）的影响。频率在更加静态的环境中是重要指标。

## Frequency-Based Strategies
使用**频率**作为划分点。典型代表为LFU策略。它们基于这样一个事实：不同的网络内容具有不同的流行度，这些不同的流行度导致不同的请求频率。Frequency-Based Strategies跟踪这些流行度并基于此对未来作出预测。LFU可以以两种形式加以使用：

1. Perfect LFU：统计对内容i的所有请求数。空间花销大。
2. In-Cache LFU：仅统计对被缓存内容的请求数。空间花销小。

以下均考虑In-Cache LFU。

### 实例
- LFU
- LFU-Aging
- LFU-DA
- α-Aging
- swLFU

### 优点
1. 考虑到请求频率的影响，适用于静态环境，即流行的内容在一定时间段内并不经常发生变化。

### 缺点
1. 复杂度较高；
2. 易形成缓存污染，对于工作负载的动态变化，频率的统计过于静态，造成当前并不流行但过去流行的内容会长时间贮存在缓存中。


## Recency/Frequency-Based Strategies

结合recency与频率。

### 实例
- SLRU (Segmented LRU)
- Generational Replacement
- LRU*
- LRU-Hot
- HYPER-G
- CSS(Cubic Selection Scheme)
- LRU-SP

### 优点
1. 如果设计的好，可以克服Recency-Based与Frequency-Based各自的缺点。

### 缺点
1. 复杂度较高，且都没有考虑内容大小的影响。

## Function-Based Strategies
使用通用的函数来计算每个内容的价值，根据这些价值决定缓存中的替换内容。

### 实例
- GD(Greedy Dual)-Size
- GDSF
- GD*
- Server-assisted cache replacement
- TSP(Taylor Series Prediction)
- Bolot/Hoschka’s strategy
- MIX
- M-Metric
- HYBRID
- LNC-R-W3
- LRV
- LUV
- LR(Logistic Regression)-Model

### 优点
1. 不需要假设固定的因素（recency，频率）组合或者固定的数据结构的使用，通过合理选择加权参数，可以优化任意性能指标；
2. 考虑了许多处理不同工作负载情况的因素。

### 缺点
1. 选择合适的参数比较困难。有些基于此的策略认为参数因来自于对网络的跟踪学习，但网络的工作负载在不断变化，需要自适应的参数设置，这又增加了策略的复杂度；
2. 使用延时（latency）作为价值计算（有些基于此的策略的价值函数与网络延时有关）可能带来一些问题（如评价不准确），因为延时不稳定，影响它的因素太多。

## Randomized Strategies
随机决定缓存中的替换内容。

### 实例
- RAND
- HARMONIC
- LRU-C and LRU-S
- Randomized replacement with general value functions 

### 优点
1. 不需要特定的数据结构来实现插入与删除内容；
2. 易于实现。

### 缺点
1. 评估起来比较麻烦，相同的实验多次进行会产生不同的结果。

# 缓存策略的重要性

























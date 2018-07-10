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

### 此类下的策略
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
2. 可以很好适应内容流量的变化（比如出现新的热点内容）；
3. 易于部署，执行快速。

### 缺点
1. 简单的LRU及其变体没有很好地将recency与内容大小结合起来。SIZE策略有考虑内容大小，但替换算法过于激进。PSS结合得不错。
2. 没有考虑频率（frequency）的影响。频率在更加静态的环境中是重要指标。

## Frequency-Based Strategies
使用**频率**作为划分点。典型代表为LFU策略。它们基于这样一个事实：不同的网络内容具有不同的流行度，这些不同的流行度导致不同的请求频率。Frequency-Based Strategies跟踪这些流行度并基于此对未来作出预测。LFU可以以两种形式加以使用：

1. Perfect LFU：统计对内容$i$的所有请求数。空间花销大。
2. In-Cache LFU：仅统计对被缓存内容的请求数。空间花销小。

以下均考虑In-Cache LFU。

### 此类下的策略
- LFU
- LFU-Aging
- LFU-DA
- α-Aging
- swLFU

### 优点
1. 考虑到请求频率的影响，适用于静态环境，即流行的内容在一定时间段内并不经常发生变化。

### 缺点
1. 算法实现起来较复杂；
2. 易形成缓存污染，对于网络流量的动态变化，频率的统计过于静态，造成当前并不流行但过去流行的内容会长时间贮存在缓存中。



























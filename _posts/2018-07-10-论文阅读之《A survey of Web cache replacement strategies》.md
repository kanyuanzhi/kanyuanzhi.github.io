---
title: A survey of Web cache replacement strategies
tags:	PaperRead
key: 20180710
---


## 综述
本文全面介绍了已有的（2003年）缓存替换策略,并提出一种新的分类方法将这些缓存替换策略归类，在归类的基础上分析比较各类之间的优缺点。进一步，本文讨论了缓存策略对于代理缓存服务器的重要性，最后展望了未来的研究方向（在2018看这些还是很有前瞻性的）。
<!--more-->

## 基本信息
**Author:** Podlipnig, S., Boszormenyi, L.<br>
**Year:** 2003<br>
**Journal:** Acm Computing Surveys<br>
**Url:** [click here](http://apps.webofknowledge.com/InboundService.do?customersID=ResearchSoft&mode=FullRecord&IsProductCode=Yes&product=WOS&Init=Yes&Func=Frame&DestFail=http%3A%2F%2Fwww.webofknowledge.com&action=retrieve&SrcApp=EndNote&SrcAuth=ResearchSoft&SID=6E2nE3hfIggQQhgCdB9&UT=WOS%3A000187181700002)

## 缓存替换策略归类

### Recency-Based Strategies
使用**近因**（recency）作为划分点。最典型代表是LRU策略，基于请求流中可以观察到的**引用局部性**(locality of reference，表征了基于过去的到达流来预测未来的到达流的能力)。有两种局部性：

1. 时间局部性：是指短时间内对同一内容的重复请求。意味着最近被访问的内容更有可能在在将来被访问。

2. 空间局部性：是指访问模式（access patterns）。对一些内容的访问意味着将来会对其他某些内容访问。

Recency-Based Strategies主要是针对时间局部性。

#### 实例
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

#### 优点
1. 网络的请求流经常表现出时间局部性的特点，所以这些策略比较适合；
2. 可以很好适应工作负载的变化（比如出现新的热点内容）；
3. 易于部署，执行快速。

#### 缺点
1. 简单的LRU及其变体没有很好地将recency与内容大小结合起来。SIZE策略有考虑内容大小，但替换算法过于激进。PSS结合得不错。
2. 没有考虑频率（frequency）的影响。频率在更加静态的环境中是重要指标。

### Frequency-Based Strategies
使用**频率**作为划分点。典型代表为LFU策略。它们基于这样一个事实：不同的网络内容具有不同的流行度，这些不同的流行度导致不同的请求频率。Frequency-Based Strategies跟踪这些流行度并基于此对未来作出预测。LFU可以以两种形式加以使用：

1. Perfect LFU：统计对内容i的所有请求数。空间花销大。
2. In-Cache LFU：仅统计对被缓存内容的请求数。空间花销小。

以下均考虑In-Cache LFU。

#### 实例
- LFU
- LFU-Aging
- LFU-DA
- α-Aging
- swLFU

#### 优点
1. 考虑到请求频率的影响，适用于静态环境，即流行的内容在一定时间段内并不经常发生变化。

#### 缺点
1. 复杂度较高；
2. 易形成缓存污染，对于工作负载的动态变化，频率的统计过于静态，造成当前并不流行但过去流行的内容会长时间贮存在缓存中。


### Recency/Frequency-Based Strategies

结合recency与频率。

#### 实例
- SLRU (Segmented LRU)
- Generational Replacement
- LRU*
- LRU-Hot
- HYPER-G
- CSS(Cubic Selection Scheme)
- LRU-SP

#### 优点
1. 如果设计的好，可以克服Recency-Based与Frequency-Based各自的缺点。

#### 缺点
1. 复杂度较高，且都没有考虑内容大小的影响。

### Function-Based Strategies
使用通用的函数来计算每个内容的价值，根据这些价值决定缓存中的替换内容。

#### 实例
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

#### 优点
1. 不需要假设固定的因素（recency，频率）组合或者固定的数据结构的使用，通过合理选择加权参数，可以优化任意性能指标；
2. 考虑了许多处理不同工作负载情况的因素。

#### 缺点
1. 选择合适的参数比较困难。有些基于此的策略认为参数因来自于对网络的跟踪学习，但网络的工作负载在不断变化，需要自适应的参数设置，这又增加了策略的复杂度；
2. 使用延时（latency）作为价值计算（有些基于此的策略的价值函数与网络延时有关）可能带来一些问题（如评价不准确），因为延时不稳定，影响它的因素太多。

### Randomized Strategies
随机决定缓存中的替换内容。

#### 实例
- RAND
- HARMONIC
- LRU-C and LRU-S
- Randomized replacement with general value functions 

#### 优点
1. 不需要特定的数据结构来实现插入与删除内容；
2. 易于实现。

#### 缺点
1. 评估起来比较麻烦，相同的实验多次进行会产生不同的结果。

## 缓存策略的重要性
### 重要性
此节的前半部分主要为评论当时关于缓存策略不重要的论点。
> Nowadays cache replacement is often considered less important [Krishnamurthy and Rexford 2001; Rabinovich and Spatscheck 2002].

这种论点主要基于以下几点：

1. 存储器件的价格逐渐降低使得可以有足够大的空间来缓存内容；
2. 内容可以被缓存的流量比例在下降；
3. 已经存在足够好的缓存算法了，如GD-Size；
4. 内容一段时间内的不断变化降低了用大缓存存储它们更长时间的价值。

作者针对以上几点评价：

1. （1）有些代理缓存服务器不具有部署大容量存储器件的条件；（2）新的大尺寸网络内容出现；
2. 不可缓存的内容大多数是具有即时性的内容，需要加强对缓存一致性的研究；
3. 足够好的缓存替换策略。关于“好”有不同指标，同一策略难以在不同的网络工作负载下均表现优异。经常使用的指标有：命中率（Hit-rate），字节命中率（Byte-hit-rate），延时节约率（Delay-savings-ratio）（由于延时的不确定性，次项指标很少使用）；
4. 见应用场景部分。

### 缓存策略的应用场景
#### Product-Based Approaches
基于产品的场景大部分使用LRU及其变体，因为易于部署，算法简单。工业界不认为缓存的大小是影响缓存性能的因素，在足够大的缓存中，简单的LRU就可以实现很好的效果。

#### Considerations Based on Statistical Properties
##### 有两个典型的工作负载因素影响着缓存替换算法的性能：

1. 内容大小：网络内容大小通常用重尾分布（heavy-tailed distribution），轻尾分布与重尾分布结合，或者是对数正态分布（log-normal dis- tribution）来建模。
2. 时间局部性：时间局部性主要包括流行度与相关度。
	- 内容流行度可以用Zipf分布或者请求流中的经验熵（empirical entropy）来刻画；
	- 时间相关性关注于对于给定内容的引用（缓存被命中）被其他内容的引用所分开的行为（*可以理解为：两个对相同内容的缓存命中之间夹杂着对其他内容的缓存命中*）。

##### 考虑到这些因素，一些选择缓存替换策略的规则可以描述如下：

1. 缓存替换策略应该包括一些内容大小的差异化处理，这些处理会使策略变得复杂。因此如果代理有较少的CPU资源，，一个简单的做法是将缓存划分为一系列小的缓存块，每一块仅储存具体尺寸范围的缓存，在每一个缓存快中再应用一下简单的缓存替换算法；
2. 其他内容价值（除了内容大小）的计算也会增加代理负担，所以对于CPU资源有限的代理应该使用简单的诸如LRU及其变体的策略；
3. 如果缓存空间对于全网内容而言非常小，则缓存替换策略应该优先考虑recency而不是频率，因为短期内的时间相关性的存在（**时间相关性--->LRU**）；
4. 如果缓存空间足够大，则应该考虑使用长期频率因素，因为内容的流行度对时间局部性的贡献更大(**时间流行性--->LFU**)。此外频率应该总是与过期机制相结合以减少缓存污染。

### 缓存管理的可选模型
1. 每隔一段时间根据历史信息同一更换缓存内容。仅适用明确定义（提前知道）内容数量的特定系统，例如网络服务器。
2. 基于TTL，缓存内容统一设置生存时间，或者请求次数大于n才会缓存，缓存后设置生存时间。

## 未来研究方向
### 自适应缓存替换
缓存替换策略可以根据不同网络环境、工作负载自主调整替换方式。function-based replacement strategies在这方面具有很大潜力，其函数参数可以用机器学习的方法不断从网络变化中学习并及时更新。但需要注意应避免不必要的自适应策略变化（有可能导致性能下降），以及在两种策略切换时避免决策错误。这种自适应的替换策略还会带来其他形式的复杂性。比如缓存必须为所有不同的替换策略管理相应的数据与数据结构。

未来的研究应关注：

- 智能自适应策略的设计与评价；
- 在真实网络环境中（高负载，突发请求序列等）验证这些策略的应用价值。


### 协作缓存替换
缓存集群中个节点的互相合作，即放置与替换策略相结合。

### 缓存替换与缓存一致性的结合
缓存一致性有两个方法：

1. 验证（validation）：客户端与源服务器通讯来验证缓存的有效性，这通常是定期通讯一次，因此是弱一致性；
2. 非验证（invalidation）：源服务器主动告知客户端缓存内容是否已更改，是强一致性。

不同的一致性策略对缓存命中率或增加或降低。

### 多媒体缓存替换

略过

### 差异化缓存替换
传统的缓存替换策略并不支持QoS（为指定的网络通信提供更好的服务能力）。缓存可以看做是一种尽力而为的服务。不同程度的Qos可以描述为：

1. **Best-effort caching**
2. **Differrent caching：**不提供任何服务保障，但对缓存中的一些特定内容，给予替换时的特殊照顾（不发生在放置阶段）；
3. **Push caching：**与Differrent caching相反，内容提供者标注某内容当其将被缓存时，随即推出缓存，即不缓存；
4. **Replication：**允许内容提供者对其内容可以选择自定义的放置与替换策略。对应于有保证的服务，例如提供生存时间或者延时之类的性能保证，但需要资源预留与准入控制，会增加额外的复杂性，且不具备可扩展性。

以下将重点关注Differrent cachin，即差异化缓存，可以分为如下两类：

1. 将差异化原理从网络域转移到缓存域，服务器标记特定对象，缓存利用标记信息进行区分替换；
2. 通过反馈控制进行区分，不依赖服务器标记信息。

对于第一条面临如下问题：

1. 任何形式的差异化处理都需要相应的定价策略，否则内容提供者将平等对待所有内容。这些定价策略在网络域中有很多研究，在缓存域中很少。
2. 与网络域相比，内容提供者在缓存域中必须对每一个内容进行标记，有时很难提前获取内容的重要性，对其标记也很难进行。

因此第二条通过反馈控制进行区分看起来更适合缓存域，代理缓存节点可以直接利用本地的实际的工作负载获得反馈参数。但这也会增加系统的额外开销，因此如何用更简单的技术实现差异化也是一个有趣的研究方向。

[*Service differentiation in Web caching and content distribution*](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.11.7579) 关于这方面给出一个例子，LRU策略被增强以考虑不同类别的内容。

## 结论
本文对Web缓存提供的缓存替换策略进行了详尽的调查。给出了这些替换策略的简单分类方案，并用于对所述替换策略的描述和一般性评论。虽然缓存替换被认为是一个已解决的问题，但我们发现仍有许多领域可供研究。






















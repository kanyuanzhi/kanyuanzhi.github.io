---
title: ndnSIM仿真平台使用之用户Consumer类
tags:	ndnSIM
key: 20200401
---

本文首先介绍ndnSIM提供的不同用户类的参数设置与使用方式，然后介绍用户类的核心方法`SendPacket()`以及如何初始化自定义的标签，最后介绍如何创建自定义的用户类。

<!--more-->

版本信息如下：

操作系统：Ubuntu 16.04 <br>ndnSIM：ndnSIM-2.1<br>
ns-3-dev：ns-3.23-dev-ndnSIM-2.1<br>

用户类用于生成兴趣包，从而驱动整个仿真程序。各个用户类的继承关系及特点：

|          类名          |         继承关系          |               特点                |
| :--------------------: | :-----------------------: | :-------------------------------: |
|        Consumer        | 用户类的基类，继承于APP类 |            发出兴趣包             |
|      ConsumerCbr       |      继承于Consumer       |       按一定间隔发出兴趣包        |
| ConsumerZipfMandelbrot |     继承于ConsumerCbr     | 按*Zipf-Mandelbrot*分布发出兴趣包 |
|     ConsumerWindow     |      继承于Consumer       |      基于滑动窗口发出兴趣包       |
|    ConsumerBatches     |      继承于Consumer       |  在指定时间发出一定数量的兴趣包   |

⭐️**在仿真实验中最常使用的是ConsumerZipfMandelbrot。**

## Consumer

Consumer类是所有用户类的基类，用于发出兴趣包。参数设置中我们仅介绍常用的参数。

### 设置参数

- Prefix：请求内容的前缀。

```c++
consumerHelper.SetPrefix("/prefix")
// 或 consumerHelper.SetAttribute("Prefix", StringValue("/prefix"));
```

- StartSeq：请求内容初始序号。

```c++
consumerHelper.SetAttribute("StartSeq", IntegerValue(0));
```

- LifeTime：兴趣包的生存时间（超过此时间未被响应的兴趣包将被丢弃或重发）。

```c++
consumerHelper.SetAttribute("LifeTime", StringValue("2s"));
```

其他用户类中也可设置以上参数

### 使用

```c++
ndn::AppHelper consumerHelper("ns3::ndn::Consumer");
consumerHelper.SetAttribute(...);
consumerHelper.Install(consumerNodes); // 在指定节点上安装用户
```

## ConsumerCbr

ConsumerCbr类按一定时间间隔发出不重复的兴趣包，所以若实验中仅有一个用户，那么缓存中的内容永远不会被响应。

### 设置参数

- Frequency：发出兴趣包的频率。

```c++
consumerHelper.SetAttribute("Frequency", StringValue("1.0"));
```

- Randomize：发出间隔的分布方式，可选值有：
  - `"none"`：间隔时间为常数，即速率为固定值；
  - `"uniform"`：间隔时间为(0,1/Frequency)的均匀分布；
  - `"exponential"`：间隔时间为均值是1/Frequency的指数分布。

```c++
consumerHelper.SetAttribute("Randomize", StringValue("none")); // 或"uniform"或"exponential"
```

- MaxSeq：请求内容的最大序号，超过该最大序号后用户将不再发出兴趣包。

```c++
consumerHelper.SetAttribute("MaxSeq", IntegerValue(std::numeric_limits<uint32_t>::max())); // 默认为整型的最大值
```

### 使用

```c++
ndn::AppHelper consumerHelper("ns3::ndn::ConsumerCbr");
consumerHelper.SetAttribute(...);
consumerHelper.Install(consumerNodes); // 在指定节点上安装用户
```

## ConsumerZipfMandelbrot

ConsumerZipfMandelbrot类继承于ConsumerCbr类，按Zipf-Mandelbrot分布发出兴趣包，这是ndnSIM仿真实验中最常使用的用户类。

### 设置参数

- Frequency：发出兴趣包的频率，同ConsumerCbr类。

```c++
consumerHelper.SetAttribute("Frequency", StringValue("1.0"));
```

- NumberOfContents：内容总数。

```c++
consumerHelper.SetAttribute("NumberOfContents", StringValue("100"));
```

- q：用以控制峰值的左右偏移，一般设置为0。

```c++
consumerHelper.SetAttribute("q", StringValue("0"));
```

- s：Zipf参数，用以控制流行内容的集中程度，一般取$[0.2,1.5]$。

```c++
consumerHelper.SetAttribute("s", StringValue("0.7"));
```

### 使用

```c++
ndn::AppHelper consumerHelper("ns3::ndn::ConsumerZipfMandelbrot");
consumerHelper.SetAttribute(...);
consumerHelper.Install(consumerNodes); // 在指定节点上安装用户
```

## ConsumerWindow

ConsumerWindow类是一个基于滑动窗口的用户类，可发出变化速率的兴趣包。

### 设置参数

- Window：初始窗口大小，即无需等待数据包返回就能发送出的初始兴趣包数（未被响应的兴趣包数）。

```c++
consumerHelper.SetAttribute("Window", StringValue("1"));
```

- PayloadSize：数据包的平均大小。

```c++
consumerHelper.SetAttribute("PayloadSize", UintegerValue(1040));
```

- Size：请求的数据总量，单位MB，当总量达到设置值时用户不再发出兴趣包。默认为-1，表示不限制数据总量。

```c++
consumerHelper.SetAttribute("Size", DoubleValue(-1));
```

- MaxSeq：请求内容的最大序号，超过该最大序号后用户将不再发出兴趣包。只有Size为-1时MaxSeq才被激活为可进行设置，否则设置无效。

```c++
consumerHelper.SetAttribute("MaxSeq", IntegerValue(std::numeric_limits<uint32_t>::max());
```

### 使用

```c++
ndn::AppHelper consumerHelper("ns3::ndn::ConsumerWindow");
consumerHelper.SetAttribute(...);
consumerHelper.Install(consumerNodes); // 在指定节点上安装用户
```

## ConsumerBatches

ConsumerBatches类是一个基于滑动窗口的用户类，可发出变化速率的兴趣包。

### 设置参数

- Batches：参数值为向量，包括时间与数量的组合，用以指定用户在何时发出多少兴趣包。

```c++
// 用户在1s时发出1个兴趣包，在2s时发出5个兴趣包
consumerHelper.SetAttribute("Window", StringValue("1s 1 2s 5"));
```

### 使用

```c++
ndn::AppHelper consumerHelper("ns3::ndn::ConsumerWindow");
consumerHelper.SetAttribute(...);
consumerHelper.Install(consumerNodes); // 在指定节点上安装用户
```

## 发出兴趣包`SendPacket()`

用户类中最重要的方法是`SendPacket()`，用以创建并按照一定规则发出兴趣包。我们以基类Consumer类为例，详细介绍`SendPacket()`方法。

```c++
void
Consumer::SendPacket()
{
  if (!m_active)
    return;

  NS_LOG_FUNCTION_NOARGS();
  
  /********************（1）创建兴趣包的序号********************/    
  uint32_t seq = std::numeric_limits<uint32_t>::max(); // 定义兴趣包序号

  while (m_retxSeqs.size()) {
    seq = *m_retxSeqs.begin(); // 设置当前兴趣包序号为序号数组的第一位
    m_retxSeqs.erase(m_retxSeqs.begin()); // 删除序号数组的第一位
    break;
  }

  if (seq == std::numeric_limits<uint32_t>::max()) {
    if (m_seqMax != std::numeric_limits<uint32_t>::max()) {
      if (m_seq >= m_seqMax) {
        return; // 发完所有兴趣包
      }
    }
		
    seq = m_seq++;
  }

  // 将当前兴趣包序号附加到兴趣包名称前缀中，构成总的兴趣包名称
  shared_ptr<Name> nameWithSequence = make_shared<Name>(m_interestName);
  nameWithSequence->appendSequenceNumber(seq);
  /********************（1）END*******************/    
  
  
  /********************（2）创建兴趣包并设置参数********************/    
  // 创建兴趣包
  shared_ptr<Interest> interest = make_shared<Interest>();
  
  // 设置兴趣包Nonce值，用以避免兴趣包在网络各节点之间循环转发
  interest->setNonce(m_rand->GetValue(0, std::numeric_limits<uint32_t>::max()));
  
  // 设置兴趣包的名称
  interest->setName(*nameWithSequence);
  
  // 设置兴趣包的生存时间
  time::milliseconds interestLifeTime(m_interestLifeTime.GetMilliSeconds());
  interest->setInterestLifetime(interestLifeTime);
  
	// 此处可设置自定义的参数或标签，详见“ndnSIM仿真平台使用之在兴趣包与数据包内添加标签（字段）”
  interest->setHops(0);
  interest->setFaces("");

  // NS_LOG_INFO ("Requesting Interest: \n" << *interest);
  NS_LOG_INFO("> Interest for " << seq);
  /********************（2）END*******************/    

  
  /********************（3）发出兴趣包********************/    
  WillSendOutInterest(seq);

  m_transmittedInterests(interest, this, m_face);
  m_face->onReceiveInterest(*interest);
	
  // 按一定规则发出下个兴趣包，如指数分布、均匀分布、Zipf-Mandelbrot分布等
  ScheduleNextPacket();
  /********************（3）END*******************/
}
```

## 创建与使用自定义的用户类

在大部分的仿真实验中并不需要对ndnSIM提供的用户类做过多修改，只需要在参数设置部分为自定义的兴趣包标签设置初始值。若直接修改原有用户类的代码，会影响原有用户类的使用（尽管大部分情况下这种方式非常方便），因此我们推荐创建一个自定义的用户类用于我们自己的仿真实验。

为此，我们创建一个ConsumerZipfMandelbrotTest类，该类与原ConsumerZipfMandelbrot类几乎一致，仅在加入一个自定义的整型标签TestTag（如何添加标签见[ndnSIM仿真平台使用之在兴趣包与数据包内添加标签（字段）](https://kanyuanzhi.github.io/2020/03/19/ndnSIM仿真平台使用之在兴趣包与数据包内添加标签-字段.html)）。此外，考虑到如果每次实验中标签TagTest有不同的初始化值，那么每次都在`SendPacket()`重新赋值该标签将大大影响实验的效率（因为ndnSIM需要重新编译该类），因此我们将在示例文件中显式地设置标签的初始化值。

### 创建类所在文件

复制ndn-consumer-zipf-mandelbrot.hpp与ndn-consumer-zipf-mandelbrot.cpp，并分别命名为ndn-consumer-zipf-mandelbrot-test.hpp与ndn-consumer-zipf-mandelbrot-test.cpp。

### 修改类名

- 将代码中所有的`ConsumerZipfMandelbrot`替换为`ConsumerZipfMandelbrotTest`；

```c++
NS_LOG_COMPONENT_DEFINE("ndn.ConsumerZipfMandelbrotTest");

namespace ns3 {
namespace ndn {

NS_OBJECT_ENSURE_REGISTERED(ConsumerZipfMandelbrotTest);

TypeId
ConsumerZipfMandelbrotTest::GetTypeId(void){
  ...
}
...
```

- 将ndn-consumer-zipf-mandelbrot-test.cpp中的include文件ndn-consumer-zipf-mandelbrot.hpp修改为ndn-consumer-zipf-mandelbrot-test.hpp：

```c++
#include "ndn-consumer-zipf-mandelbrot-test.hpp"
```

### 添加自定义标签设置接口

- 在ndn-consumer-zipf-mandelbrot-test.hpp中的ConsumerZipfMandelbrot类定义里添加私有属性m_test_tag：

```c++
class ConsumerZipfMandelbrot : public ConsumerCbr {
...
private:
  uint32_t m_N;               // number of the contents
  double m_q;                 // q in (k+q)^s
  double m_s;                 // s in (k+q)^s
  std::vector<double> m_Pcum; // cumulative probability
	
  int32_t m_test_tag;				// 自定义标签
  
  Ptr<UniformRandomVariable> m_seqRng; // RNG
};
...
```

- 在ndn-consumer-zipf-mandelbrot-test.cpp中的`GetTypeId()`方法里添加设置接口：

```c++
ConsumerZipfMandelbrot::GetTypeId(void){
  static TypeId tid =
    TypeId("ns3::ndn::ConsumerZipfMandelbrotTest")
      ...
      .AddAttribute("TestTag", "it is a test tag", IntegerValue(1),
                    MakeIntegerAccessor(&ConsumerZipfMandelbrotTest::m_test_tag),
                    MakeIntegerChecker<int32_t>());	// 默认为1

  return tid;
}
```

- 在`SendPacket()`方法中对标签初始化：

```c++
void
ConsumerZipfMandelbrot::SendPacket()
{
  ...
  shared_ptr<Interest> interest = make_shared<Interest>();
  interest->setNonce(m_rand->GetValue(0, std::numeric_limits<uint32_t>::max()));
  interest->setName(*nameWithSequence);
  
	// 初始TestTag，注意此步要求已经为兴趣包添加了TestTag标签
  interest->setTestTag(m_test_tag);
  ...
}
```

### 在示例文件中使用自定义的用户类

每次运行仿真实验时，我们仅需要在示例文件中修改TestTag的值即可，从而避免编译整个ConsumerZipfMandelbrotTest类。

```c++
...
ndn::AppHelper consumerHelper("ns3::ndn::ConsumerZipfMandelbrotTest");
consumerHelper.SetAttribute("TestTag", IntegerValue(1)); 
...
```


















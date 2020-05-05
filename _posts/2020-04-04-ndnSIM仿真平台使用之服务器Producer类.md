---
title: ndnSIM仿真平台使用之服务器Producer类
tags:	ndnSIM
key: 20200404
---

ndnSIM仅提供了一个服务器Producer类，本文首先介绍Producer类的参数设置与使用方式，然后介绍roducer类的核心方法`OnInterest()`以及如何初始化自定义的数据包标签，最后介绍如何创建与使用自定义的服务器类。

<!--more-->

版本信息如下：

操作系统：Ubuntu 16.04 <br>ndnSIM：ndnSIM-2.1<br>
ns-3-dev：ns-3.23-dev-ndnSIM-2.1<br>

## Producer

Producer类发出数据包，用以响应用户发出的兴趣包。

> 路径：/ndnSIM/apps

### 设置参数

- Prefix：该Producer响应请求何种前缀的兴趣包。

```c++
consumerHelper.SetPrefix("/prefix")
// 或 consumerHelper.SetAttribute("Prefix", StringValue("/prefix"));
```

- Postfix：添加到发出的数据包中，用以区别是哪个Producer做出的响应（producer-uniqueness）。

```c++
consumerHelper.SetAttribute("Postfix", StringValue("/"));
```

- PayloadSize：数据包的大小，单位KB。

```c++
consumerHelper.SetAttribute("PayloadSize", UintegerValue(1024));
```

- Freshness：数据包在缓存中的过期时间，此参数需要与`ns3::ndn::cs::Freshness::Lru`配合使用，默认值为0，表示不启用该参数。

```c++
ndnHelper.SetOldContentStore("ns3::ndn::cs::Freshness::Lru", "MaxSize", "100"); 
···
producerHelper.SetAttribute("Freshness", TimeValue(Seconds(2.0))); 
```

- Signature：对数据包签名，提供安全性保证，一般实验中用不到。

```c++
producerHelper.SetAttribute("Signature", UintegerValue(0)); 
```

### 使用

```c++
ndn::AppHelper producerHelper("ns3::ndn::Producer");
producerHelper.SetPrefix("/prefix"); // 该服务器将响应前缀为"/prefix"的兴趣包
producerHelper.SetAttribute(...);
producerHelper.Install(node); // 在指定节点上安装服务器
```

## 兴趣包触发OnInterest()

Producer类中最重要的方法是`OnInterest()`，是服务器收到兴趣包后触发一些操作，主要包括创建并按照一定规则发出数据包。我们在此详细介绍`OnInterest`方法。

```c++
void
Producer::OnInterest(shared_ptr<const Interest> interest)
{
  App::OnInterest(interest); // tracing inside

  NS_LOG_FUNCTION(this << interest);

  if (!m_active)
    return;
  
  /********************（1）创建数据包并设置参数********************/ 
  // ndnSIM中数据名称的详细设置说明见http://named-data.net/doc/tech-memos/naming-conventions.pdf
  Name dataName(interest->getName());
  // dataName.append(m_postfix);
  // dataName.appendVersion(); // 用于设置数据包的版本号

  auto data = make_shared<Data>();
  data->setName(dataName);
  
  // 设置过期时间
  data->setFreshnessPeriod(::ndn::time::milliseconds(m_freshness.GetMilliSeconds()));

  // 设置一定大小的数据段
  data->setContent(make_shared< ::ndn::Buffer>(m_virtualPayloadSize));

  // 设置签名
  Signature signature;
  SignatureInfo signatureInfo(static_cast< ::ndn::tlv::SignatureTypeValue>(255));

  if (m_keyLocator.size() > 0) {
    signatureInfo.setKeyLocator(m_keyLocator);
  }

  signature.setInfo(signatureInfo);
  signature.setValue(::ndn::nonNegativeIntegerBlock(::ndn::tlv::SignatureValue, m_signature));

  data->setSignature(signature);
  
  // 此处可设置自定义的参数或标签，详见“ndnSIM仿真平台使用之在兴趣包与数据包内添加标签（字段）”
  data->setTestTag(1);
  
  NS_LOG_INFO("node(" << GetNode()->GetId() << ") responding with Data: " << data->getName());

  // to create real wire encoding
  data->wireEncode();
  /********************（1）END*******************/ 
  
  
  /********************（2）发出数据包********************/ 
  // 跟踪数据包
  m_transmittedDatas(data, this, m_face);
  // 在m_face上发数据包
  m_face->onReceiveData(*data);
  /********************（2）END*******************/ 
}
```

## 创建与使用自定义的服务器类

与用户类类似，在大部分的仿真实验中并不需要对ndnSIM提供的服务器类做过多修改，只需要在参数设置部分为自定义的数据包标签设置初始值。在此，我们创建一个自定义的服务器类ProducerTest用于我们自己的仿真实验。该类与原Producer类几乎一致，仅在加入一个自定义的整型标签TestTag（如何添加标签见[ndnSIM仿真平台使用之在兴趣包与数据包内添加标签（字段）](https://kanyuanzhi.github.io/2020/03/19/ndnSIM仿真平台使用之在兴趣包与数据包内添加标签-字段.html)）。同样，我们将在示例文件中显式地设置标签的初始化值以避免没进行一次实验都要修改ProducerTest类。

### 创建类所在文件

复制ndn-producer.hpp与ndn-producer.cpp，分别命名为ndn-producer-test.hpp与ndn-producer-test.cpp

### 修改类名

- 将两个文件的代码中所有的`Producer`替换为`ProducerTest`：

```c++
NS_LOG_COMPONENT_DEFINE("ndn.ProducerTest");

namespace ns3 {
namespace ndn {

NS_OBJECT_ENSURE_REGISTERED(ProducerTest);

TypeId
Producer::GetTypeId(void){
  ...
}
...
```

- 将ndn-producer-test.cpp中的include文件ndn-producer.hpp修改为ndn-producer-test.hpp：

```c++
#include "ndn-producer-test.hpp"
```

### 添加自定义标签设置接口

- 在ndn-producer-test.hpp中的ProducerTest类定义里添加私有属性m_test_tag：

```c++
class ProducerTest : public App {
...
private:
  Name m_prefix;
  Name m_postfix;
  uint32_t m_virtualPayloadSize;
  Time m_freshness;
  
  int32_t m_test_tag;				// 自定义标签

  uint32_t m_signature;
  Name m_keyLocator;
};
...
```

- 在ndn-producer-test.cpp中的`GetTypeId()`方法里添加属性设置接口：

```c++
ProducerTest::GetTypeId(void){
  static TypeId tid =
    TypeId("ns3::ndn::ProducerTest")
      ...
      .AddAttribute("TestTag", "it is a test tag", IntegerValue(1),
                    MakeIntegerAccessor(&ProducerTest::m_test_tag),
                    MakeIntegerChecker<int32_t>());	// 默认为1

  return tid;
}
```

- 在`OnInterest`方法中对标签初始化：

```c++
void
Producer::OnInterest(shared_ptr<const Interest> interest)
{
  ...
  data->setSignature(signature);
  
  // 初始TestTag，注意此步要求已经为数据包添加了TestTag标签
  data->setTestTag(m_test_tag);
  ...
}
```

### 在示例文件中使用自定义的服务器类

每次运行仿真实验时，我们仅需要在示例文件中修改TestTag的值即可，从而避免编译整个ProducerTest类。

```c++
...
ndn::AppHelper producerHelper("ns3::ndn::ProducerTest");
producerHelper.SetAttribute("TestTag", IntegerValue(1));
...
```


















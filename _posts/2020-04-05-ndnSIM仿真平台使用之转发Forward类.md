---
title: ndnSIM仿真平台使用之转发Forward类
tags:	ndnSIM
key: 20200405
---

Forward类包含一系列事件触发器，是ndnSIM转发过程的核心。通过理解Forward类，也有助于我们理解NDN的包处理机制。本文将详细介绍仿真实验中最常用到的几种事件触发器源码，基于此将很容易实现自定义的转发处理策略。需要注意的是，Forward类不同于具体的转发策略（如best-route、multicast等），该类仅是NDN转发引擎的实现，换句话说，兴趣包经过Forward类处理后再交由具体的转发策略转发。

<!--more-->

⭐️**此部分强烈推荐阅读[NFD开发者手册](http://named-data.net/publications/techreports/ndn-0021-4-nfd-developer-guide/)以获取更详细的说明。**

版本信息如下：

操作系统：Ubuntu 16.04 <br>ndnSIM：ndnSIM-2.1<br>
ns-3-dev：ns-3.23-dev-ndnSIM-2.1<br>

## Forward类概要

> 路径：/ndnSIM/NFD/daemon/fw

Forward类是一系列转发管道（forwarding pipelines），包含从接受兴趣包/数据包到发出兴趣包/数据包之间的所有操作步骤。

### 主要私有属性

forward.hpp中定义了Forward类的若干私有属性，其中最重要的是：

- FIB表：m_fib；
- PIT表：m_pit；
- 新CS表：m_cs；
- 旧CS表：m_csFromNdnSim，若在示例文件中使用`ndnHelper.SetOldContentStore()`设置CS表，则在转发过程中使用旧CS表。

```c++
class Forwarder{
...
private:
  ForwarderCounters m_counters;
  FaceTable m_faceTable;
  // tables
  NameTree       m_nameTree; 
  Fib            m_fib; // FIB表
  Pit            m_pit; // PIT表
  Cs             m_cs; // 新CS表
  Measurements   m_measurements;
  StrategyChoice m_strategyChoice;
  DeadNonceList  m_deadNonceList;
  shared_ptr<NullFace> m_csFace;
  ns3::Ptr<ns3::ndn::ContentStore> m_csFromNdnSim; // 旧CS表
  ...
};
```

### 主要事件触发器

Forward类由一系列时间触发器组成，在此我们仅介绍最常用到的几种（最常修改）：

- onIncomingInterest()：路由节点收到兴趣包时触发的操作；
- onContentStoreMiss()：兴趣包没有命中缓存时触发的操作；
- onContentStoreHit()：兴趣包命中缓存时触发的操作；
- onOutgoingInterest()：路由节点转发出兴趣包时触发的操作；
- onIncomingData()：路由节点收到数据包时触发的操作；
- onOutgoingData()：路由节点转发出兴趣包时触发的操作。

## 主要事件触发器源码详解

此部分主要为逻辑代码介绍，技术性的知识需要了解C++中的转换运算符const_cast和智能指针shared_ptr等，网上相关内容有很多，可自行查阅。

### onIncomingInterest()

```c++
void
Forwarder::onIncomingInterest(Face& inFace, const Interest& interest)
{
  // receive Interest
  NFD_LOG_DEBUG("onIncomingInterest face=" << inFace.getId() <<
                " interest=" << interest.getName());
  
  /****************************************/ 
  // 此处为自定义的兴趣包标签（跳数与入端口列表）处理流程
  // 由于interest是一个常量指针，不能改变其原有属性，所以需要先转为非常量指针再做处理
  const_cast<Interest &>( interest ).setHops( interest.getHops() + 1 ); // 设置兴趣包跳数

  std::string faces = interest.getFaces(); // 获取兴趣包的入端口列表
  // inFace是兴趣包进入当前节点的端口
  faces += std::to_string( inFace.getId() ) + " ";
  const_cast<Interest &>( interest ).setFaces( faces ); // 添加兴趣包的入端口号至入端口列表
  /****************************************/ 

  const_cast<Interest&>(interest).setIncomingFaceId(inFace.getId()); // 设置
  ++m_counters.getNInInterests();

  // /localhost scope control
  // 一种安全机制，来自非本地Face的Interest不允许有一个以/localhost前缀开头的名字，
  // 因为它是为本地主机通信保留的
  bool isViolatingLocalhost = !inFace.isLocal() &&
                              LOCALHOST_NAME.isPrefixOf(interest.getName());
  if (isViolatingLocalhost) {
    NFD_LOG_DEBUG("onIncomingInterest face=" << inFace.getId() <<
                  " interest=" << interest.getName() << " violates /localhost");
    // (drop)
    return;
  }

  // 在同名PIT记录列表中新增该兴趣包的记录，并返回同名PIT记录pitEntry，
  // 此处详见/ndnSIM/NFD/daemon/table/pit.cpp中的insert()方法，
  // 该方法返回一个二元组{ entry, bool }，前者表示同名记录，后者表示是否查询到同名记录
  shared_ptr<pit::Entry> pitEntry = m_pit.insert(interest).first;

  // 检测重复的Nonce，避免循环转发
  int dnw = pitEntry->findNonce(interest.getNonce(), inFace);
  bool hasDuplicateNonce = (dnw != pit::DUPLICATE_NONCE_NONE) ||
                           m_deadNonceList.has(interest.getName(), interest.getNonce());
  if (hasDuplicateNonce) {
    // goto Interest loop pipeline
    this->onInterestLoop(inFace, interest, pitEntry);
    return;
  }

  // cancel unsatisfy & straggler timer
  this->cancelUnsatisfyAndStragglerTimer(pitEntry);

  // 此处ndnSIM的处理流程是先查询PIT中有无记录，如果有记录，那么说明同名兴趣包未被满足，
  // 也就不需要再查询CS了。而NDN原始的处理流程是先直接查询CS中有无所请求副本
  
  // 获取同名pitEntry中的入端口号列表
  const pit::InRecordCollection& inRecords = pitEntry->getInRecords(); 
  
  // 如果inRecords.begin() != inRecords.end()，则说明入端口号列表中有至少两个记录，
  // 即该兴趣包到达之前已经有同名兴趣包达到且未被满足，此时需要聚合该兴趣包
  bool isPending = inRecords.begin() != inRecords.end();
  
  
  if (!isPending) {
    // isPending==False，说明该兴趣包到达之前没有同名兴趣包到达，此时需要查询CS中有无所请求副本
    if (m_csFromNdnSim == nullptr) { // 启用新CS
      m_cs.find(interest,
                bind(&Forwarder::onContentStoreHit, this, ref(inFace), pitEntry, _1, _2),
                bind(&Forwarder::onContentStoreMiss, this, ref(inFace), pitEntry, _1));
    }
    else { // 启用旧CS
      // 查询CS，返回空指针或内容副本。注意此处传入参数是整个兴趣包，而不是兴趣包的名称，
      // 具体的Lookup()方法在各缓存策略中实现，详见/ndnSIM/model/cs
      shared_ptr<Data> match = m_csFromNdnSim->Lookup(interest.shared_from_this());
      if (match != nullptr) {
        // 查询到CS中有所请求副本，触发命中触发器
        this->onContentStoreHit(inFace, pitEntry, interest, *match);
      }
      else {
        // 没有查询到CS中有所请求副本，触发未命中触发器
        this->onContentStoreMiss(inFace, pitEntry, interest);
      }
    }
  }
  else {
    // isPending==True，说明该兴趣包到达之前有同名兴趣包到达，
    // 此时CS中肯定没有所请求副本，触发未命中触发器
    this->onContentStoreMiss(inFace, pitEntry, interest);
  }
}
```

### onContentStoreMiss()

```c++
void
Forwarder::onContentStoreMiss(const Face& inFace,
                              shared_ptr<pit::Entry> pitEntry,
                              const Interest& interest)
{
  NFD_LOG_DEBUG("onContentStoreMiss interest=" << interest.getName());

  shared_ptr<Face> face = const_pointer_cast<Face>(inFace.shared_from_this());
  // 将该兴趣包的入端口号插入到同名PIT记录中
  pitEntry->insertOrUpdateInRecord(face, interest);

  // 重置pitEntry的未满足时间，涉及到过期丢弃
  this->setUnsatisfyTimer(pitEntry);

  // 在FIB表中查询该同名PIT记录的转出端口
  // 注意此处查询FIB表时的传入参数是整个同名PIT记录，
  // 而不是同名PIT记录的名称（即该记录中兴趣包请求内容的名称）
  shared_ptr<fib::Entry> fibEntry = m_fib.findLongestPrefixMatch(*pitEntry);
  
  // 需要注意的是，此处并未直接交给onOutgoingInterest()处理，
  // 而是先交给相应转发策略的afterReceiveInterest()方法处理，
  // 再交给onOutgoingInterest()处理。
  // &Strategy::afterReceiveInterest()在基类/ndnSIM/NFD/daemon/fw/strategy.hpp中定义，
  // 由各转发策略派生类实现
  this->dispatchToStrategy(pitEntry, bind(&Strategy::afterReceiveInterest, _1,
                                          cref(inFace), cref(interest), 
                                          fibEntry, pitEntry));
}
```

### onContentStoreHit()

```c++
void
Forwarder::onContentStoreHit(const Face& inFace,
                             shared_ptr<pit::Entry> pitEntry,
                             const Interest& interest,
                             const Data& data)
{
  NFD_LOG_DEBUG("onContentStoreHit interest=" << interest.getName());
  
  // 兴趣包被满足后的一些发出信号类操作，没有实际行为
  beforeSatisfyInterest(*pitEntry, *m_csFace, data);
  
  // 交由相应的转发策略处理，
  // &Strategy::beforeSatisfyInterest()仅在基类/ndnSIM/NFD/daemon/fw/strategy.hpp中定义，
  // 各转发策略派生类并未实现，因为命中缓存不需要转发兴趣包
  this->dispatchToStrategy(pitEntry, bind(&Strategy::beforeSatisfyInterest, _1,
                                          pitEntry, cref(*m_csFace), cref(data)));
  // 设置数据包的入端口地址
  const_pointer_cast<Data>
    (data.shared_from_this())->setIncomingFaceId(FACEID_CONTENT_STORE);

  // 设置PIT记录驻留时间
  this->setStragglerTimer(pitEntry, true, data.getFreshnessPeriod());

  // 交由onOutgoingData()触发器处理
  this->onOutgoingData(data, *const_pointer_cast<Face>(inFace.shared_from_this()));
}
```

### onOutgoingInterest()

```c++
void
Forwarder::onOutgoingInterest(shared_ptr<pit::Entry> pitEntry, Face& outFace,
                              bool wantNewNonce)
{
  if (outFace.getId() == INVALID_FACEID) {
    NFD_LOG_WARN("onOutgoingInterest face=invalid interest=" << pitEntry->getName());
    return;
  }
  NFD_LOG_DEBUG("onOutgoingInterest face=" << outFace.getId() <<
                " interest=" << pitEntry->getName());

  // 范围控制，包含/localhost和/localhop的兴趣包将被特殊处理，具体参见NFD开发者手册
  if (pitEntry->violatesScope(outFace)) {
    NFD_LOG_DEBUG("onOutgoingInterest face=" << outFace.getId() <<
                  " interest=" << pitEntry->getName() << " violates scope");
    return;
  }

  // 通过pitEntry选择兴趣包，原兴趣包已经作为一个属性记录在inRecords中，
  // 所以onOutgoingInterest()触发器的传入参数中并不含有interest
  const pit::InRecordCollection& inRecords = pitEntry->getInRecords();
  pit::InRecordCollection::const_iterator pickedInRecord = std::max_element(
    inRecords.begin(), inRecords.end(), bind(&compare_pickInterest, _1, _2, &outFace));
  BOOST_ASSERT(pickedInRecord != inRecords.end());
  shared_ptr<Interest> interest = const_pointer_cast<Interest>(
    pickedInRecord->getInterest().shared_from_this());

  // 是否需要重置Nonce值
  if (wantNewNonce) {
    interest = make_shared<Interest>(*interest);
    static boost::random::uniform_int_distribution<uint32_t> dist;
    interest->setNonce(dist(getGlobalRng()));
  }

  // 记录兴趣包的转出端口
  pitEntry->insertOrUpdateOutRecord(outFace.shared_from_this(), *interest);

  // 在outFace端口上发出，outFace在相应的转发策略中获得
  outFace.sendInterest(*interest);
  ++m_counters.getNOutInterests();
}
```

此处要强调的是，在ndnSIM中，兴趣包未命中缓存时，兴趣包并不直接交由`onOutgoingInterest()`处理，而是先交由各转发策略的`afterReceiveInterest()`方法处理，再交由`onOutgoingInterest()`处理，所以在`onContentStoreMiss()`中并未出现`onOutgoingInterest()`，**中间一步交由转发策略策略处理的原因是为了得到兴趣包的转出端口`outFace`**。以best-route为例，`afterReceiveInterest()`方法在头文件/ndnSIM/NFD/daemon/fw/best-route-strategy.hpp中的定义如下：

```c++
class BestRouteStrategy : public Strategy
{
public:
  ...
  virtual void
  afterReceiveInterest(const Face& inFace,
                       const Interest& interest,
                       shared_ptr<fib::Entry> fibEntry,
                       shared_ptr<pit::Entry> pitEntry) DECL_OVERRIDE;
  ...
};
```

在文件/ndnSIM/NFD/daemon/fw/best-route-strategy.cpp中的实现如下：

```c++
void
BestRouteStrategy::afterReceiveInterest(const Face& inFace,
                   const Interest& interest,
                   shared_ptr<fib::Entry> fibEntry,
                   shared_ptr<pit::Entry> pitEntry)
{
  ...
  this->sendInterest(pitEntry, outFace);
}
```

其中`sendInterest()`方法在头文件/ndnSIM/NFD/daemon/fw/strategy.hpp中的实现如下：

```c++
class Strategy : public enable_shared_from_this<Strategy>, noncopyable
{
  ...
  inline void
  Strategy::sendInterest(shared_ptr<pit::Entry> pitEntry,
                       shared_ptr<Face> outFace,
                       bool wantNewNonce)
  {
    // 调用Forward类中的onOutgoingInterest()触发器
    m_forwarder.onOutgoingInterest(pitEntry, *outFace, wantNewNonce);
  }
}
```

以上可以清晰看出兴趣包的处理流程。

### onIncomingData()

```c++
void
Forwarder::onIncomingData(Face& inFace, const Data& data)
{
  // receive Data
  NFD_LOG_DEBUG("onIncomingData face=" << inFace.getId() << " data=" << data.getName());
  const_cast<Data&>(data).setIncomingFaceId(inFace.getId()); // 设置数据包的入端口号
  ++m_counters.getNInDatas();

  // /localhost scope control
  // 一种安全机制，原理与onIncomingInterest()中的/localhost scope control部分类似
  bool isViolatingLocalhost = !inFace.isLocal() &&
                              LOCALHOST_NAME.isPrefixOf(data.getName());
  if (isViolatingLocalhost) {
    NFD_LOG_DEBUG("onIncomingData face=" << inFace.getId() <<
                  " data=" << data.getName() << " violates /localhost");
    // (drop)
    return;
  }

  // 在PIT中查找请求该数据包的PIT记录
  pit::DataMatchResult pitMatches = m_pit.findAllDataMatches(data);
  if (pitMatches.begin() == pitMatches.end()) {
    // 没有查到，该数据包没有对应的兴趣包，交由onDataUnsolicited()处理
    this->onDataUnsolicited(inFace, data);
    return;
  }

  // Remove Ptr<Packet> from the Data before inserting into cache, serving two purposes
  // - reduce amount of memory used by cached entries
  // - remove all tags that (e.g., hop count tag) that could have been associated with Ptr<Packet>
  //
  // Copying of Data is relatively cheap operation, as it copies (mostly) a collection of Blocks
  // pointing to the same underlying memory buffer.
  // 以上是源码中对于在数据包缓存副本之前移除其中数据与标签的说明，
  // 主要是为了节省空间以及避免一些与包本身有关的标签对副本的属性产生影响。
  // ndnSIM中数据包携带的数据字段仅是一个有一定大小的buffer，并不具有实际意义，
  shared_ptr<Data> dataCopyWithoutPacket = make_shared<Data>(data);
  dataCopyWithoutPacket->removeTag<ns3::ndn::Ns3PacketTag>();

  // 在CS缓存副本
  if (m_csFromNdnSim == nullptr)
    m_cs.insert(*dataCopyWithoutPacket); // 使用新CS
  else
    m_csFromNdnSim->Add(dataCopyWithoutPacket); // 使用旧CS

  std::set<shared_ptr<Face> > pendingDownstreams; // 定义向下转发的聚合端口列表，从pitEntry中获取
  // foreach PitEntry
  for (const shared_ptr<pit::Entry>& pitEntry : pitMatches) {
    NFD_LOG_DEBUG("onIncomingData matching=" << pitEntry->getName());

    // 取消pitEntry的unsatisfy和straggler计时器，因为已经被响应
    this->cancelUnsatisfyAndStragglerTimer(pitEntry);

    // 获得向下转发的聚合端口列表，
    // pitEntry中的入端口即是数据包的出端口
    const pit::InRecordCollection& inRecords = pitEntry->getInRecords();
    for (pit::InRecordCollection::const_iterator it = inRecords.begin();
                                                 it != inRecords.end(); ++it) {
      if (it->getExpiry() > time::steady_clock::now()) {
        // 未过期的PIT记录才会被响应
        pendingDownstreams.insert(it->getFace());
      }
    }

    // 兴趣包被满足后的一些发出信号类操作，没有实际行为
    beforeSatisfyInterest(*pitEntry, inFace, data);
    
    // 此处与onContentStoreHit()中的this->dispatchToStrategy()一样，交由相应的转发策略处理，
    // &Strategy::beforeSatisfyInterest()仅在基类/ndnSIM/NFD/daemon/fw/strategy.hpp中定义，
    // 用于发出兴趣包被满足的信号，各转发策略派生类中并未具体实现该方法
    this->dispatchToStrategy(pitEntry, bind(&Strategy::beforeSatisfyInterest, _1,
                                            pitEntry, cref(inFace), cref(data)));

    // Dead Nonce List insert if necessary (for OutRecord of inFace)
    this->insertDeadNonceList(*pitEntry, true, data.getFreshnessPeriod(), &inFace);

    // PIT表中相应的PIT记录被满足，需要删除
    pitEntry->deleteInRecords();
    pitEntry->deleteOutRecord(inFace);

    // 设置PIT记录驻留时间
    this->setStragglerTimer(pitEntry, true, data.getFreshnessPeriod());
  }

  // 在每个向下转发的聚合端口上发出数据包
  for (std::set<shared_ptr<Face> >::iterator it = pendingDownstreams.begin();
      it != pendingDownstreams.end(); ++it) {
    shared_ptr<Face> pendingDownstream = *it;
    if (pendingDownstream.get() == &inFace) {
      continue;
    }
    // 交由onOutgoingData()触发器处理
    this->onOutgoingData(data, *pendingDownstream);
  }
}
```

### onOutgoingData()

```c++
void
Forwarder::onOutgoingData(const Data& data, Face& outFace)
{
  if (outFace.getId() == INVALID_FACEID) {
    NFD_LOG_WARN("onOutgoingData face=invalid data=" << data.getName());
    return;
  }
  NFD_LOG_DEBUG("onOutgoingData face=" << outFace.getId() << " data=" << data.getName());

  // 一种安全机制，原理与onIncomingInterest()中的/localhost scope control部分类似
  bool isViolatingLocalhost = !outFace.isLocal() &&
                              LOCALHOST_NAME.isPrefixOf(data.getName());
  if (isViolatingLocalhost) {
    NFD_LOG_DEBUG("onOutgoingData face=" << outFace.getId() <<
                  " data=" << data.getName() << " violates /localhost");
    // (drop)
    return;
  }

  // TODO traffic manager
  // 彩蛋？2.1版本的ndnSIM准备在此进行流量管理

  // 在outFace端口上发出数据包，outFace从PIT记录中获取
  outFace.sendData(data);
  ++m_counters.getNOutDatas();
}
```
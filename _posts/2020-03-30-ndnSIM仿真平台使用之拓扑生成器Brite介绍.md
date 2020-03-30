---
title: ndnSIM仿真平台使用之拓扑生成器Brite介绍
tags:	ndnSIM
key: 20200330
---

本文参考Brite官网提供的用户手册，介绍示拓扑生成器Brite的安装与使用，并提供一个简单的Python脚本将生成的拓扑文件转为ndnSIM可读取的文件。

<!--more-->



版本信息如下：

Brite：Brite2.1 <br>jdk：1.8.0_151<br>

## 安装

### 下载

Brite官网：<http://www.cs.bu.edu/brite>

注意到Brite有C++和Java两种版本，前者不支持GUI可视化界面，所以一般使用Java版本。

### 安装

下载完成后将压缩包移动到用户根目录下并解压，得到BRITE文件夹：

```
~$ sudo cp ~/下载/BRITE.tar.gz ~
~$ gunzip BRITE.tar.gz
~$ tar xvf BRITE.tar
```

进入BRITE目录，编译并运行：

```
~$ cd BRITE
~/BRITE$ make java
~/BRITE$ ./brite
```

这里需要系统有Java环境，关于如何在Ubuntu中安装Java网上有大量的参考文档。本文用的是jdk1.8.0，其他版本未测试，Oracle官网下载可能比较慢，可以去国内镜像网站中下载，比如华为的jdk镜像<https://repo.huaweicloud.com/java/jdk/>。

然后就可看到Brite的应用界面：

![image](https://github.com/kanyuanzhi/kanyuanzhi.github.io/raw/master/assets/myimages/20200330/1.jpg)

### 使用

Brite可生成的拓扑类型（Topology Type）有四种：

1. 1 Level: AS ONLY
2. 1 Level: ROUTER(IP) ONLY
3. 2 Level: TOP-DOWN 
4. 2 Level: BOTTOM-UP

我们只需要选择第二种ROUTER(IP) ONLY。在下方的参数设置里（Router Topology Parameters）也有很多可选参数，我们用到的是HS、LS和N，其中HS与LS表示生成拓扑平面的大小（与ndnSIM拓扑格式中节点的坐标有关），N表示节点总数，N<HS*LS。其他参数选择默认值即可，具体含义如下：

|   Parameter    |                 Meaning                 |              Values              |
| :------------: | :-------------------------------------: | :------------------------------: |
|       HS       |      Size of one side of the plane      |             int ≥ 1              |
|       LS       | Size of one side of a high-level square |             int ≥ 1              |
|       N        |             Number of nodes             |        int 1 ≤ N ≤ HS*HS         |
|     Model      |                model id                 |             int ≥ 1              |
|     alpha      |        Waxman-specific exponent         |        0 ＜ α ＜ 1; α ∈ R        |
|      beta      |        Waxman-specific exponent         |        0 ＜ β ＜ 1; β ∈ R        |
| Node Placement |    how nodes are placed in the plane    |         1: Random, 2: HT         |
|       m        |      Number of links per new node       |             int ≥ 1              |
|  Growth Type   |       how nodes join the topology       |    1: Incremental, 2: Random     |
|     BWdist     |      bandwidth assignment to links      | 1: Const, 2: Unif, 3: Exp, 4: HT |
|  MaxBW, MinBW  |     min, max link bandwidth values      |            float ＞ 0            |

比如我们设置HS=100、LS=1、N=100，输入Location为test，选择Formats为BTITE，然后点击Build Topology，在BRITE目录里会生成test.brite文件，即拓扑文件。

### 导出文件格式

Brite导出的文件格式如下：

节点格式：

|   Field   |                       Meaning                       |
| :-------: | :-------------------------------------------------: |
|  NodeId   |               Unique id for each node               |
|   xpos    |           x-axis coordinate in the plane            |
|   ypos    |           y-axis coordinate in the plane            |
| indegree  |                Indegree of the node                 |
| outdegree |                Outdegree of the node                |
|   ASid    | id of the AS this node belongs to (if hierarchical) |
|   type    |     Type assigned to the node (e.g. router, AS)     |

链路格式：

|   Field   |                       Meaning                       |
| :-------: | :-------------------------------------------------: |
|  EdgeId   |               Unique id for each edge               |
|   from    |                  node id of source                  |
|    to     |               node id of destination                |
|  length   |                  Euclidean length                   |
|   delay   |                  propagation delay                  |
| bandwidth |       bandwidth (assigned by AssignBW method)       |
|  ASfrom   |   if hierarchical topology, AS id of source node    |
|   ASto    | if hierarchical topology, AS id of destination node |
|   type    | Type assigned to the edge by classification routine |

而ndnSIM中拓扑文件的格式为：

```
router
# node  comment     yPos    xPos
Src1   NA        1       3
Src2   NA        3       3
Rtr1   NA        2       5
Rtr2   NA        2       7
Dst1   NA        1       9
Dst2   NA        3       9

link
# srcNode   dstNode     bandwidth   metric  delay   queue
Src1        Rtr1        10Mbps      1        10ms    20
Src2        Rtr1        10Mbps      1        10ms    20
Rtr1        Rtr2        1Mbps       1        10ms    20
Dst1        Rtr2        10Mbps      1        10ms    20
Dst2        Rtr2        10Mbps      1        10ms    20
```

因此我们需要进一步转换。有两种转换方式，一种是将test.brite文件的内容复制到excel表格中然后手动调整，另一种是编写脚本自动调整。此处需要注意的是：（1）Brite拓扑文件中节点的xy坐标与ndnSIM拓扑文件中节点的xy坐标是相反的；（2）一般情况下我们只需要用到Brite生成拓扑的结构，链路信息可以在转换过程中再次指定，比如ndnSIM拓扑文件中链路的bandwidth、metric、delay和queue的信息可以最后统一设置。此处我们提供一个简单的Python转换脚本，将此文件放入BRITE目录中然后运行，可以得到适用于ndnSIM的拓扑文件test.txt。

```python
import sys

if __name__ == "__main__":
    bandwidth = "10Mbps"
    metric = "1"
    delay = "10ms"
    queue = "20"

    f_in = open("test.brite","r")
    line = f_in.readline()

    routers = []
    links = []

    routers_start = False
    links_start = False

    while line:
        if line != "\n":
            if "Nodes:" in line:
                routers_start = True
                links_start = False
            elif "Edges:" in line:
                routers_start = False
                links_start = True
            elif routers_start and not links_start:
                items = line.split("\t")
                routers.append(items[0]+"\tNA\t"+items[2]+"\t"+items[1]+"\n")
            elif not routers_start and links_start:
                items = line.split("\t")
                links.append(items[1]+"\t"+items[2]+"\t"+bandwidth
                             +"\t"+metric+"\t"+delay+"\t"+queue+"\n")                
        line = f_in.readline()
    f_in.close()

    f_out = open("test.txt","w")
    f_out.write("router\n")
    f_out.writelines(routers)
    f_out.write("\nlink\n")
    f_out.writelines(links)
    f_out.close()
```

#### 最后，关于使用Brite生成更复杂的网络拓扑，可参照官网的用户手册<http://www.cs.bu.edu/brite/docs.html>。














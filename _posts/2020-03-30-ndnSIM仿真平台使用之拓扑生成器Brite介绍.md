---
title: ndnSIM仿真平台使用之示例文件结构介绍
tags:	ndnSIM
key: 20200330
---

本文介绍示拓扑生成器Brite的安装与使用，并提供一个简单的Python脚本将生成的拓扑文件转为ndnSIM可读取的文件。

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

### 简单使用



### 导出文件格式
















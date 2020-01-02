#### 数据存储单位

bit、Byte、 KB、MB、GB、TB、PB、EB、ZB、YB、 BB、NB、DB

1Byte = 8bit 

1K = 1024Byte 

1MB = 1024K 

1G = 1024M 

1T = 1024G 

1P = 1024T


#### 大数据解决的问题

海量数据的存储和海量数据的分析计算


#### 大数据的特点

Volume(大量)、Velocity(高速)、Variety(多样)、Value(低价值密度)

数据被分为结构化数据和非结构化数据。相对于以往便于存储的以数据库/文本为主的结构化数据，非结构化数据越来越多，包括网络日志、 音频、视频、图片、地理位置信息等。


#### Hadoop简介

1）Hadoop 是一个由 Apache 基金会所开发的分布式系统基础架构 

2）主要解决，海量数据的存储和海量数据的分析计算问题

3）广义上来说，HADOOP 通常是指一个更广泛的概念——HADOOP 生态圈


#### Hadoop 发展历史

1）Lucene--Doug Cutting 开创的开源软件，用 java 书写代码，实现与 Google 类似的全文搜索功能，它提供了全文检索引擎的架构，包括完整的查询引擎和索引引擎

2）2001 年年底成为 apache 基金会的一个子项目 

3）对于大数量的场景，Lucene 面对与 Google 同样的困难

4）学习和模仿 Google 解决这些问题的办法 :微型版 Nutch

5）可以说 Google 是 hadoop 的思想之源(Google 在大数据方面的三篇论文)

GFS --->HDFS 

Map-Reduce --->MR 

BigTable --->Hbase

6）2003-2004 年，Google 公开了部分 GFS 和 Mapreduce 思想的细节，以此为基础 Doug Cutting 等人用了 2 年业余时间实现了 DFS 和 Mapreduce 机制，使 Nutch 性能飙升

7）2005 年Hadoop 作为 Lucene的子项目 Nutch的一部分正式引入Apache基金会。2006 年 3 月份，Map-Reduce 和 Nutch Distributed File System (NDFS) 分别被纳入称为 Hadoop 的项 目中

8）名字来源于 Doug Cutting 儿子的玩具大象


#### Hadoop 三大发行版本

1）Hadoop 三大发行版本:Apache、Cloudera、Hortonworks

2）Apache 版本最原始(最基础)的版本，对于入门学习最好

3）Cloudera 在大型互联网企业中用的较多

Hortonworks 文档较好

1）Apache Hadoop 官网地址:http://hadoop.apache.org/releases.html

2）Cloudera Hadoop 官网地址:https://www.cloudera.com/downloads/cdh/5-10-0.html

3）Hortonworks Hadoop 官网地址:https://hortonworks.com/products/data-center/hdp/


#### Hadoop 的优势

1）高可靠性: 因为 Hadoop 假设计算元素和存储会出现故障，因为它维护多个工作数据副本，在出现故障时可以对失败的节点重新分布处理

2）高扩展性: 在集群间分配任务数据，可方便的扩展数以千计的节点

3）高效性: 在 MapReduce 的思想下，Hadoop 是并行工作的，以加快任务处理速度

4）高容错性: 自动保存多份副本数据，并且能够自动将失败的任务重新分配


#### Hadoop 组成

1）Hadoop HDFS: 一个高可靠、高吞吐量的分布式文件系统

2）Hadoop MapReduce: 一个分布式的离线并行计算框架

3）Hadoop YARN: 作业调度与集群资源管理的框架

4）Hadoop Common: 支持其他模块的工具模块(Configuration、RPC、序列化机制、日志 操作)


#### HDFS 的架构

1）NameNode(nn): 存储文件的元数据，如文件名，文件目录结构，文件属性(生成时间、副本数、文件权限)，以及每个文件的块列表和块所在的DataNode等

2）DataNode(dn): 在本地文件系统存储文件块数据，以及块数据的校验和

3）Secondary NameNode(2nn): 用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照


#### YARN 的架构

1）ResourceManager(rm): 处理客户端请求、启动/监控 ApplicationMaster、监控 NodeManager、 资源分配与调度

2）NodeManager(nm): 单个节点上的资源管理、处理来自 ResourceManager 的命令、处理来 自 ApplicationMaster 的命令

3）ApplicationMaster: 数据切分、为应用程序申请资源，并分配给内部任务、任务监控与容错

4）Container: 对任务运行环境的抽象，封装了 CPU、内存等多维资源以及环境变量、启动 命令等任务运行相关的信息


#### MapReduce 的架构

MapReduce 将计算过程分为两个阶段: Map 和 Reduce 

1）Map 阶段并行处理输入数据

2）Reduce 阶段对 Map 结果进行汇总


#### Hadoop 运行模式

1）本地模式(默认模式): 不需要启用单独进程，直接可以运行，测试和开发时使用。 

2）伪分布式模式: 等同于完全分布式，只有一个节点。 

3）完全分布式模式: 多个节点一起运行。

#### 本地运行 Hadoop 案例

1、官方 grep 案例

1）创建在 hadoop-2.7.2 文件下面创建一个 input 文件夹

```
mkdir input
```

2）将 hadoop 的 xml 配置文件复制到 input

```
cp etc/hadoop/*.xml input
```

3）执行 share 目录下的 mapreduce 程序

```
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'
```

4）查看输出结果

```
cat output/*
```

2、官方 wordcount 案例

1）创建在 hadoop-2.7.2 文件下面创建一个 wcinput 文件夹

```
mkdir wcinput
```

2）在 wcinput 文件下创建一个 wc.input 文件

```
touch wc.input

```

3）编辑 wc.input 文件，在文件中输入如下内容

```
hadoop yarn hadoop mapreduce test
```

4）回到 hadoop 目录/opt/module/hadoop-2.7.2 执行程序

```
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount wcinput wcoutput
```

5）查看结果

```
cat wcoutput/part-r-00000
```
















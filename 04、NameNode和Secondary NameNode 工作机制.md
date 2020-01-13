#### NameNode 和 Secondary NameNode 工作机制

1、第一阶段:namenode 启动

1）第一次启动 namenode 格式化后，创建 fsimage 和 edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存 

2）客户端对元数据进行增删改的请求

3）namenode 记录操作日志，更新滚动日志

4）namenode 在内存中对数据进行增删改查

2、第二阶段:Secondary NameNode 工作

1）Secondary NameNode 询问 namenode 是否需要 checkpoint，直接带回 namenode 是否检查结果

2）Secondary NameNode 请求执行 checkpoint

3）namenode 滚动正在写的 edits 日志

4）将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode

5）Secondary NameNode 加载编辑日志和镜像文件到内存，并合并

6）生成新的镜像文件 fsimage.chkpoint

7）拷贝 fsimage.chkpoint 到 namenode

8）namenode 将 fsimage.chkpoint 重新命名成 fsimage


#### 镜像文件和编辑日志文件

namenode 被格式化之后，将在/opt/module/hadoop-2.7.2/data/tmp/dfs/name/current 目录中 产生如下文件：

```
edits_0000000000000000000 
fsimage_0000000000000000000.md5 
seen_txid
VERSION

```

1）Fsimage 文件 :HDFS 文件系统元数据的一个永久性的检查点，其中包含 HDFS 文件系统的所有目录和文件 idnode 的序列化信息

2）Edits 文件 :存放 HDFS 文件系统的所有更新操作的路径，文件系统客户端执行 的所有写操作首先会被记录到 edits 文件中

3）seen_txid :文件保存的是一个数字，就是最后一个 edits_的数字

4）每次 Namenode 启动的时候都会将 fsimage 文件读入内存，并从 00001 开始

到 seen_txid 中记录的数字依次执行每个 edits 里面的更新操作，保证内存中的元数据信息是最新的、同步的，可以看成 Namenode 启动的时候就将 fsimage 和 edits 文件进行了合并。

1、oiv 查看 fsimage 文件

基本语法: 

hdfs oiv -p 文件类型 -i 镜像文件 -o 转换后文件输出路径

案例: 

```
hdfs oiv -p XML -i fsimage_0000000000000000025 -o /opt/module/hadoop-2.7.2/fsimage.xml

```

```
cat /opt/module/hadoop-2.7.2/fsimage.xml

```

2、oev 查看 edits 文件

基本语法:

hdfs oev -p 文件类型 -i 编辑日志 -o 转换后文件输出路径

案例：

```
hdfs oev -p XML -i edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop-2.7.2/edits.xml

```

```
cat /opt/module/hadoop-2.7.2/edits.xml

```

#### Namenode 版本号

1、查看 namenode 版本号
在/opt/module/hadoop-2.7.2/data/tmp/dfs/name/current 这个目录下查看 VERSION 

```
namespaceID=1933630176 
clusterID=CID-1f2bf8d1-5ad2-4202-af1c-6713ab381175
cTime=0
storageType=NAME_NODE 
blockpoolID=BP-97847618-192.168.10.102-1493726072779 
layoutVersion=-63

```

2、namenode 版本号具体解释

1）namespaceID 在 HDFS 上，会有多个 Namenode，所以不同 Namenode 的namespaceID 是不同的，分别管理一组 blockpoolID

2）clusterID 集群 id，全局唯一

3）cTime 属性标记了 namenode 存储系统的创建时间，对于刚刚格式化的存储系统， 这个属性为 0;但是在文件系统升级之后，该值会更新到新的时间戳

4）storageType 属性说明该存储目录包含的是 namenode 的数据结构

5）blockpoolID:一个 block pool id 标识一个 block pool，并且是跨集群的全局唯一，当一个新的 Namespace 被创建的时候(format 过程的一部分)会创建并持久化一个 唯一 ID。在创建过程构建全局唯一的 BlockPoolID 比人为的配置更可靠一些。NN 将 BlockPoolID 持久化到磁盘中，在后续的启动过程中，会再次 load 并使用

6）layoutVersion 是一个负整数，通常只有 HDFS 增加新特性时才会更新这个版本号


#### chkpoint 检查时间参数设置 

hdfs-default.xml 设置：

1、通常情况下，SecondaryNameNode 每隔一小时执行一次

```
<property> 
	<name>dfs.namenode.checkpoint.period</name> 
	<value>3600</value>
</property>

```

2、一分钟检查一次操作次数，当操作次数达到 1 百万时，SecondaryNameNode 执行 一次

```
<property> 
	<name>dfs.namenode.checkpoint.txns</name> 
	<value>1000000</value> 
	<description>操作动作次数</description>
</property>

<property> 
	<name>dfs.namenode.checkpoint.check.period</name> 
	<value>60</value>
	<description> 1 分钟检查一次操作次数</description>
</property>

```


#### SecondaryNameNode 目录结构

Secondary NameNode 用来监控 HDFS 状态的辅助后台程序，每隔一段时间获取 HDFS 元数据的快照

在 /opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary/current 这个目录中查看 SecondaryNameNode 目录结构

```
edits_0000000000000000001-0000000000000000002 
fsimage_0000000000000000002 
fsimage_0000000000000000002.md5
VERSION

```

SecondaryNameNode 的 namesecondary/current 目录和主 namenode 的 current 目录的布局相同

好处: 在主 namenode 发生故障时(假设没有及时备份数据)，可以从 SecondaryNameNode 恢复数据


#### Namenode 故障处理方法

Namenode 故障后，可以采用如下两种方法恢复数据:

1、将 SecondaryNameNode 中数据拷贝到 namenode 存储数据的目录

2、使用-importCheckpoint 选项启动 namenode 守护进程，从而将 SecondaryNameNode 中数据拷贝到 namenode 目录中


#### 手动拷贝 SecondaryNameNode 数据

模拟 namenode 故障，并采用方法一，恢复 namenode 数据

1）kill -9 namenode 进程

2）删除 namenode 存储的数据(/opt/module/hadoop-2.7.2/data/tmp/dfs/name)

```
rm -rf /opt/module/hadoop-2.7.2/data/tmp/dfs/name/*

```

3）拷贝 SecondaryNameNode 中数据到原 namenode 存储数据目录

```
scp -r shicong@hadoop104:/opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary/* ./name/

```

4）重新启动 namenode

```
sbin/hadoop-daemon.sh start namenode

```

#### 采用 importCheckpoint 命令拷贝 SecondaryNameNode 数据 

模拟 namenode 故障，并采用方法二，恢复 namenode 数据

1）修改 hdfs-site.xml 中的

```
<property> 
	<name>dfs.namenode.checkpoint.period</name> 
	<value>120</value>
</property>
<property>
	<name>dfs.namenode.name.dir</name> 
	<value>/opt/module/hadoop-2.7.2/data/tmp/dfs/name</value>
</property>

```

2）kill -9 namenode 进程

3）删除 namenode 存储的数据(/opt/module/hadoop-2.7.2/data/tmp/dfs/name)

```
rm -rf /opt/module/hadoop-2.7.2/data/tmp/dfs/name/*

```

4）如果 SecondaryNameNode 不和 Namenode 在一个主机节点上，需要将 SecondaryNameNode 存储数据的目录拷贝到 Namenode 存储数据的平级目录，并删除 in_use.lock 文件

```
scp -r shicong@hadoop104:/opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary ./

```

```
 rm -rf in_use.lock

 ```

 5）导入检查点数据

 ```
 bin/hdfs namenode -importCheckpoint

 ```

 6）启动 namenode

 ```
 sbin/hadoop-daemon.sh start namenode

 ```


#### 集群安全模式操作

Namenode 启动时，首先将映像文件(fsimage)载入内存，并执行编辑日志（edits）中的各项操作，一旦在内存中成功建立文件系统元数据的映像，则创建一个新的 fsimage 文件和一个空的编辑日志。此时，namenode 开始监听 datanode 请求。但是此刻，namenode 运行在安全模式，即 namenode 的文件系统对于客户端来说是只读的。 

系统中的数据块的位置并不是由 namenode 维护的，而是以块列表的形式存储在 datanode 中。在系统的正常操作期间，namenode 会在内存中保留所有块位置的映射信息。 在安全模式下，各个 datanode 会向 namenode 发送最新的块列表信息，namenode 了解到足 够多的块位置信息之后，即可高效运行文件系统。

如果满足“最小副本条件”，namenode 会在 30 秒钟之后就退出安全模式。所谓的最小副 本条件指的是在整个文件系统中 99.9%的块满足最小副本级别(默认值: dfs.replication.min=1)。在启动一个刚刚格式化的 HDFS 集群时，因为系统中还没有任何块， 所以 namenode 不会进入安全模式。

基本语法：

集群处于安全模式，不能执行重要操作(写操作)。集群启动完成后，自动退出安全模式

1）查看安全模式状态：bin/hdfs dfsadmin -safemode get    

2）进入安全模式状态：bin/hdfs dfsadmin -safemode enter 

3）离开安全模式状态：bin/hdfs dfsadmin -safemode leave 

4）等待安全模式状态：bin/hdfs dfsadmin -safemode wait


#### Namenode 多目录配置

namenode 的本地目录可以配置成多个，且每个目录存放内容相同，增加了可靠性。

具体配置如下:

1、在 hdfs-site.xml 文件中增加如下内容

```
<property>
	<name>dfs.namenode.name.dir</name> 
	<value>file:///${hadoop.tmp.dir}/dfs/name1,file:///${hadoop.tmp.dir}/dfs/name2</value> 
</property>

```

2、停止集群，删除每台机器的 data 和 logs 中所有数据

```
rm -rf data/ logs/

```

3、格式化集群并启动

```
bin/hdfs namenode –format

```

```
sbin/start-dfs.sh

```











#### HDFS 产生背景

随着数据量越来越大，在一个操作系统管辖的范围内存不下了，那么就分配到更多的操作，系统管理的磁盘中，但是不方便管理和维护，迫切需要一种系统来管理多台机器上的文件， 这就是分布式文件管理系统。HDFS 只是分布式文件管理系统中的一种。


#### HDFS 概念

HDFS，它是一个文件系统，用于存储文件，通过目录树来定位文件;其次，它是分布 式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。
HDFS 的设计适合一次写入，多次读出的场景，且不支持文件的修改。适合用来做数据 分析，并不适合用来做网盘应用。


#### HDFS 优点

1、高容错性 

数据自动保存多个副本，它通过增加副本的形式，提高容错性 

某一个副本丢失以后，它可以自动恢复

2、适合大数据处理

数据规模: 能够处理数据规模达到 GB、TB、甚至 PB 级别的数据
文件规模: 能够处理百万规模以上的文件数量，数量相当之大

3、流式数据访问

一次写入，多次读取，不能修改，只能追加

它能保证数据的一致性

4、可构建在廉价机器上，通过多副本机制，提高可靠性


#### HDFS 缺点

1、不适合低延时数据访问，比如毫秒级的存储数据，是做不到的

2、无法高效的对大量小文件进行存储

存储大量小文件的话，它会占用 NameNode 大量的内存来存储文件、目录和块信息，这样是不可取的，因为 NameNode 的内存总是有限的

小文件存储的寻址时间会超过读取时间，它违反了 HDFS 的设计目标 

3、并发写入、文件随机修改

一个文件只能有一个写，不允许多个线程同时写

仅支持数据 append(追加)，不支持文件的随机修改


#### HDFS 架构

这种架构主要由四个部分组成，分别为 HDFS Client、NameNode、DataNode 和 Secondary NameNode

1、Client:就是客户端

1）文件切分，文件上传 HDFS 的时候，Client 将文件切分成一个一个的 Block，然后进行存储
2）与 NameNode 交互，获取文件的位置信息
3）与 DataNode 交互，读取或者写入数据
4）Client 提供一些命令来管理 HDFS，比如启动或者关闭 HDFS。 (5)Client 可以通过一些命令来访问 HDFS

2、NameNode:就是 master，它是一个主管、管理者

1）管理 HDFS 的名称空间
2）管理数据块(Block)映射信息 
3）配置副本策略 
4）处理客户端读写请求

3、DataNode:就是 Slave，NameNode 下达命令，DataNode 执行实际的操作

1）存储实际的数据块
2）执行数据块的读/写操作

4、Secondary NameNode:并非 NameNode 的热备，当 NameNode 挂掉的时候，它并不能马上替换 NameNode 并提供服务

1）辅助 NameNode，分担其工作量
2）定期合并 Fsimage 和 Edits，并推送给 NameNode。 (3)在紧急情况下，可辅助恢复 NameNode


#### HDFS 文件块大小

HDFS 中的文件在物理上是分块存储(block)，块的大小可以通过配置参数( dfs.blocksize) 来规定，默认大小在 hadoop2.x 版本中是 128M，老版本中是 64M

HDFS 的块比磁盘的块大，其目的是为了最小化寻址开销。如果块设置得足够大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。因而，传输一个由多个块组成的文件的时间取决于磁盘传输速率

如果寻址时间约为 10ms，而传输速率为 100MB/s，为了使寻址时间仅占传输时间的 1%， 我们要将块大小设置约为 100MB。默认的块大小 128MB

块的大小:10ms*100*100M/s = 100M


#### HFDS 命令行操作

基本语法：bin/hadoopfs 具体命令


#### 常用命令

1、启动 Hadoop 集群

sbin/start-dfs.sh

sbin/start-yarn.sh

2、-help: 输出这个命令参数

hadoop fs -help rm

3、-ls: 显示目录信息

hadoop fs -ls /

4、-mkdir:在 hdfs 上创建目录

hadoop fs -mkdir -p /user/test

5、-moveFromLocal 从本地剪切粘贴到 hdfs

hadoop fs -moveFromLocal ./test.txt /user/test

6、--appendToFile :追加一个文件到已经存在的文件末尾

hadoop fs -appendToFile test1.txt /user/test/test.txt

7、-cat :显示文件内容

8、-tail :显示一个文件的末尾

hadoop fs -tail /user/test/test.txt

9、-chgrp 、-chmod、-chown:linux 文件系统中的用法一样，修改文件所属权限

hadoop fs -chmod 666 /user/test/test.txt

hadoop fs -chown shicong:shicong /user/test/test.txt

10、-copyFromLocal: 从本地文件系统中拷贝文件到 hdfs 路径去

hadoop fs -copyFromLocal README.txt /user/test

11、-copyToLocal:  从 hdfs 拷贝到本地

hadoop fs -copyToLocal /user/test/test.txt ./test.txt

12、-cp :从 hdfs 的一个路径拷贝到 hdfs 的另一个路径

hadoop fs -cp /user/test/test.txt /test1.txt

13、-mv:在 hdfs 目录中移动文件

hadoop fs -mv /test.txt /user/test/

14、-get:等同于 copyToLocal，就是从 hdfs 下载文件到本地

hadoop fs -get /user/test/test.txt ./

15、-getmerge :合并下载多个文件，比如 hdfs 的目录 /aaa/下有多个文件:log.1, log.2,log.3,...

hadoop fs -getmerge /user/test/* ./all.txt

16、-put:等同于 copyFromLocal

hadoop fs -put ./all.txt /user/test/

17、-rm :删除文件或文件夹

hadoop fs -rm /user/test/test.txt

18、-rmdir :删除空目录

hadoop fs -rmdir /test

19、-df :统计文件系统的可用空间信息

hadoop fs -df -h /

20、-du 统计文件夹的大小信息

hadoop fs -du -s -h /user/test 

hadoop fs -du -h /user/test

21、-setrep  :设置 hdfs 中文件的副本数量

hadoop fs -setrep 2 /user/test/test.txt

这里设置的副本数只是记录在 namenode 的元数据中，是否真的会有这么多副本，还得看 datanode 的数量。因为目前只有 3 台设备，最多也就 3 个副本，只有节点数的增加到 10 台时，副本数才能达到 10。


#### 通过 API 操作 HDFS

客户端去操作 hdfs 时，是有一个用户身份的，默认情况下，hdfs 客户端 api 会从 jvm 中获取一个参数来作为自己的用户身份:-DHADOOP_USER_NAME=shicong，shicong 为用户 名称。

1、HDFS 获取文件系统

```
public void initHDFS() throws Exception{
// 1 创建配置信息对象
Configuration configuration = new Configuration();

// 2 获取文件系统
FileSystem fs = FileSystem.get(configuration);

// 3 打印文件系统
System.out.println(fs.toString()); 
}
```

2、HDFS 文件上传

```
public void testCopyFromLocalFile() throws IOException, InterruptedException, URISyntaxException {
// 1 获取文件系统
Configuration configuration = new Configuration(); 
configuration.set("dfs.replication", "2");
FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration,"shicong");

// 2 上传文件
fs.copyFromLocalFile(new Path("e:/hello.txt"), new Path("/hello5.txt"));

// 3 关闭资源 
fs.close();
System.out.println("over"); 
}
```

测试参数优先级:

客户端代码中设置的值 > classpath 下的用户自定义配置文件 > 服务器的默认配置

3、HDFS 文件下载

```
public void testCopyToLocalFile() throws IOException, InterruptedException, URISyntaxException{

// 1 获取文件系统
Configuration configuration = new Configuration();
FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration,"shicong");

// 2 执行下载操作
// boolean delSrc 指是否将原文件删除
// Path src 指要下载的文件路径
// Path dst 指将文件下载到的路径
// boolean useRawLocalFileSystem 是否开启文件效验 
fs.copyToLocalFile(false, new Path("/hello1.txt"), new Path("e:/hello1.txt"), true);

// 3 关闭资源
fs.close(); 
}
```

4、HDFS 目录创建

```
public void testMkdirs() throws IOException, InterruptedException, URISyntaxException{

// 1 获取文件系统
Configuration configuration = new Configuration();
FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration,"shicong");

// 2 创建目录
fs.mkdirs(new Path("/0906/test/test1"));

// 3 关闭资源
fs.close(); 
}
```

5、HDFS 文件夹删除

```
public void testDelete() throws IOException, InterruptedException, URISyntaxException{

// 1 获取文件系统
Configuration configuration = new Configuration();
FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration,"shicong");

// 2 执行删除
fs.delete(new Path("/0906/"), true);

// 3 关闭资源
fs.close(); 
}
```

6、HDFS 文件名更改

```
public void testRename() throws IOException, InterruptedException, URISyntaxException{

// 1 获取文件系统
Configuration configuration = new Configuration();
FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration,"shicong");

// 2 修改文件名称
fs.rename(new Path("/hello.txt"), new Path("/hello6.txt"));

// 3 关闭资源
fs.close(); 
}
```

7、HDFS 文件详情查看

查看文件名称、权限、长度、块信息

```
public void testListFiles() throws IOException, InterruptedException, URISyntaxException{

	// 1 获取文件系统
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration,"shicong");

	// 2 获取文件详情
	RemoteIterator<LocatedFileStatus> listFiles = fs.listFiles(new Path("/"), true);
	while(listFiles.hasNext()){
		LocatedFileStatus status = listFiles.next();

		// 输出详情
		// 文件名称 
		System.out.println(status.getPath().getName()); 

		// 长度
		System.out.println(status.getLen());

		// 权限 
		System.out.println(status.getPermission());

		// 组
		System.out.println(status.getGroup());

		// 获取存储的块信息
		BlockLocation[] blockLocations = status.getBlockLocations();
		for (BlockLocation blockLocation : blockLocations) {

			// 获取块存储的主机节点
			String[] hosts = blockLocation.getHosts();
			for (String host : hosts) { 
				System.out.println(host);
			} 
		}
	System.out.println("----------------班长的分割线-----------");
	}
}
```

8、HDFS 文件和文件夹判断

```
public void testListStatus() throws IOException, InterruptedException, URISyntaxException{
	// 1 获取文件配置信息
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration,"shicong");
	
	// 2 判断是文件还是文件夹
	FileStatus[] listStatus = fs.listStatus(new Path("/"));
	for (FileStatus fileStatus : listStatus) {
	
		// 如果是文件
		if (fileStatus.isFile()) {
			System.out.println("f:"+fileStatus.getPath().getName()); 
		}else {
			System.out.println("d:"+fileStatus.getPath().getName()); 
			}
	}
	// 3 关闭资源
	fs.close(); 
}

```


#### HDFS 写数据流程

1、客户端通过 Distributed FileSystem 模块向 namenode 请求上传文件，namenode 检查目标 文件是否已存在，父目录是否存在。

2、namenode 返回是否可以上传

3、客户端请求第一个 block 上传到哪几个 datanode 服务器上

4、namenode 返回 3 个 datanode 节点，分别为 dn1、dn2、dn3

5、客户端通过 FSDataOutputStream 模块请求 dn1 上传数据，dn1 收到请求会继续调用 dn2， 然后 dn2 调用 dn3，将这个通信管道建立完成

6、dn1、dn2、dn3 逐级应答客户端

7、客户端开始往 dn1 上传第一个 block(先从磁盘读取数据放到一个本地内存缓存)，以 packet 为单位，dn1 收到一个 packet 就会传给 dn2，dn2 传给 dn3;dn1 每传一个 packet 会放 入一个应答队列等待应答

8、当一个 block 传输完成之后，客户端再次请求 namenode 上传第二个 block 的服务器。(重 复执行 3-7 步)


#### 机架感知

1、低版本 Hadoop 副本节点选择

第一个副本在 client 所处的节点上，如果客户端在集群外，随机选一个

第二个副本和第一个副本位于不相同机架的随机节点上

第三个副本和第二个副本位于相同机架，节点随机。


2、Hadoop2.7.2 副本节点选择

第一个副本在 client 所处的节点上，如果客户端在集群外，随机选一个

第二个副本和第一个副本位于相同机架，随机节点

第三个副本位于不同机架，随机节点


HDFS 满足客户端访问副本数据的最近原则，即客户端距离哪个副本数据最近，HDFS 就让哪个节点把数据给客户端。


#### HDFS 读数据流程

1、客户端通过 Distributed FileSystem 向 namenode 请求下载文件，namenode 通过查询元数 据，找到文件块所在的 datanode 地址。

2、挑选一台 datanode(就近原则，然后随机)服务器，请求读取数据。

3、datanode 开始传输数据给客户端(从磁盘里面读取数据输入流，以 packet 为单位来做校 验)。

4、客户端以 packet 为单位接收，先在本地缓存，然后写入目标文件。








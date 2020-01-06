

#### SSH 无密码登录配置

基本语法：ssh 另一台电脑的 ip 地址

1、无密钥配置

进入到我的 home 目录:

```
cd ~/.ssh
```

2、生成公钥和私钥:

```
 ssh-keygen -t rsa
 ```

然后敲(三个回车)，就会生成两个文件 id_rsa(私钥)、id_rsa.pub(公钥)

3、将公钥拷贝到要免密登录的目标机器上:

```
ssh-copy-id hadoop103
```

```
ssh-copy-id hadoop104
```

.ssh 文件夹下的文件功能解释

1）~/.ssh/known_hosts :记录 ssh 访问过计算机的公钥(public key) 

2）id_rsa :生成的私钥

3）id_rsa.pub :生成的公钥

4）authorized_keys :存放授权过得无秘登录服务器公钥


#### rsync

rsync 远程同步工具，主要用于备份和镜像，具有速度快、避免复制相同内容和支持符 号链接的优点。

rsync 和 scp 区别: 用 rsync 做文件的复制要比 scp 的速度快，rsync 只对差异文件做更新

scp 是把所有文件都复制过去

查看 rsync 使用说明 

```
man rsync | more
```

基本语法:

命令 命令参数 要拷贝的文件路径/名称 目的用户@主机:目的路径

```
rsync -rvl $pdir/$fname $user@hadoop$host:$pdir
```

选项:

-r 递归

-v 显示复制过程

-l 拷贝符号连接

案例:

把本机/opt/tmp 目录同步到 hadoop103 服务器的 root 用户下的/opt/tmp 目录

```
rsync -rvl /opt/tmp root@hadoop103:/opt/
```

#### 集群分发脚本 xsync

需求:循环复制文件到所有节点的相同目录下。 

原始拷贝:

```
rsync -rvl /opt/module root@hadoop103:/opt/ 
```

xsync 要同步的文件名称

在/usr/local/bin 这个目录下存放的脚本，可以在系统任何地方直接执行。

案例:

在/usr/local/bin 目录下创建 xsync 文件，文件内容如下:

创建文件：

```
touch xsync
```

```
#!/bin/bash

#1 获取输入参数个数，如果没有参数，直接退出 
pcount=$#
if((pcount==0)); then
echo no args;
exit;
fi

#2 获取文件名称 
p1=$1 
fname=`basename $p1` 
echo fname=$fname

#3 获取上级目录到绝对路径 
pdir=`cd -P $(dirname $p1); pwd` 
echo pdir=$pdir

#4 获取当前用户名称 
user=`whoami`

#5 循环
for((host=103; host<105; host++)); do
#echo $pdir/$fname $user@hadoop$host:$pdir 
echo --------------- hadoop$host ---------------- 
rsync -rvl $pdir/$fname $user@hadoop$host:$pdir
done
```

修改脚本 xsync 具有执行权限:

```
chmod 777 xsync
```

修改脚本 xsync 拥有者:

```
chown shicong:shicong -R xsync
```

调用脚本形式:xsync 文件名称:

```
xsync tmp/
```

#### 集群操作脚本 xcall

需求: 在所有主机上同时执行相同的命令 xcall +命令

在/usr/local/bin 目录下创建 xcall 文件，文件内容如下:

创建文件：

```
touch xcall
```

```
#!/bin/bash 
pcount=$# 
if((pcount==0));then
echo no args;
exit; 
fi
echo -------------localhost----------
$@
for((host=101; host<=108; host++)); do
echo ----------hadoop$host---------
ssh hadoop$host $@ 
done
```

修改脚本 xcall 具有执行权限:

```
chmod 777 xcall
```

修改脚本 xsync 拥有者:

```
chown shicong:shicong xcall
```

调用脚本形式: xcall 操作命令:

```
xcall rm -rf /opt/tmp/
```

#### 配置集群

hadoop102: NameNode DataNode NodeManager

hadoop103: DataNode ResourceManager NodeManager

hadoop104: SecondaryNameNode DataNode NodeManager

配置文件 core-site.xml

```
<!-- 指定 HDFS 中 NameNode 的地址 --> 
<property>
	<name>fs.defaultFS</name>
	<value>hdfs://hadoop102:9000</value> 
</property>

<!-- 指定 hadoop 运行时产生文件的存储目录 --> 
<property>
	<name>hadoop.tmp.dir</name>
	<value>/opt/module/hadoop-2.7.2/data/tmp</value> 
</property>
```

配置文件 hadoop-env.sh，在文件末尾添加

```
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

配置文件 hdfs-site.xml

```
<configuration> 
	<property>
		<name>dfs.replication</name>
		<value>3</value> 
	</property>

	<property> 
		<name>dfs.namenode.secondary.http-address</name> 
		<value>hadoop104:50090</value>
	</property>
</configuration>
```

配置 slaves

```
hadoop102
hadoop103
hadoop104
```

配置 yarn-env.sh

```
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

配置 yarn-site.xml

```
<configuration>
	<!-- reducer 获取数据的方式 --> 
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value> 
	</property>

<!-- 指定 YARN 的 ResourceManager 的地址 --> 
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>hadoop103</value> 
	</property>
</configuration>
```

配置 mapred-env.sh

```
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

配置 mapred-site.xml

```
<configuration>
<!-- 指定 mr 运行在 yarn 上 --> 
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value> 
	</property>
</configuration>
```

在集群上分发以上所有文件:

```
xsync /opt/module/hadoop-2.7.2/etc/hadoop/core-site.xml
```

```
xsync /opt/module/hadoop-2.7.2/etc/hadoop/yarn-site.xml
```

```
xsync /opt/module/hadoop-2.7.2/etc/hadoop/slaves
```

查看文件分发情况:

```
xcall cat /opt/module/hadoop-2.7.2/etc/hadoop/slaves
```


#### 集群启动及测试

启动集群

如果集群是第一次启动，需要格式化 namenode

```
bin/hdfs namenode -format
```

启动 HDFS:

```
sbin/start-dfs.sh

```

启动 yarn:

```
sbin/start-yarn.sh
```

查看进程情况：

```
jps
```

注意:Namenode 和 ResourceManger 如果不是同一台机器，不能在 NameNode 上启动 yarn，应该在 ResouceManager 所在的机器上启动 yarn。


集群基本测试

上传小文件:

```
bin/hdfs dfs -mkdir -p /user/shicong/tmp/conf
```

```
bin/hdfs dfs -put etc/hadoop/*-site.xml /user/shicong/tmp/conf
```


#### Hadoop 启动停止方式

1、各个服务组件逐一启动

1）分别启动 hdfs 组件

hadoop-daemon.sh start|stop namenode|datanode|secondarynamenode 

2）分别启动 yarn

yarn-daemon.sh start|stop resourcemanager|nodemanager

2、各个模块分开启动(配置 ssh 是前提)

1）整体启动/停止 hdfs

start-dfs.sh

stop-dfs.sh

2）整体启动/停止 yarn

start-yarn.sh

stop-yarn.sh 

3、全部启动/停止(不建议使用)

start-all.sh 

stop-all.sh


#### 集群时间同步

时间同步的方式: 找一个机器，作为时间服务器，所有的机器与这台集群时间进行定时 的同步，比如，每隔十分钟，同步一次时间。

时间服务器配置(必须 root 用户) 

检查 ntp 是否安装

```
rpm -qa|grep ntp
```

修改 ntp 配置文件，vi /etc/ntp.conf，修改内容如下：


修改 1，也就是去掉这行的注释

#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap 为:

```
restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap 
```

修改 2，也就是给以下三行加上注释

server 0.centos.pool.ntp.org iburst 
server 1.centos.pool.ntp.org iburst 
server 2.centos.pool.ntp.org iburst 
server 3.centos.pool.ntp.org iburst 为:

```
#server 0.centos.pool.ntp.org iburst 
#server 1.centos.pool.ntp.org iburst 
#server 2.centos.pool.ntp.org iburst 
#server 3.centos.pool.ntp.org iburst
```

添加 3

```
server 127.127.1.0
fudge 127.127.1.0 stratum 10
```

修改/etc/sysconfig/ntpd 文件：

vim /etc/sysconfig/ntpd

增加内容如下：

SYNC_HWCLOCK=yes

查看 ntpd 状态：

```
service ntpd status
```

重新启动 ntpd:

```
service ntpd start
```

设置开机启动：

```
chkconfig ntpd on
```

其他机器配置(必须 root 用户):

在其他机器配置 10 分钟与时间服务器同步一次

```
crontab -e
```

编写脚本

```
*/10 * * * * /usr/sbin/ntpdate hadoop102

```

修改任意机器时间:

```
date -s "2020-01-06 22:10:11"
```

十分钟后查看机器是否与时间服务器同步:

```
date
```

#### 配置集群常见问题

1）防火墙没关闭、或者没有启动 yarn

INFO client.RMProxy: Connecting to ResourceManager at hadoop108/192.168.10.108:8032

2）主机名称配置错误

3）ip 地址配置错误

4）ssh 没有配置好

5）root 用户和 atguigu 两个用户启动集群不统一 

6）配置文件修改不细心

7）未编译源码

Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
17/05/22 15:38:58 INFO client.RMProxy: Connecting to ResourceManager at hadoop108/192.168.10.108:8032

8）datanode 不被 namenode 识别问题

Namenode 在 format 初始化的时候会形成两个标识，blockPoolId 和 clusterId

新的 datanode 加入时，会获取这两个标识作为自己工作目录中的标识。

一旦 namenode 重新 format 后，namenode 的身份标识已变，而 datanode 如果依然持有 原来的 id，就不会被 namenode 识别。
解决办法，删除 datanode 节点中的数据后，再次重新格式化 namenode。 

9）不识别主机名称

解决办法:

1、在/etc/hosts 文件中添加 192.168.1.102 hadoop102 

2、主机名称不要起 hadoop hadoop000 等特殊名称

10）datanode 和 namenode 进程同时只能工作一个。

11）执行命令不生效，粘贴 word 中命令时，遇到-和长–没区分开，导致命令失效

解决办法: 尽量不要粘贴 word 中代码。

12）jps 发现进程已经没有，但是重新启动集群，提示进程已经开启。原因是在 linux 的根目 录下/tmp 目录中存在启动的进程临时文件，将集群相关进程删除掉，再重新启动集群。 

13）jps 不生效

原因:全局变量 hadoop java 没有生效，需要 source /etc/profile 文件。

14）8088 端口连接不上

查看 cat /etc/hosts

注释掉如下代码：

#127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
#::1 hadoop102


















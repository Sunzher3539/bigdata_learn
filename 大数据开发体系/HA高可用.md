在进行配置前，确保您已经配置好了hadoop3.3.6集群
## 1、下载Zookeeper
前往清华源网站[Index of /apache/zookeeper/zookeeper-3.9.2 (tsinghua.edu.cn)](https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.9.2/)
下载zookeeper3.9.2，上传至master节点并解压缩至/usr/loacl
```
tar -zxf apache-zookeeper-3.9.2-bin.tar.gz -C /usr/local/
```
## 2、修改配置文件
### 1、配置系统变量
```shell
vi /etc/profile
```
追加：
```shell
export ZOOKEEPER_HOME=/usr/local/apache-zookeeper-3.9.2
export PATH=$HADOOP_HOME/bin:$PATH:$JAVA_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin
```
![[Pasted image 20240610115821.png]]
然后分发到各个节点
```shell
for SLAVE in slave1 slave2 slave3; do scp /etc/profile $SLAVE:/etc/profile; done
```
在各个节点执行命令 
```shell
source /etc/profile
```
### 2、编写zoo.cfg

```shell
cd /usr/local/apache-zookeeper-3.9.2/config
vi zoo_sample.cfg
```
将配置文件修改为如下内容
```shell
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/usr/local/apache-zookeeper-3.9.2/data/
dataLogDir=/usr/local/apache-zookeeper-3.9.2/log/
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
maxClientCnxns=2000
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
autopurge.snapRetainCount=10
# Purge task interval in hours
# Set to "0" to disable auto purge feature
autopurge.purgeInterval=24

## Metrics Providers
#
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpHost=0.0.0.0
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true

# Machine Message
server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888
server.4=slave3:2888:3888
```
然后重命名
```shell
mv zoo_sample.cfg zoo.cfg
```
前往/usr/local/apache-zookeeper-3.9.2下
```shell
mkdir data
mkdir log
cd data
vi myip
```
myip内容
```shell
1
```
目的是告诉zookeeper自己是几号机器
#### 分发/usr/local/apache-zookeeper-3.9.2
```shell
for SLAVE in slave1 slave2 slave3 slave4; do scp -r /usr/local/apache-zookeeper-3.9.2 $SLAVE:/usr/local/; done
```
前往从节点的data文件夹，将myip修改为对应的号码
比如slave1就是2号机器了
## 3.尝试运行zookeeper集群
```shell
# 一键启动zookeeper集群，脚本，文章末尾有脚本配置
zkcontrol start
checkjps
```
确认集群状态
![[Pasted image 20240610143447.png]]
## 4、hadoopHA高可用配置
### 1、修改core-site.xml
```xml
<configuration>
<!-- 单节点hadoop -->
 <!--  <property>
    <name>fs.defaultFS</name>  
      <value>hdfs://master:8020</value>  
      </property>  -->
<!--指定hadoop数据的存储目录-->
    <property>
      <name>hadoop.tmp.dir</name>
      <value>/var/log/hadoop/tmp</value>
    </property>
<!--配置所有节点的atguigu用户都可作为代理用户-->
<property>
    <name>hadoop.proxyuser.atguigu.hosts</name>
    <value>*</value>
</property>

<!--配置atguigu用户能够代理的用户组为任意组-->
<property>
    <name>hadoop.proxyuser.atguigu.groups</name>
    <value>*</value>
</property>

<!--配置atguigu用户能够代理的用户为任意用户-->
<property>
    <name>hadoop.proxyuser.atguigu.users</name>
    <value>*</value>
</property>
    <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
    </property>
<!--以下为HA配置参数-->
<!-- Namenode高可用配置-自定义集群名称,且不用指定端口号 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hacluster</value>
    </property>
<!-- 配置ZKFC进程连接zookeeper的地址 -->
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>master:2181,slave1:2181,slave2:2181,slave3:2181</value>
    </property>
<!--修改core-site.xml中的ipc参数,防止出现连接journalnode服务ConnectException-->
<property>
     <name>ipc.client.connect.max.retries</name>
     <value>100</value>
     <description>Indicates the number of retries a client will make to establish a server connection.</description>
 </property>
<property>
    <name>ipc.client.connect.retry.interval</name>
    <value>10000</value>
     <description>Indicates the number of milliseconds a client will wait for before retrying to establish a server connection.</description>
 </property>
</configuration>
```
### 2、修改hdfs-site.xml
```xml
<configuration>
<!-- namenode节点目录 -->
<property>
    <name>dfs.namenode.name.dir</name>
    <value>/data/hadoop/hdfs/name</value>
</property>
<!-- datanode节点数据存放目录 -->
<property>
    <name>dfs.datanode.data.dir</name>
    <value>/data/hadoop/hdfs/data</value>
</property>
<!-- journalnode 数据存储目录-->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/data/hadoop/hdfs/jn</value>
</property>
<!-- 第二节点访问地址 -->
<!--<property>
     <name>dfs.namenode.secondary.http-address</name>
     <value>master:50090</value>
</property> -->
<!-- 数据副本数 -->
<property>
     <name>dfs.replication</name>
     <value>3</value>
</property>

<!-- 指定集群逻辑名字为hacluster 与 core-site中保持一致 -->
    <property>
        <name>dfs.nameservices</name>
        <value>hacluster</value>
</property>
<!-- 指定逻辑namenode的节点，自定义名称为nn1，2，3 -->
<property>
        <name>dfs.ha.namenodes.hacluster</name>
        <value>nn1,nn2,nn3,nn4</value>
</property>
<!-- 定义nn1的rpc通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.hacluster.nn1</name>
        <value>master:8020</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-address.hacluster.nn2</name>
        <value>slave1:8020</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-address.hacluster.nn3</name>
        <value>slave2:8020</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-address.hacluster.nn4</name>
        <value>slave3:8020</value>
    </property>

<!-- 定义nn1的web页面的http地址 -->
    <property>
        <name>dfs.namenode.http-address.hacluster.nn1</name>
        <value>master:9870</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.hacluster.nn2</name>
        <value>slave1:9870</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.hacluster.nn3</name>
        <value>slave2:9870</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.hacluster.nn4</name>
        <value>slave3:9870</value>
    </property>

    <!-- 定义namenode元数据在journalnode上的存放位置 -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://master:8485;slave1:8485;slave2:8485;slave3:8485/hacluster</value>
    </property>

    <!-- 访问代理类：访问hacluster时，client用于确定哪个NameNode为Active，将请求转发过去 -->
    <property>
        <name>dfs.client.failover.proxy.provider.hacluster</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
<!-- 配置隔离机制 -->
    <property>
        <name>dfs.ha.fencing.methods</name>
        <value>
	sshfence
	shell(/bin/true)
	</value>
    </property>
<!-- 配置隔离机制时需要的ssh密钥登录 -->
    <property>
        <name>dfs.ha.fencing.ssh.private-key-files</name>
        <value>/root/.ssh/id_rsa.pub</value>
    </property>
    <!-- 启动namenode故障自动转移 -->
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
    <!-- 关闭权限检查 -->
<property>
    <name>dfs.permissions.enabled</name>
    <value>false</value>
</property>
</configuration>
```
### 3、修改yarn-site.xml
```xml
<!-- 指定ResourceManager的地址 单节点配置-->
  <!-- <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>master</value>
  </property>    -->
<!-- 环境变量的继承 -->
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>

    <!-- 开启resourcemanager HA-->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>

    <!-- 自定义一个resourcemanager的逻辑集群id-->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>yarn-cluster</value>
    </property>

   <!-- 指定resourcemanager集群的逻辑节点名称列表-->
<!-- 本来应该配置4个，但考虑到没必要，所以就只配制了3个-->
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2,rm3</value>
    </property>

<!-- 单节点配置-->
   <!--  <property>
    <name>yarn.resourcemanager.address</name>
    <value>${yarn.resourcemanager.hostname}:8032</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>${yarn.resourcemanager.hostname}:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>${yarn.resourcemanager.hostname}:8088</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.https.address</name>
    <value>${yarn.resourcemanager.hostname}:8090</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>${yarn.resourcemanager.hostname}:8031</value>
  </property>
  <property>
    <name>yarn.resourcemanager.admin.address</name>
    <value>${yarn.resourcemanager.hostname}:8033</value>
  </property> -->


<!-- rm1的节点信息-->
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>master</value>
    </property>
    <!-- yarn web页面地址  -->
    <property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>master:8088</value>
    </property>
    <!-- rm1 对客户端暴露的地址，客户端通过该地址向RM提交任务等 -->
    <property>
        <name>yarn.resourcemanager.address.rm1</name>
        <value>master:8032</value>
    </property>
    <!-- rm1 与 applicationMaster的通信地址  -->
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm1</name>
        <value>master:8030</value>
    </property>
    <!-- rm1 与 nm的通信地址  -->
    <property>
        <name>yarn.resourcemanager.resource-tracker.address.rm1</name>
        <value>master:8031</value>
    </property>


<!-- rm2的节点信息-->
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>slave1</value>
    </property>
    <!-- yarn web页面地址  -->
    <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>slave1:8088</value>
    </property>
    <!-- rm2 对客户端暴露的地址，客户端通过该地址向RM提交任务等 -->
    <property>
        <name>yarn.resourcemanager.address.rm2</name>
        <value>slave1:8032</value>
    </property>
    <!-- rm2 与 applicationMaster的通信地址  -->
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm2</name>
        <value>slave1:8030</value>
    </property>
    <!-- rm2 与 nm的通信地址  -->
    <property>
        <name>yarn.resourcemanager.resource-tracker.address.rm2</name>
        <value>slave1:8031</value>
    </property>


<!-- rm3的节点信息-->
    <property>
        <name>yarn.resourcemanager.hostname.rm3</name>
        <value>slave2</value>
    </property>
    <!-- yarn web页面地址  -->
    <property>
        <name>yarn.resourcemanager.webapp.address.rm3</name>
        <value>slave2:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address.rm3</name>
        <value>slave2:8032</value>
    </property>
    <!-- rm3 与 applicationMaster的通信地址  -->
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm3</name>
        <value>slave2:8030</value>
    </property>
    <!-- rm3 与 nm的通信地址  -->
    <property>
        <name>yarn.resourcemanager.resource-tracker.address.rm3</name>
        <value>sslave2:8031</value>
    </property>

  <property>
    <name>yarn.nodemanager.local-dirs</name>
    <value>/data/hadoop/yarn/local</value>
  </property>
   <!-- 启用日志聚合  -->
  <property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
  </property>
  <property>
    <name>yarn.nodemanager.remote-app-log-dir</name>
    <value>/data/tmp/logs</value>
  </property>
<property> 
 <name>yarn.log.server.url</name> 
 <value>http://master:19888/jobhistory/logs/</value>
 <description>URL for job history server</description>
</property>
<property>
   <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
  </property>
<!-- 配置zookeeper信息  -->
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>master:2181,slave1:2181,slave2:2181</value>
    </property>

   <!-- 启动自动恢复 -->
    <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
    </property>

   <!-- 配置将recourcemanager的状态信息存储在zookeeper中 -->
    <property>
        <name>yarn.resourcemanager.store.class</name>
        <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
    </property>

<!-- 指定MR走shuffle -->
 <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
      <value>org.apache.hadoop.mapred.ShuffleHandler</value>
      </property>
<property>  
        <name>yarn.nodemanager.resource.memory-mb</name>  
        <value>2048</value>  
 </property>  
 <property>  
        <name>yarn.scheduler.minimum-allocation-mb</name>  
        <value>512</value>  
 </property>   
 <property>  
        <name>yarn.scheduler.maximum-allocation-mb</name>  
        <value>4096</value>  
 </property> 
 <property> 
    <name>mapreduce.map.memory.mb</name> 
    <value>2048</value> 
 </property> 
 <property> 
    <name>mapreduce.reduce.memory.mb</name> 
    <value>2048</value> 
 </property> 
 <property> 
    <name>yarn.nodemanager.resource.cpu-vcores</name> 
    <value>1</value> 
 </property>
<property>
    <name>yarn.application.classpath</name>
    <value>
/usr/local/hadoop-3.3.6/etc/hadoop:/usr/local/hadoop-3.3.6/share/hadoop/common/lib/*:/usr/local/hadoop-3.3.6/share/hadoop/common/*:/usr/local/hadoop-3.3.6/share/hadoop/hdfs:/usr/local/hadoop-3.3.6/share/hadoop/hdfs/lib/*:/usr/local/hadoop-3.3.6/share/hadoop/hdfs/*:/usr/local/hadoop-3.3.6/share/hadoop/mapreduce/lib/*:/usr/local/hadoop-3.3.6/share/hadoop/mapreduce/*:/usr/local/hadoop-3.3.6/share/hadoop/yarn:/usr/local/hadoop-3.3.6/share/hadoop/yarn/lib/*:/usr/local/hadoop-3.3.6/share/hadoop/yarn/*
    </value>
  </property>

<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认
     是 true -->
<property>
 <name>yarn.nodemanager.pmem-check-enabled</name>
 <value>false</value>
</property>
<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认
     是 true -->
<property>
 <name>yarn.nodemanager.vmem-check-enabled</name>
 <value>false</value>
</property>
</configuration>

```
### 4、分发文件
```shell
# 分发yarn-site.xml
for SLAVE in slave1 slave2 slave3; do scp /usr/local/hadoop-3.3.6/etc/hadoop/yarn-site.xml $SLAVE:/usr/local/hadoop-3.3.6/etc/hadoop/yarn-site.xml; done
# 分发core-site.xml
for SLAVE in slave1 slave2 slave3; do scp /usr/local/hadoop-3.3.6/etc/hadoop/core-site.xml $SLAVE:/usr/local/hadoop-3.3.6/etc/hadoop/core-site.xml; done
# 分发hdfs-site.xml
for SLAVE in slave1 slave2 slave3; do scp /usr/local/hadoop-3.3.6/etc/hadoop/hdfs-site.xml $SLAVE:/usr/local/hadoop-3.3.6/etc/hadoop/hdfs-site.xml; done
```
## 5、集群启动
```shell
zkcontrol start
hjn start
checkjps
```
![[Pasted image 20240610160317.png]]
出现上图启动成功
## 6、HA集群初始化
### 1、在第一台namenode节点上格式化namenode
```shell
hdfs namenode -format
```
启动这台节点的namenode
```shell
hdfs --daemon start namenode
```
 其他Namenode节点上首次要手动同步一次数据
 在其他节点上运行
 ```shell
 hdfs namenode -bootstrapStandby
```
同步完成后启动namenode
```shell
hdfs --daemon start namenode
```
### 2、初始化ZKFC
ZKFC用于监控active namenode节点是否挂掉，通知其它节点上的ZKFC强行杀死自己ZKFC节点上的namenode（防止其假死状态产生集群namenode脑裂的发生），然后选举出其他namenode为active节点
集群首次搭建需要在zookeeper中初始化namenode信息，在namenode1节点执行命令：
```shell
hdfs zkfc -formatZK
```
### 3、启动hadoop集群
```shell
myhadoop start
```
### 4、查看节点状态
```shell
hdfs haadmin -getAllServiceState
```
### 5、验证高可用
kill -9 active namenode进程，查看页面状态，可发现另外某个namenode自动切换成active状态。

        记住kill掉的，实验结束后再启动起来。
 如果验证成功，那我们的HA高可用就配置完成了


## 编写便捷脚本
### 1、一键启动zookeeper集群
```shell
vi /bin/zkcontrol
```
输入
```shell
#!/bin/bash

# 定义Zookeeper节点列表
ZOOKEEPER_NODES=("master" "slave1" "slave2" "slave3")

# Zookeeper安装路径
ZOOKEEPER_HOME=$ZOOKEEPER_HOME

# 检查是否提供了start或stop参数
if [ "$#" -ne 1 ]; then
    echo "Usage: $0 {start|stop}"
    exit 1
fi

ACTION=$1

# 验证输入参数
if [ "$ACTION" != "start" ] && [ "$ACTION" != "stop" ]; then
    echo "Invalid action: $ACTION"
    echo "Usage: $0 {start|stop}"
    exit 1
fi

# 启动或停止Zookeeper集群
for NODE in "${ZOOKEEPER_NODES[@]}"
do
    echo "$ACTION Zookeeper on $NODE"
    ssh $NODE "$ZOOKEEPER_HOME/bin/zkServer.sh $ACTION"
    if [ $? -eq 0 ]; then
        echo "Successfully ${ACTION}ed Zookeeper on $NODE"
    else
        echo "Failed to $ACTION Zookeeper on $NODE"
    fi
done

# 验证Zookeeper服务状态
for NODE in "${ZOOKEEPER_NODES[@]}"
do
    echo "Checking Zookeeper status on $NODE"
    ssh $NODE "$ZOOKEEPER_HOME/bin/zkServer.sh status"
done


```
给予权限
```shell
chmod 777 zkcontrol
```
用法：
```shell
# 启动集群
zkcontrol start
# 关闭集群
zkcontrol stop
```
### 2、编写一键启动Hadoop JournalNode脚本
```shell
cd /bin
vi hjn
```
内容如下
```shell
#!/bin/bash

# 定义JournalNode节点列表
JOURNALNODE_NODES=("master" "slave1" "slave2" "slave3")

# Hadoop安装路径
HADOOP_HOME=$HADOOP_HOME

# 检查是否提供了start或stop参数
if [ "$#" -ne 1 ]; then
    echo "Usage: $0 {start|stop}"
    exit 1
fi

ACTION=$1

# 验证输入参数
if [ "$ACTION" != "start" ] && [ "$ACTION" != "stop" ]; then
    echo "Invalid action: $ACTION"
    echo "Usage: $0 {start|stop}"
    exit 1
fi

# 启动或停止JournalNode服务
for NODE in "${JOURNALNODE_NODES[@]}"
do
    echo "$ACTION JournalNode on $NODE"
    ssh $NODE "hdfs --daemon $ACTION journalnode"
    if [ $? -eq 0 ]; then
        echo "Successfully ${ACTION}ed JournalNode on $NODE"
    else
        echo "Failed to $ACTION JournalNode on $NODE"
    fi
done

```
```shell
chmod 777 hjn
```
用法
```shell
# 启动集群Hadoop JournalNode
hjn start
# 关闭集群Hadoop JournalNode
hjn stop
```

本教程安装hbase
安装版本为hbase-2.6.0-hadoop3-bin.tar.gz
## 1、下载habase
前往[Index of /apache/hbase/2.6.0 (tsinghua.edu.cn)](https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.6.0/)
下载hbase-2.6.0-hadoop3-bin.tar.gz
上传至/opt目录下
解压缩至/usr/local/目录下
```shell
tar -zxf hbase-2.6.0-hadoop3-bin.tar.gz -C /usr/local/
cd /usr/local
mv hbase-2.6.0-hadoop3 hbase
```
## 2、配置hbase
### 1、配置hbase-site.xml
```shell
vi /usr/local/hbase/conf/hbase-site.xml
```
编辑如下内容
```xml
  	<property>
		<!--    hbase集群模式    -->
    		<name>hbase.cluster.distributed</name>
    		<value>true</value>
  	</property>
  		<!-- Namenode高可用配置-自定义集群名称,且不用指定端口号 -->
    	<property>
        	<name>fs.defaultFS</name>
        	<value>hdfs://hacluster</value>
    	</property>
	<property>
		<!--    zookeeper集群    -->
    		<name>hbase.zookeeper.quorum</name>
    		<value>master:2181,slave1:2181,slave2:2181,slave3:2181</value>
  	</property>
	<property>
		<!--    habase根目录    -->
    		<name>hbase.rootdir</name>
    		<value>hdfs://hacluster:8020/hbase</value>
  	</property>
  	<property>
    		<name>hbase.tmp.dir</name>
    		<value>./tmp</value>
  	</property>
	<property>
		<!--    配置web可创建删除文件    -->
  		<name>hbase.wal.provider</name>
  		<value>filesystem</value>
	</property>
	<property>
		<!--    默认HMaster HTTP访问端口    -->
                <name>hbase.master.info.port</name>
                <value>16010</value>
	</property>
    	<property>
		<!--    默认regionserver HTTP访问端口    -->
       		<name>hbase.regionserver.info.port</name>
       		<value>16030</value>
	</property>
  	<property>
    		<name>hbase.unsafe.stream.capability.enforce</name>
    		<value>true</value>
  	</property>
	<property>
		<!--    配置文件副本数2    -->
     		<name>dfs.replication</name>
       		<value>2</value>
 	</property>
</configuration>
```
### 2、配置hbase-env.sh
```shell
vi /usr/local/hbase/conf/hbase-env.sh
```
追加
```shell
export JAVA_HOME=/usr/java/jdk1.8.0_281-amd64
export HBASE_CLASSPATH=/usr/local/hadoop-3.3.6/etc/hadoop
export HBASE_MANAGES_ZK=false
```
### 3、配置regionservers
```shell
vi /usr/local/hbase/conf/regionservers
```
覆写
```
master
slave1
slave2
slave3
```
### 4、配置backup-masters
为保持高可用
```shell
vi /usr/local/hbase/conf/backup-masters
```
填充
```
slave1
```
### 5、复制hadoop的配置文件
复制hadoop的配置文件core-site.xml，hdfs-site.xml到conf文件夹下
## 3、分发hbase及配置环境变量
### 1、分发hbase
```shell
for SLAVE in slave1 slave2 slave3; do scp -r /usr/local/hbase/ $SLAVE:/usr/local/hbase/; done
```
### 2、配置环境变量
```shell
vi /etc/profile
```
追加
```
# hbase 路径
export HBASE_HOME=/usr/local/hbase
export PATH=$PATH:$HBASE_HOME/bin
```
分发/etc/profile文件
```shell
for SLAVE in slave1 slave2 slave3; do scp /etc/profile $SLAVE:/etc/profile; done
```

资源生效，需对每个节点执行一次
```shell
source /etc/profile
```
## 4、启动集群
### 1、启动zookeeper和hadoop集群
```shell
zkcontrol start
myhadoop start
myhbase start
```
### 遇到binding错误
这属于jar包冲突，移除hbase jar包
每个节点都要移除jar包
```shell
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hadoop-3.3.6/share/hadoop/common/lib/slf4j-reload4j-1.7.36.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hbase/lib/client-facing-thirdparty/log4j-slf4j-impl-2.17.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Reload4jLoggerFactory]

```
前往/usr/local/hbase/lib/client-facing-thirdparty/
```shell
mv log4j-slf4j-impl-2.17.2.jar log4j-slf4j-impl-2.17.2.jar.bak
```
## 编写一键启动hbase集群
```shell
vi /bin/myhbase
```
填充
```shell
#!/bin/bash
if [ "$1" == "start" ]; then
    cd /usr/local/hbase/bin/
    ./start-hbase.sh
elif [ "$1" == "stop" ]; then
    cd /usr/local/hbase/bin/
    ./stop-hbase.sh
else
    echo "Invalid argument. Please provide 'start' or 'stop'."
fi
```
命令生效
```shell
chmod 777 myhbase
```

## 优化设置：网上参考，未实际配置
5.5 基础优化
允许在 HDFS 的文件中追加内容

hdfs-site.xml、hbase-site.xml
属性： dfs.support.append
解释： 开启 HDFS 追加同步，可以优秀的配合 HBase 的数据同步和持久化。默认值为 true。

优化 DataNode 允许的最大文件打开数

hdfs-site.xml
属性： dfs.datanode.max.transfer.threads
解释： HBase 一般都会同一时间操作大量的文件，根据集群的数量和规模以及数据动作，设置为 4096 或者更高。默认值：4096

优化延迟高的数据操作的等待时间

hdfs-site.xml
属性： dfs.image.transfer.timeout
解释： 如果对于某一次数据操作来讲，延迟非常高，socket 需要等待更长的时间，建议把
该值设置为更大的值（默认 60000 毫秒），以确保 socket 不会被 timeout 掉。

优化数据的写入效率

mapred-site.xml
属性： mapreduce.map.output.compress、mapreduce.map.output.compress.codec
解释： 开启这两个数据可以大大提高文件的写入效率，减少写入时间。第一个属性值修改为 true，第二个属性值修改为：org.apache.hadoop.io.compress.GzipCodec 或者其他压缩方式。

设置 RPC 监听数量

hbase-site.xml
属性： Hbase.regionserver.handler.count
解释： 默认值为 30，用于指定 RPC 监听的数量，可以根据客户端的请求数进行调整，读写请求较多时，增加此值。

优化 HStore 文件大小

hbase-site.xml
属性： hbase.hregion.max.filesize
解释： 默认值 10737418240（10GB），如果需要运行 HBase 的 MR 任务，可以减小此值，因为一个 region 对应一个 map 任务，如果单个 region 过大，会导致 map 任务执行时间过长。该值的意思就是，如果 HFile 的大小达到这个数值，则这个 region 会被切分为两个 Hfile。

优化 HBase 客户端缓存

hbase-site.xml
属性： hbase.client.write.buffer
解释： 用于指定 Hbase 客户端缓存，增大该值可以减少 RPC 调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到减少 RPC 次数的目的。

指定 scan.next 扫描 HBase 所获取的行数

hbase-site.xml
属性： hbase.client.scanner.caching
解释： 用于指定 scan.next 方法获取的默认行数，值越大，消耗内存越大。

flush、compact、split 机制

当 MemStore 达到阈值，将 Memstore 中的数据 Flush 进 Storefile；
compact 机制则是把 flush出来的小文件合并成大的 Storefile 文件；
split 则是当 Region 达到阈值，会把过大的 Region一分为二。

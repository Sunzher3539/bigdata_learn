## 1、获取scala2.13.8和spark3.5.1
### 获取scala2.13.8
[Scala 2.13.8 | The Scala Programming Language (scala-lang.org)](https://www.scala-lang.org/download/2.13.8.html)
下载[scala-2.13.8.rpm](https://downloads.lightbend.com/scala/2.13.8/scala-2.13.8.rpm)并上传至linux的/opt文件下
### 获取spark3.5.1
[Index of /apache/spark/spark-3.5.1 (tsinghua.edu.cn)](https://mirrors.tuna.tsinghua.edu.cn/apache/spark/spark-3.5.1/)
下载[spark-3.5.1-bin-hadoop3-scala2.13.tgz](https://mirrors.tuna.tsinghua.edu.cn/apache/spark/spark-3.5.1/spark-3.5.1-bin-hadoop3-scala2.13.tgz)
上传至linux的/opt文件夹下
## 2、安装scala2.13.8
### 1、安装scala
```shell
cd /opt
rpm -ivh scala-2.13.8.rpm
```
### 2、配置Scala环境变量
```shell
vi /etc/profile
```
追加
```
# scala 路径
export SCALA_HOME=/usr/share/scala
export PATH=$PATH:$SCALA_HOME/bin
```
分发
```shell
for SLAVE in slave1 slave2 slave3; do scp /etc/profile $SLAVE:/etc/profile; done
```
资源生效，需对每个节点执行一次
```shell
source /etc/profile
```
## 3、安装Spark3.5.1 yarn模式HA
### 1、解压缩spark3.5.1
```shell
cd /opt
tar -zxvf spark-3.5.1-bin-hadoop3-scala2.13 -C /usr/local/
```
重命名
```shell
cd /usr/local
mv spark-3.5.1-bin-hadoop3-scala2.13 spark-yarn
```
### 2、配置spark环境变量
```
vi etc/profile
```
追加
```
# spark 路径
export SPARK_HOME=/usr/local/spark-yarn
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin 
```
分发
```shell
for SLAVE in slave1 slave2 slave3; do scp /etc/profile $SLAVE:/etc/profile; done
```
资源生效，需对每个节点执行一次
```shell
source /etc/profile
```
### 3、配置spark-env.sh
```shell
cd /usr/local/spark-yarn/conf/
cp spark-env.sh.template spark-env.sh
vi spark-env.sh
```
在开头bin下追加
```
#!/usr/bin/env bash

#JAVAhome
export JAVA_HOME=/usr/java/jdk1.8.0_281-amd64
# 指定hadoophome
export HADOOP_CONF_DIR=/usr/local/hadoop-3.3.6/etc/hadoop
export YARN_CONF_DIR=/usr/local/hadoop-3.3.6/etc/hadoop 
# 指定默认master的ip或主机名
# export SPARK_MASTER_HOST=master
#指定maaster提交任务的默认端口为7077    
export SPARK_MASTER_PORT=7077
#指定masster节点的webui端口       
export SPARK_MASTER_WEBUI_PORT=8081
#每个worker从节点能够支配的内存数 
export SPARK_WORKER_MEMORY=1g
#允许Spark应用程序在计算机上使用的核心总数（默认值：所有可用核心）
export SPARK_WORKER_CORES=1
#每个worker从节点的实例（可选配置） 
export SPARK_WORKER_INSTANCES=1
# worker 的工作通讯地址
SPARK_WORKER_PORT=7078
# worker 的 webui 地址
SPARK_WORKER_WEBUI_PORT=8082
#指向包含Hadoop集群的（客户端）配置文件的目录，运行在Yarn上配置此项   
export HADOOP_CONF_DIR="-Dspark.history.fs.logDirectory=hdfs://hacluster/sparklog/ -Dspark.history.fs.cleaner.enabled=true"
#指定整个集群状态是通过zookeeper来维护的，包括集群恢复
export SPARK_DAEMON_JAVA_OPTS="      
-Dspark.deploy.recoveryMode=ZOOKEEPER 
-Dspark.deploy.zookeeper.url=master:2181,slave1:2181,slave2:2181,slave3:2181
-Dspark.deploy.zookeeper.dir=/spark/data"
```
### 4、创建日志
启动集群，创建日志文件夹
```shell
hadoop fs -mkdir /sparklog
hadoop fs -chmod 777 /sparklog
```
### 5、配置spark-defaults.conf
```shell
cd /usr/local/spark-yarn/conf
mv spark-defaults.conf.template spark-defaults.conf
vi spark-defaults.conf
```
for SLAVE in slave1 slave2 slave3; do scp /usr/local/spark-yarn/conf/spark-defaults.conf $SLAVE:/usr/local/spark-yarn/conf/; done

追加
```
# 指定提交到 yarn 运行
spark.master                             yarn
# 开启spark的日期记录功能
spark.eventLog.enabled  true
# # 设置spark日志记录的路径
# spark.eventLog.dir       hdfs://hadoop01:8021/sparklog/
# # 设置spark日志是否启动压缩
spark.eventLog.compress         true
 
# hadoop HA 的配置
spark.eventLog.dir       hdfs://hacluster/sparklog/
```
### 6、配置log4j.properties
```shell
mv log4j2.properties.template log4j2.properties
vi log4j2.properties
```
```
# 将这一行值修改为WARN
rootLogger.level = WARN
```
## 4、分发spark-yarn
```shell
for SLAVE in slave1 slave2 slave3; do scp -r /usr/local/spark-yarn/ $SLAVE:/usr/local/spark-yarn/; done
```
## 配置快速启动spark脚本
```shell
vi /bin/myspark
#!/bin/bash
if [ "$1" == "start" ]; then
    cd /usr/local/spark-yarn/sbin/
    ./start-all.sh
./start-history-server.sh
elif [ "$1" == "stop" ]; then
    cd /usr/local/spark-yarn/sbin/
    ./stop-all.sh
./stop-history-server.sh
else
    echo "Invalid argument. Please provide 'start' or 'stop'."
fi

chmod 777 myspark

```

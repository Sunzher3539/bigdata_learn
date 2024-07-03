本文配置dolphinscheduler3.2.0
## 1、下载dolphinscheduler3.2.0
前往[Index of /apache/dolphinscheduler/3.2.0 (tsinghua.edu.cn)](https://mirrors.tuna.tsinghua.edu.cn/apache/dolphinscheduler/3.2.0/)
下载apache-dolphinscheduler-3.2.0-bin.tar.gz
上传至/opt目录下
解压至/usr/local目录下
```shell
tar -zxf apache-dolphinscheduler-3.2.0-bin.tar.gz -C /usr/local/
cd /usr/local
mv apache-dolphinscheduler-3.2.0-bin/ dolphinscheduler
```
## 2、配置文件
### 1、添加MYSQL驱动
我的mysql-8.3.0
驱动为
此处使用 MySQL 8.2.0版本，对应使用 JDBC 驱动为 mysql-connector-j.jar，将该驱动移动至 DolphinScheduler 的每个模块下的 libs 目录下。共5个目录：
1. api-server/libs
2. alert-server/libs
3. master-server/libs
4. worker-server/libs
5. tools/libs

### 2、dolphinscheduler_env.sh 配置
```shell
vim /usr/local/dolphinscheduler/bin/env/dolphinscheduler_env.sh
```
```shell
# MySQL数据库配置
export DATABASE=${DATABASE:-mysql}
export SPRING_PROFILES_ACTIVE=${DATABASE}
export SPRING_DATASOURCE_URL="jdbc:mysql://192.168.128.140:3306/dolphinscheduler?useUnicode=true&characterEncoding=UTF-8&useSSL=false&allowPublicKeyRetrieval=true"
export SPRING_DATASOURCE_USERNAME=${SPRING_DATASOURCE_USERNAME:-"root"}
export SPRING_DATASOURCE_PASSWORD=${SPRING_DATASOURCE_PASSWORD:-"123456"}
# java配置
export JAVA_HOME=/usr/java/jdk1.8.0_281-amd64
# DolphinScheduler server related configuration
export SPRING_CACHE_TYPE=${SPRING_CACHE_TYPE:-none}
export SPRING_JACKSON_TIME_ZONE=${SPRING_JACKSON_TIME_ZONE:-UTC}
export MASTER_FETCH_COMMAND_NUM=${MASTER_FETCH_COMMAND_NUM:-10}
# Registry center configuration, determines the type and link of the registry center
# zk注册中心
export REGISTRY_TYPE=${REGISTRY_TYPE:-zookeeper}
export REGISTRY_ZOOKEEPER_CONNECT_STRING=${REGISTRY_ZOOKEEPER_CONNECT_STRING:-master:2181,slave1:2181,slave2:2181,slave3:2181}
export YARN_PROXYSERVER_USER=root
export HDFS_ZKFC_USER=root
export HDFS_JOURNALNODE_USER=root
# Hadoop-site
export HADOOP_HOME=/usr/local/hadoop-3.3.6
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
# zookeeper 路径
export ZOOKEEPER_HOME=/usr/local/apache-zookeeper-3.9.2
# 主环境路径
export PATH=$HADOOP_HOME/bin:$PATH:$JAVA_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin
# 以下路径根据自己的组件配置即可
# HIVE路径
export HIVE_HOME=/usr/local/hive 
export PATH=$PATH:$HIVE_HOME/bin
# scala 路径
export SCALA_HOME=/usr/share/scala
export PATH=$PATH:$SCALA_HOME/bin
# spark 路径
export SPARK_HOME=/usr/local/spark-yarn
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
# flink 路径
export FLINK_HOME=/usr/local/flink
export PATH=$PATH:$FLINK_HOME/bin 
# hbase 路径
export HBASE_HOME=/usr/local/hbase
export PATH=$PATH:$HBASE_HOME/bin
# kafka 路径
export KAFKA_HOME=/usr/local/kafka
export PATH=$PATH:$KAFKA_HOME/bin
# sea-tunnel 路径
export SEATUNNEL_HOME=/usr/local/sea-tunnel/apache-seatunnel-2.3.5
export PATH=$PATH:$SEATUNNEL_HOME/bin
```
### 3、install_env.sh文件修改
```shell
vim /usr/local/dolphinscheduler/bin/env/install_env.sh
```
```shell
# 按照集群部署方案，内容如下：

# 集群节点
ips=${ips:-"master,slave1,slave2,slav3"}

# ssh免密端口,使用默认
sshPort=${sshPort:-"22"}

# master节点
masters=${masters:-"master,slave1"}

# worker节点
workers=${workers:-"slave1:default,salve2:default,salve3:default"}

# alert节点
alertServer=${alertServer:-"slave2"}

# api节点
apiServers=${apiServers:-"slave3"}

# dolphinscheduler实际安装路径
installPath=${installPath:-"/usr/local/dolphinscheduler-real"}

# 部署dolphinscheduler使用的用户名
deployUser=${deployUser:-"dolphinscheduler"}

# zk根节点
zkRoot=${zkRoot:-"/dolphinscheduler"}

```
## 3、初始化元数据
 切换到dolphinscheduler目录下，执行命令
```shell
 sh ./tools/bin/upgrade-schema.sh
```
此操作，会向MySQL数据库写入元数据，共计65张表，如图所示：
![[Pasted image 20240629115149.png]]
## 4、安装执行
```shell
./bin/install.sh
```
安装完后出现
![[Pasted image 20240629120600.png]]
即为成功
## 5、访问web平台
[Dolphin Scheduler Admin](http://slave3:12345/dolphinscheduler/ui/ui-setting)
账户:admin
密码:dolphinscheduler123
![[Pasted image 20240629120705.png]]
**提示：**
> 安装完成后，此时安装用到的 dolphinscheduler 文件就没用了。
> 此时，已经将 DS 安装到配置中指定的/usr/local/dolphinscheduler-real目录下了
## 常用指令
### 一键停止集群所有服务
sh /opt/soft/dolphinscheduler-3.2.0/bin/stop-all.sh

### 一键启动集群所有服务
sh /opt/soft/dolphinscheduler-3.2.0/bin/start-all.sh

### 启/停 master 服务
sh /opt/soft/dolphinscheduler-3.2.0/bin/dolphinscheduler-daemon.sh start master-server 
sh /opt/soft/dolphinscheduler-3.2.0/bin/dolphinscheduler-daemon.sh stop master-server 

### 启/停 worker 服务
sh /opt/soft/dolphinscheduler-3.2.0/bin/dolphinscheduler-daemon.sh start worker-server 
sh /opt/soft/dolphinscheduler-3.2.0/bin/dolphinscheduler-daemon.sh stop worker-server 

### 启/停 api 服务
sh /opt/soft/dolphinscheduler-3.2.0/bin/dolphinscheduler-daemon.sh start api-server 
sh /opt/soft/dolphinscheduler-3.2.0/bin/dolphinscheduler-daemon.sh stop api-server 

### 启/停 alert 服务
sh /opt/soft/dolphinscheduler-3.2.0/bin/dolphinscheduler-daemon.sh start alert-server 
sh /opt/soft/dolphinscheduler-3.2.0/bin/dolphinscheduler-daemon.sh stop alert-server 

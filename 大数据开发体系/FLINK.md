本教程安装flink
flink-1.19.0-bin-scala_2.12.tgz
## 1、下载flink
前往[Index of /apache/flink/flink-1.19.0 (tsinghua.edu.cn)](https://mirrors.tuna.tsinghua.edu.cn/apache/flink/flink-1.19.0/)
下载flink-1.19.0-bin-scala_2.12.tgz
上传至/opt目录下
解压缩至/usr/local/文件夹下
```shell
tar -zxf flink-1.19.0-bin-scala_2.12.tgz -C /usr/local/
cd /usr/local
mv flink-1.19.0/ flink
```
## 2、配置config.yaml
```shell
cd flink
vi /conf/config.yaml
```
### 1、注释掉java高版本环境
```
# These parameters are required for Java 17 support.
# They can be safely removed when using Java 8/11.
# env:
#  java:
#    opts:
#      all: --add-exports=java.base/sun.net.util=ALL-UNNAMED --add-exports=java.rmi/sun.rmi.registry=ALL-UNNAMED --add-exports=jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED --add-exports=jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED --add-exports=jdk.compiler/com.sun.tools.javac.parser=ALL-UNNAMED --add-exports=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED --add-exports=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED --add-exports=java.security.jgss/sun.security.krb5=ALL-UNNAMED --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.net=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.nio=ALL-UNNAMED --add-opens=java.base/sun.nio.ch=ALL-UNNAMED --add-opens=java.base/java.lang.reflect=ALL-UNNAMED --add-opens=java.base/java.text=ALL-UNNAMED --add-opens=java.base/java.time=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens=java.base/java.util.concurrent.locks=ALL-UNNAMED
```
### 2、在Commen中追加
```xml
#==============================================================================
# Common
jobmanager.rpc.address: master
#==============================================================================
```
### 3、配置jobmanager
```
jobmanager:
  # The host interface the JobManager will bind to. By default, this is localhost, and will prevent
  # the JobManager from communicating outside the machine/container it is running on.
  # On YARN this setting will be ignored if it is set to 'localhost', defaulting to 0.0.0.0.
  # On Kubernetes this setting will be ignored, defaulting to 0.0.0.0.
  #
  # To enable this, set the bind-host address to one that has access to an outside facing network
  # interface, such as 0.0.0.0.
  # 注释bind-host，为了可以上传jar包到集群
  # bind-host: master
  rpc:
    # The external address of the host on which the JobManager runs and can be
    # reached by the TaskManagers and any clients which want to connect. This setting
    # is only used in Standalone mode and may be overwritten on the JobManager side
    # by specifying the --host <hostname> parameter of the bin/jobmanager.sh executable.
    # In high availability mode, if you use the bin/start-cluster.sh script and setup
    # the conf/masters file, this will be taken care of automatically. Yarn
    # automatically configure the host name based on the hostname of the node where the
    # JobManager runs.
    # 配置内通信接口指向地址master：6123
    address: master
    # The RPC port where the JobManager is reachable.
    port: 6123
  memory:
    process:
      # The total process memory size for the JobManager.
      # Note this accounts for all memory usage within the JobManager process, including JVM metaspace and other overhead.
      # 配置JVM内存
      size: 1024m
  execution:
    # The failover strategy, i.e., how the job computation recovers from task failures.
    # Only restart tasks that may have been affected by the task failure, which typically includes
    # downstream tasks and potentially upstream tasks if their produced data is no longer available for consumption.
    failover-strategy: region
```
### 4、配置taskmanager
```
taskmanager:
  # The host interface the TaskManager will bind to. By default, this is localhost, and will prevent
  # the TaskManager from communicating outside the machine/container it is running on.
  # On YARN this setting will be ignored if it is set to 'localhost', defaulting to 0.0.0.0.
  # On Kubernetes this setting will be ignored, defaulting to 0.0.0.0.
  #
  # To enable this, set the bind-host address to one that has access to an outside facing network
  # interface, such as 0.0.0.0.
  # 配置taskmanager启动地址，每个节点需配置自己的地址
  bind-host: master
  # The address of the host on which the TaskManager runs and can be reached by the JobManager and
  # other TaskManagers. If not specified, the TaskManager will try different strategies to identify
  # the address.
  #
  # Note this address needs to be reachable by the JobManager and forward traffic to one of
  # the interfaces the TaskManager is bound to (see 'taskmanager.bind-host').
  #
  # Note also that unless all TaskManagers are running on the same machine, this address needs to be
  # configured separately for each TaskManager.
  # # 配置taskmanager启动地址，每个节点需配置自己的地址
  host: localhost
  # The number of task slots that each TaskManager offers. Each slot runs one parallel pipeline.
  # 槽数
  numberOfTaskSlots: 4
  memory:
    process:
      # The total process memory size for the TaskManager.
      #
      # Note this accounts for all memory usage within the TaskManager process, including JVM metaspace and other overhead.
      # To exclude JVM metaspace and overhead, please, use total Flink memory size instead of 'taskmanager.memory.process.size'.
      # It is not recommended to set both 'taskmanager.memory.process.size' and Flink memory.
      # task最大内存
      size: 1728m

parallelism:
  # The parallelism used for programs that did not specify and other parallelism.
  # 并行度
  default: 4

```
### 5、配置webui
```
rest:
  # The address to which the REST client will connect to
  # webui client访问地址  
  address: master
  # The address that the REST & web server binds to
  # By default, this is localhost, which prevents the REST & web server from
  # being able to communicate outside of the machine/container it is running on.
  #
  # To enable this, set the bind address to one that has access to outside-facing
  # network interface, such as 0.0.0.0.
  # webui对外访问地址
  bind-address: master
  # # The port to which the REST client connects to. If rest.bind-port has
  # # not been specified, then the server will bind to this port as well.
  # 对外访问端口
  port: 8081
  # # Port range for the REST and web server to bind to.
  bind-port: 8080-8090
web:
  submit:
	# 开启web提交作业功能，可以网页上传jar哦！
    # Flag to specify whether job submission is enabled from the web-based
    # runtime monitor. Uncomment to disable.
    enable: true
  cancel:
    # 开启web取消作业功能，可以网页杀死作业哦！
    # Flag to specify whether job cancellation is enabled from the web-based
    # runtime monitor. Uncomment to disable.
    enable_cancellation: true  # 重命名键

```
## 3、配置zoo.cfg,masters,workers
### 1、配置zoo.cfg
高可用的配置，但是没配置成功
```shell
vi /conf/zoo.cfg
```
将配置更改为
```xml
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

### 2、配置masters
```shell
vi /conf/masters
```
内容为
```
master:8081
```
### 3、配置workers
```shell
vi /conf/workers
```
内容为
```
slave1
slave2
slave3
```
### 4、分发flink
```shell
for SLAVE in slave1 slave2 slave3; do scp -r /usr/local/flink/ $SLAVE:/usr/local/flink/; done
for SLAVE in slave1 slave2 slave3; do scp -r /usr/local/flink/conf/config.yaml $SLAVE:/usr/local/flink/conf/config.yaml; done
# 记得分发config.yaml后修改本地地址
```
```bash
# 测试 flink on yarn
./bin/yarn-session.sh --detached
./bin/flink run ./examples/streaming/TopSpeedWindowing.jar
```
## 5、启动flink

### 1、集群启动
```shell
/usr/local/flink/bin/start-cluster.sh
```
### 2、flink on yarn启动集群session模式
其实也就是调用yarn资源模拟一个集群
对yarn配置要求很高，很容易启动不成功
```shell
cd /usr/local/flink
./bin/yarn-session.sh --detached
```
如果启动后查看yarn中哪个节点负责启动，到对应节点node：8081查看webui查看集群启动情况
### 3、flink on yarn启动集群job模式
仅仅用于执行flink作业，无法在webui上查看集群情况

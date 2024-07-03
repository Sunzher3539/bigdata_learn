本教程采用apache-seatunnel-2.3.5-bin.tar.gz
没有选择flume，dataX等技术的原因是因为seatunnel技术更先进，更好
## 1、下载seatunnel
前往[Index of /apache/seatunnel/2.3.5 (tsinghua.edu.cn)](https://mirrors.tuna.tsinghua.edu.cn/apache/seatunnel/2.3.5/)
下载apache-seatunnel-2.3.5-bin.tar.gz
上传至/opt文件夹下
解压缩
```shell
mkdir /usr/local/sea-tunnel
tar -zxf apache-seatunnel-2.3.5-bin.tar.gz -C /usr/local/sea-tunnel
```
## 2、配置seatunnel
### 1、配置环境变量
```shell
vi /etc/profile
```
追加
```
# sea-tunnel 路径
export SEATUNNEL_HOME=/usr/local/sea-tunnel/apache-seatunnel-2.3.5
export PATH=$PATH:$SEATUNNEL_HOME/bin
```

分发/etc/profile文件
```shell
for SLAVE in slave1 slave2 slave3; do scp /etc/profile $SLAVE:/etc/profile; done
```

资源生效，需对每个节点执行一次
```shell
source /etc/profile
```
### 2、下载jar包
#### 1、在/usr/local/sea-tunnel/apache-seatunnel-2.3.5/connectors创建下创建一些目录
```shell
mkdir flink  
mkdir flink-sql  
mkdir seatunnel  
mkdir spark
```
#### 2、自动下载jar包
下载在config/plugin_config配置的seatunnel的连接器到connectors/seatunnel文件下
```shell
vi /usr/local/sea-tunnel/apache-seatunnel-2.3.5/bin/install-plugin.sh
```
```
# 修改其中代码，将文件下载位置定位到lib中
${SEATUNNEL_HOME}/mvnw dependency:get -DgroupId=org.apache.seatunnel -DartifactId=${line} -Dversion=${version} -Ddest=${SEATUNNEL_HOME}/lib
```
在下载之前，可以对config/plugin_config进行编辑，注释不需要的connector，可以添加需要的connector
我这边全量下载
#### 3、开始下载


### 3、修改启动脚本
1. 修改启动脚本`$SEATUNNEL_HOME/bin/seatunnel-cluster.sh`，首行增加java内存配置`JAVA_OPTS="-Xms2G -Xmx2G"`
### 4、修改配置文件
#### 1、seatunnel.yaml 
有相关HA配置
```
seatunnel:
  engine:
    history-job-expire-minutes: 1440
    classloader-cache-mode: true
    backup-count: 1
    queue-type: blockingqueue
    print-execution-info-interval: 60
    print-job-metrics-info-interval: 60
    slot-service:
      dynamic-slot: true
    checkpoint:
      interval: 300000
      timeout: 60000
      storage:
        type: hdfs
        max-retained: 3
        plugin-config:
          namespace: /tmp/seatunnel/checkpoint_snapshot
          storage.type: hdfs
          fs.defaultFS: hdfs://hacluster
          seatunnel.hadoop.dfs.nameservices: hacluster
          seatunnel.hadoop.dfs.ha.namenodes.emr-cluster: nn1,nn2,nn3,nn4
          seatunnel.hadoop.dfs.namenode.rpc-address.emr-cluster.nn1: 192.168.128.140:8020
          seatunnel.hadoop.dfs.namenode.rpc-address.emr-cluster.nn2: 192.168.128.141:8020
          seatunnel.hadoop.dfs.namenode.rpc-address.emr-cluster.nn3: 192.168.128.142:8020
          seatunnel.hadoop.dfs.namenode.rpc-address.emr-cluster.nn4: 192.168.128.143:8020
          seatunnel.hadoop.dfs.client.failover.proxy.provider.emr-cluster: org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider
```
#### 2、hazelcast.yaml
```
hazelcast:
  cluster-name: seatunnel
  network:
    rest-api:
      enabled: true
      endpoint-groups:
        CLUSTER_WRITE:
          enabled: true
        DATA:
          enabled: true
    join:
      tcp-ip:
        enabled: true
        member-list:
          - master
          - slave1
          - slave2
          - slave3
    port:
      auto-increment: false
      port: 5801
  properties:
    hazelcast.invocation.max.retry.count: 20
    hazelcast.tcp.join.port.try.count: 30
    hazelcast.logging.type: log4j2
    hazelcast.operation.generic.thread.count: 50
```
3、hazelcast-client.yaml
```
hazelcast-client:
  cluster-name: seatunnel
  properties:
    hazelcast.logging.type: log4j2
  connection-strategy:
    connection-retry:
      cluster-connect-timeout-millis: 3000
  network:
    cluster-members: 
          - master:5801
          - slave1:5801
          - slave2:5801
          - slave3:5801
```
### 5、创建logs文件
```shell
mkdir -p $SEATUNNEL_HOME/logs
```
### 6、分发节点
for SLAVE in slave1 slave2 slave3; do scp -r /usr/local/sea-tunnel/apache-seatunnel-2.3.5 $SLAVE:/usr/local/sea-tunnel/; done

## 启动seatunnel

服务启动[基于Seatunnel最新2.3.5版本分布式集群安装部署指南(小白版)_seatunnel 集群部署-CSDN博客](https://blog.csdn.net/qq_41865652/article/details/139514250)

7.Seatunnel集群常用管理命令
1)启动集群（需要在集群每台节点上都执行）
nohup $SEATUNNEL_HOME/bin/seatunnel-cluster.sh 2>&1 &

2)停止集群（需要在集群每台节点上都执行）
$SEATUNNEL_HOME/bin/stop-seatunnel-cluster.sh

3)默认引擎任务提交命令


$SEATUNNEL_HOME/bin/seatunnel.sh --config /opt/software/seatunnel-2.3.5/config/app-config/v2.batch.config.template

5)spark3.X版本引擎任务提交命令


$SEATUNNEL_HOME/bin/start-seatunnel-spark-3-connector-v2.sh --master local[4] --deploy-mode client --config /opt/software/seatunnel-2.3.5/config/app-config/v2.batch.config.template

6)flink低版本引擎任务提交命令(Flink版本1.12.x到1.14.x)

$SEATUNNEL_HOME/bin/start-seatunnel-flink-13-connector-v2.sh --config /opt/software/seatunnel-2.3.5/config/app-config/v2.streaming.conf.template

7)flink高版本引擎任务提交命令(Flink版本1.15.x到1.16.x)

$SEATUNNEL_HOME/bin/start-seatunnel-flink-15-connector-v2.sh --config /opt/software/seatunnel-2.3.5/config/app-config/v2.streaming.conf.template

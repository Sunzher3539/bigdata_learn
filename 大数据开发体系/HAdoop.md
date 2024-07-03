# 集群规划

| master | slave1 | slave2 | slave3 |
| ------ | ------ | ------ | ------ |
|        |        |        |        |

启动步骤
# 1、分布式协调服务ZooKeeper集群启动
```shell
zkcontrol start/stop
```
# 2、分布式文件系统Hadoop集群启动
```shell
myhadoop start/stop
```
# 3、验证节点状态
```shell
chenckjps
```
验证节点状态
```shell
hdfs haadmin -getAllServiceState
```
# 4、数据仓库服务启动
```shell
hiveservice
```
# 5、大数据处理引擎SPARK集群启动
```shell
myspark start/stop
```
# 6、流处理框架flink集群（可选）
与spark有冲突
```shell
/usr/local/flink/bin/start-cluster.sh
```
# 7、分布式的数据库HABASE集群启动
```shell
myhbase start/stop
```
# 8、分布式流处理平台和消息队列系统Kafka集群启动
```shell
mykafka start/stop
```
本地连接工具Kafka tool
[Offset Explorer (kafkatool.com)](https://www.kafkatool.com/download.html)
# 9、分布式数据集成Seatunnel集群启动
```shell
#启动集群（需要在集群每台节点上都执行）
nohup $SEATUNNEL_HOME/bin/seatunnel-cluster.sh 2>&1 &
#关闭集群（需要在集群每台节点上都执行）
$SEATUNNEL_HOME/bin/stop-seatunnel-cluster.sh
```
# 10、分布式可视化 DAG 工作流任务调度开源系统DolphinScheduler集群启动
```
myds start/stop
```


web访问方式
hadoopweb访问：http:master:9870
hiveweb访问：http:master:10020
SPARK master访问:http:master:8081
SPARK worker访问:http:slave1:8082
hbase HMaster HTTP访问端口：master:16010
hbase regionserver HTTP访问端口：master:16030
sea-tunnel web访问端口：52013
DolphinScheduler:配置的时候是在slave3上开启web[Dolphin Scheduler Admin](http://slave3:12345/dolphinscheduler/ui/ui-setting)

hadoop账号信息
LINUX虚拟机器统一密码
账户:root
密码:SUNZHER
mysql账户密码
账户:root
密码:123456
HIVE账户密码
账户：root
密码：123456
DolphinScheduler
账户:admin
密码:dolphinscheduler123
## 1、下载Hadoop3.3.6
前往官网：[Index of /apache/hadoop/common/hadoop-3.3.6 (tsinghua.edu.cn)](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.3.6/)
下载[hadoop-3.3.6-src.tar.gz](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.3.6/hadoop-3.3.6-src.tar.gz)到本地即可
## 2、关闭linux防火墙与selinux
1.查看防火墙状态
```shell
systemctl status firewalld
```
![[Pasted image 20240609164938.png]]
可以看到默认是开启的状态
我们需要把他关闭
```shell
systemctl disable firewalld
```
2.关闭selinux
```shell
vi /etc/sysconfig/selinux
```
将SELINUX=disable
![[Pasted image 20240609165329.png]]
3.reboot,重启虚拟机
## 3、上传Hadoop3.3.6
1. 通过xmanager的Xftp上传hadoop-3.3.6.tar.gz文件到/opt目录
2. 解压缩hadoop-3.3.6.tar.gz 文件
```shell
tar -zxf hadoop-3.3.6.tar.gz -C /usr/local/
```
3. 解压后即可，看到 /usr/local/hadoop-3.3.6文件夹
## 4、配置Hadoop
进入目录：
```shell
cd /usr/local/hadoop-3.3.6/etc/hadoop/
```
### 1.core-site.xml
```xml

<configuration>
<!-- 配置访问端口8020 -->
    <property>
    <name>fs.defaultFS</name>  
      <value>hdfs://master:8020</value>  
      </property>
<!-- 配置缓存目录：计算中的数据块存放位置 -->  
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
</configuration>
```
### 2.hadoop-env.sh  
末尾追加
```xml
export JAVA_HOME=/usr/java/jdk1.8.0_281-amd64
```
### 3. yarn-en.sh
末尾追加
```xml
export JAVA_HOME=/usr/java/jdk1.8.0_281-amd64
```
### 4.hdfs-site.xml
```xml
<configuration>
<!--配置namenode.name数据存放地点-->
<property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///data/hadoop/hdfs/name</value>
</property>
<!--配置namenode.data数据存放地点-->
<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///data/hadoop/hdfs/data</value>
</property>
<!--配置secondary访问地址-->
<property>
     <name>dfs.namenode.secondary.http-address</name>
     <value>master:50090</value>
</property>
<!--配置数据备份数-->
<property>
     <name>dfs.replication</name>
     <value>3</value>
</property>

</configuration>
```
### 5、mapred-site.xml 
```xml
<configuration>
<!--设置执行框架设置为 Hadoop YARN.-->
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
<!-- 设置历史访问地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>master:10020</value>
</property>
<property>
     <name>mapreduce.jobhistory.webapp.address</name>
     <value>master:19888</value>
</property>
<!-- 设置mapreduceAM启动的环境变量-->
<property>
  <name>yarn.app.mapreduce.am.env</name>
  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
  <name>mapreduce.map.env</name>
  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
  <name>mapreduce.reduce.env</name>
  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
</configuration>
```
### 6、yarn-site.xml
```xml
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>master</value>
  </property>    
  <property>
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
  </property>
  <property>
    <name>yarn.nodemanager.local-dirs</name>
    <value>/data/hadoop/yarn/local</value>
  </property>
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
 <!--配置yarnapp启动的环境变量-->
<property>
    <name>yarn.application.classpath</name>
    <value>
/usr/local/hadoop-3.3.6/etc/hadoop:/usr/local/hadoop-3.3.6/share/hadoop/common/lib/*:/usr/local/hadoop-3.3.6/share/hadoop/common/*:/usr/local/hadoop-3.3.6/share/hadoop/hdfs:/usr/local/hadoop-3.3.6/share/hadoop/hdfs/lib/*:/usr/local/hadoop-3.3.6/share/hadoop/hdfs/*:/usr/local/hadoop-3.3.6/share/hadoop/mapreduce/lib/*:/usr/local/hadoop-3.3.6/share/hadoop/mapreduce/*:/usr/local/hadoop-3.3.6/share/hadoop/yarn:/usr/local/hadoop-3.3.6/share/hadoop/yarn/lib/*:/usr/local/hadoop-3.3.6/share/hadoop/yarn/*
    </value>
  </property>
<!--配置yarnvm内存检查机制启动-->
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
### 7.workers
追加
```
master
slave1
slave2
slave3
```
### 8.启动文件配置
修改对应的启动文件：
```
cd /usr/local/hadoop-3.3.6/sbin
```
start-dfs.sh、stop-dfs.sh、start-yarn.sh、stop-yarn.sh
（1）在start-dfs.sh、stop-dfs.sh文件的开头添加如下代码：
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs	
这行可以替换成：HDFS_DATANODE_SECURE_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
（2）在start-yarn.sh、stop-yarn.sh文件的开头添加如下代码：
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
### 9、设置IP映射
编辑各节点/etc/hosts
添加：
192.168.128.140 master master.centos.com
192.168.128.141 slave1 slave1.centos.com
192.168.128.142 slave2 slave2.centos.com
192.168.128.143 slave3 slave3.centos.com
### 10、克隆出4个从节点
完整克隆出4个从节点后

需对4个节点进行重命名和设置网卡操作
一下就以slave1，为例，对其他从节点也进行类似操作即可
登入系统
#### 1、修改主机名字
```shell
hostnamectl set-hostname slave1
```
#### 2、修改网址
```shell
vi /etc/NetworkManager/system-connections/ens160.nmconnection
```
修改网址为：
```shell
原来的值address1=192.168.128.140/24,192.168.128.2
修改后值address1=192.168.128.141/24,192.168.128.2
```
#### 3、重启网卡
```shell
nmcli c reload # 重新加载配置文件 
nmcli c up ens160 # 重启ens33网卡
```

#### 4、验证一下
在master上ping一下各个从节点
如果出现以下图样说明配置成功
![[Pasted image 20240610090628.png]]
### 11、配置ssh无密码登录
#### (1)使用ssh-keygen产生公钥与私钥对。
```shell
[root@master ~]# ssh-keygen -t rsa
```
然后连续按三次enter键
默认ssh存放地点
```shell
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
```
#### (2)将公钥发放到各节点机器
```shell
ssh-copy-id -i /root/.ssh/id_rsa.pub master
```
然后依次输入yes,root用户的密码
```shell
ssh-copy-id -i /root/.ssh/id_rsa.pub slave1
ssh-copy-id -i /root/.ssh/id_rsa.pub slave2
ssh-copy-id -i /root/.ssh/id_rsa.pub slave3
```
#### (3)验证一下
```shell
[root@master ~]# ssh slave1
[root@master ~]# ssh slave2
[root@master ~]# ssh slave3
```
如果能成功进入说明配置成功
注意：从节点进入master是需要密码的
#### (4)配置从节点的ssh
由于后续可能涉及到HA的配置，需要从节点对于其他各节点之间也需要ssh
所以需要在其他机器也进行响应的配置
如遇到
![[Pasted image 20240618102907.png]]
输入yes即可
### 12、配置时间同步服务
Centos9 时间同步要使用chrony命令，ntp命令不能使用了，centos9 一般默认有chrony
如果没有的话
```shell
yum install chrony
```
重新下载即可
#### 1、查看chrony状态
```shell
# 启用chronyd服务
systemctl enable chronyd

# 重启chronyd服务
systemctl restart chronyd

# 查看chronyd服务状态
systemctl status chronyd

# 设置服务开机自动启动，一定要打
systemctl enable chronyd
```
#### 2、修改服务端配置
```shell
vi /etc/chrony.conf
```
在这里配置，作用是设置该网段192.168.128.X的所有机器可访问改节点服务
![[Pasted image 20240610094356.png]]
#### 3、重启服务
```shell
systemctl restart chronyd
```
#### 4、配置从节点服务
```shell
vi /etc/chrony.conf
```
将图示部分改为master即可：结果如下
![[Pasted image 20240610095400.png]]
```shell
# 重启服务
systemctl restart chronyd
# 查看chronyd服务状态
systemctl status chronyd
# 查看同步节点
chronyc sources -v
# 设置服务开机自动启动，一定要打
systemctl enable chronyd
```
查看同步节点应出现如下：
![[Pasted image 20240610095623.png]]
对其他节点页进行服务配置即可
### 13、配置环境变量
在master、3个slave上，都要配置
在/etc/profile添加JAVA_HOME和Hadoop路径
```shell
vi /etc/profile
```
追加
```shell

export HADOOP_HOME=/usr/local/hadoop-3.3.6
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
export PATH=$HADOOP_HOME/bin:$PATH:$JAVA_HOME/bin:$HADOOP_HOME/sbin
```
配置完执行
```shell
source ~/.bashrc
```
使用命令查看配置是否生效
```shell
echo $PATH | tr ':' '\n'
```
出现如下图片即可
![[Pasted image 20240610102427.png]]
分发etc/profile文件至从节点
```shell
[root@master ~]# for SLAVE in slave1 slave2 slave3; do scp /etc/profile $SLAVE:/etc/profile; done
```
在各个从节点执行
```shell
source /etc/profile
```
### 14、格式化节点
在master节点上
```shell
hdfs namenode -format
```
等待格式化完成
```shell
# 验证是否格式化成功
cd $HADOOP_HOME/sbin
./start-all.sh
```
查看节点jps
```shell
check jps
```
出现
![[Pasted image 20240610103347.png]]
即为成功
### 15、访问web界面
访问集群界面http://master:9870
[Namenode information](http://master:9870/dfshealth.html#tab-overview)
![[Pasted image 20240610104721.png]]
访问APP管理器
[All Applications](http://master:8088/cluster)
![[Pasted image 20240610104759.png]]

#### hadoop离开安全模式
```
hadoop dfsadmin -safemode leave
```

## 便捷脚本开发
### 1、配置快速查看JPS脚本
```shell
cd /bin
vi checkjps
```
编写脚本内容如下
```shell
#!/bin/bash
echo -e "\033[0;36m <==jps check in master==> \033[0m"
jps
echo -e "\033[0;36m <==jps check in slave1==> \033[0m"
ssh slave1 "/usr/java/jdk1.8.0_281-amd64/bin/jps;exit"
echo -e "\033[0;36m <==jps check in slave2==> \033[0m"
ssh slave2 "/usr/java/jdk1.8.0_281-amd64/bin/jps;exit"
echo -e "\033[0;36m <==jps check in slave3==> \033[0m"
ssh slave3 "/usr/java/jdk1.8.0_281-amd64/bin/jps;exit"
```
令命令生效
```shell
chmod 777 checkjps
```
验证一下：成功！
![[Pasted image 20240610103858.png]]
### 2、快速启动Hadoop脚本
```shell
cd /bin
vi myhadoop
```
编写如下脚本
```shell
#!/bin/bash
if [ "$1" == "start" ]; then
    cd /usr/local/hadoop-3.3.6/sbin
    ./start-all.sh
./mr-jobhistory-daemon.sh start historyserver
elif [ "$1" == "stop" ]; then
    cd /usr/local/hadoop-3.3.6/sbin
    ./stop-all.sh
./mr-jobhistory-daemon.sh stop historyserver
else
    echo "Invalid argument. Please provide 'start' or 'stop'."
fi

```
令命令生效
```shell
chmod 777 myhadoop
```
启动命令格式
```shell
# 启动hadoop集群
myhadoop start
# 关闭hadoop集群
myhadoop stop
```

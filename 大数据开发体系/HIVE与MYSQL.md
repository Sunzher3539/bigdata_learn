
本次配置使用HIVE4.0.0和MYSQL8.3.0
## 1、前往HIVE官网
下载apache-hive-4.0.0-bin.tar.gz
[Index of /dist/hive/hive-4.0.0 (apache.org)](https://archive.apache.org/dist/hive/hive-4.0.0/)
## 2、HIVE4.0.0安装
#### 1、上传apache-hive-4.0.0-bin.tar.gz至/opt/文件夹下
#### 2、解压缩apache-hive-4.0.0-bin.tar.gz至/usr/local/下

```shell
cd /opt
tar -zxvf apache-hive-4.0.0-bin.tar.gz -C /usr/local/
```
#### 3、重命名
```shell
cd /usr/local
mv apache-hive-4.0.0-bin hive
```
#### 4、编辑环境变量
```shell
vi /etc/profile
```
在里面追加如下内容
```shell
#HIVE_HOME 
export HIVE_HOME=/usr/local/hive 
export PATH=$PATH:$HIVE_HOME/bin
```
令资源生效
```shell
source /etc/profile
```
#### 5、初始化
```shell
cd $HIVE_HOME
bin/schematool -dbType derby -initSchema
```
出现：
![[Pasted image 20240618120001.png]]
证明初始化完成
## 2、MYSQL8.3.0安装
#### 1、下载MYSQL8.3.0
[MySQL :: Download MySQL Community Server (Archived Versions)](https://downloads.mysql.com/archives/community/)
下载mysql-8.3.0-1.el9.x86_64.rpm-bundle.tar
上传mysql-8.3.0-1.el9.x86_64.rpm-bundle.tar至/opt文件夹下
创建文件夹
```shell
mkdir /usr/local/mysql
```
#### 2、解压缩mysql-8.3.0-1.el9.x86_64.rpm-bundle.tar
```shell
cd /opt
tar -xf mysql-8.3.0-1.el9.x86_64.rpm-bundle.tar -C /usr/local/mysql/
```
#### 3、卸载系统自带的mariadb
```shell
sudo rpm -qa | grep mariadb | xargs sudo rpm -e --nodeps
```
![[Pasted image 20240618154624.png]]
代表已经删除了
#### 4、安装mysql
前往mysql安装目录
```shell
cd /usr/local/mysql
```
##### 1、安装mysql-community-common
```shell
rpm -ivh mysql-community-common-8.3.0-1.el9.x86_64.rpm
```
##### 2、安装mysql-community-client-plugins
```shell
rpm -ivh mysql-community-client-plugins-8.3.0-1.el9.x86_64.rpm
```
##### 3、安装mysql-community-libs
```shell
rpm -ivh mysql-community-libs-8.3.0-1.el9.x86_64.rpm
```
##### 4、安装mysql-community-client
```shell
rpm -ivh mysql-community-client-8.3.0-1.el9.x86_64.rpm
```
##### 5、安装mysql-community-icu-data-files
```shell
rpm -ivh mysql-community-icu-data-files-8.3.0-1.el9.x86_64.rpm 
```
#### 5、启动mysql
```shell
systemctl start mysqld #启动mysql服务器 
systemctl status mysqld #查看服务器状态 
systemctl enable mysqld #设置虚拟机开机mysql服务自动启动
```
## 3、Mysql配置
#### 1、查看mysql密码
```sql
sudo cat /var/log/mysqld.log | grep password
```
![[Pasted image 20240618170106.png]]
记住最后的密码串
#### 2、进入mysql进行初始化配置
```shell
mysql -uroot -p'!oee5REFIQq('
```
#### 3、设置密码
先重置密码为一个可以记住的复杂密码
```sql
alter user 'root'@'localhost' identified by 'Sunzher=3539'
```
更改密码策略

```sql
-- 特殊字符检查
set global validate_password.special_char_count = 0;
-- 密码长度最小限制
set global validate_password.length = 4;
-- 密码安全策略
set global validate_password.policy = low;
-- 刷新权限
flush privileges;
```
重置密码
```sql
alter user 'root'@'localhost' identified by '123456'
```
新建一个可供全局访问的用户
```sql
create user 'root'@'%' identified by '123456';
grant all privileges on *.* to 'root'@'%';
flush privileges;
```

## 4、配置hive on mysql
### 1、新建hivemysql库 
```shell
mysql -uroot -p123456 
```
创建Hive元数据库 
```mysql
create database metastore; 
quit;
```
### 2、获取mysql-connect
前往官网[MySQL :: Download MySQL Connector/J (Archived Versions)](https://downloads.mysql.com/archives/c-j/)
获取mysql-connector-j-8.3.0-1.el9.noarch.rpm
上传至/opt文件夹下
```shell
cd /opt
rpm -ivh mysql-connector-j-8.3.0-1.el9.noarch.rpm
```
执行安装命令
```shell
rpm mysql-connector-j-8.3.0-1.el9.noarch.rpm
```
然后可以在/usr/share/java文件夹下找到它
## 5、配置hive-site.xml
```shell
cd $HIVE_HOME/conf/
vi hive-site.xml
```
输入以下内容
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://192.168.128.140:3306/metastore?createDatabaseIfNotExist=true&amp;useSSL=false&amp;allowPublicKeyRetrieval=true&amp;serverTimezone=Asia/Shanghai</value>
    <description>连接驱动</description>  
 </property>
 <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.cj.jdbc.Driver</value>
    <description>JDBC连接驱动</description>
 </property>
 <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
 </property>
 <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>123456</value>
 </property>
<property>
    <name>hive.metastore.warehouse.dir</name>
    <value>/user/hive/warehouse</value>
 </property>
<!-- 指定hiveserver2连接的host -->
<property>
	<name>hive.server2.thrift.bind.host</name>
	<value>master</value>
</property>

<!-- 指定hiveserver2连接的端口号 -->
<property>
	<name>hive.server2.thrift.port</name>
	<value>10000</value>
</property>
<property>
    <name>hive.cli.print.header</name>
    <value>true</value>
    <description>Whether to print the names of the columns in query output.</description>
</property>
<property>
    <name>hive.cli.print.current.db</name>
    <value>true</value>
    <description>Whether to include the current database in the Hive prompt.</description>
</property>
<property>
    <name>hive.metastore.uris</name>
    <value>thrift://master:9083</value>
</property>
</configuration>
```
然后
```shell
mv hive-env.sh.template hive-env.sh
```
## 6、初始化HIVE元数据库
```shell
cd $HIVE_HOME
bin/schematool -dbType mysql -initSchema -verbose
```
![[Pasted image 20240618183541.png]]
出现上图，即为成功
## 7、验证HIVE
```shell
hive
```
![[Pasted image 20240618183634.png]]
```shell
beline> !connect jdbc:mysql://192.168.128.140:3306
Connecting to jdbc:mysql://192.168.128.140:3306
Enter username for jdbc:mysql://192.168.128.140:3306: root
Enter password for jdbc:mysql://192.168.128.140:3306: ******
```
![[Pasted image 20240618183733.png]]
验证完毕，我们的hive就配置完成了
## 8、HIVESERVER2配置
`hivesever2`的模拟用户功能，依赖于Hadoop提供的`proxy user`（代理用户功能），只有Hadoop中的代理用户才能模拟其他用户的身份访问Hadoop集群。因此，需要将`hiveserver2`的启动用户设置为Hadoop的代理用户，配置方式如下：

### 1、配置core.xml
```shell
cd $HADOOP_HOME/etc/hadoop
vi core.xml
```
追加
```xml
<!-- 配置访问hadoop的权限，能够让hive访问到 -->
	<property>
		<name>hadoop.proxyuser.root.hosts</name>
		<value>*</value>
	</property>
	<property>
		<name>hadoop.proxyuser.root.users</name>
		<value>*</value>
	</property>
```
### 2、配置hive-site.xml
```shell
vi $HIVE_HOME/conf/hive-site.xml
```
```xml
<!-- 指定hiveserver2连接的host -->
<property>
	<name>hive.server2.thrift.bind.host</name>
	<value>hadoop001</value>
</property>

<!-- 指定hiveserver2连接的端口号 -->
<property>
	<name>hive.server2.thrift.port</name>
	<value>10000</value>
</property>

```
执行代码：
```shell
$HIVE_HOME/bin/hive --service metastore
# 再开一个
$HIVE_HOME/bin/hive --service hiveserver2
```
### 3、验证连接
```shell
hive
bin/beeline jdbc:hive2://192.168.128.140:10000 -uroot -p'123456'
```
能成功进入即可
至此，完成配置
##### 报错
1、net-tools 被 mysql-community-server-8.3.0-1.el9.x86_64 需要
执行如下代码即可
```shell
yum -y install net-tools
```
2、/usr/bin/perl 被 mysql-community-server-8.3.0-1.el9.x86_64 需要
```shell
yum -y install perl.x86_64
```
3、遇到java-headless >= 1:1.8.0 被 mysql-connector-j-1:8.3.0-1.el9.noarch 需要
```shell
yum -y install java-headless
```
## 启动hiveserve2脚本
进入bin目录
```shell
vi hiveservice
```
```xml
#!/bin/bash
cd /usr/local/hive/bin
nohup hive --service metastore >> /usr/local/hive/log/metastorelog.log 2>&1 &
nohup hive --service hiveserver2 2>&1 &

```
编辑权限
```shell
chmod 777 hiveservice
```
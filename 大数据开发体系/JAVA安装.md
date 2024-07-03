前清提要
默认您已经具备了
linux：CentOS-Stream-9-20240527.0-x86_64-dvd1.iso
windows软件：XSHELL,XFTP
![[Pasted image 20240609163138.png]]
## 1、获取JAVA8 JDK
这里我们选用jdk-8u281-linux-x64.rpm，目前官网上已无该资源信息，可以下载最新的JDK8
[Java Archive Downloads - Java SE 8 (oracle.com)](https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html)

![[Pasted image 20240609163358.png]]
或者选择从其他途径获得jdk-8u281-linux-x64.rpm
## 2、利用XSHELL/XFTP上传文件到master
1.上传jdk文件到/opt文件夹
2.执行 
```shell
rpm -ivh jdk-8u281-linux-x64.rpm
```
这个安装包之后是将JDK安装到：/usr/java/jdk1.8.0_281-amd64/文件加下
 3.查看版本。
 ```shell
 [root@master opt]# java -version
java version "1.8.0_281"
Java(TM) SE Runtime Environment (build 1.8.0_281-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.281-b09, mixed mode)
```
出现该信息我们的java就安装成功了！
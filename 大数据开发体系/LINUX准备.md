## 1.LINUX镜像下载：
###### 1.前往centos官网：[Download (centos.org)](https://www.centos.org/download/)
![[Pasted image 20240609152233.png]]
下载镜像，大小约为9.61G
![[Pasted image 20240609135619.png]]
###### 2.前往清华园网站下载
[Index of /centos-stream/9-stream/BaseOS/x86_64/iso/ | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/centos-stream/9-stream/BaseOS/x86_64/iso/)
下载
![[Pasted image 20240609135947.png]]
## 2.VMware安装
以VMware17pro为例子
#### 1、新建虚拟机
![[Pasted image 20240609143049.png]]
#### 2、为了最大兼容性选择16安装
![[Pasted image 20240609143144.png]]
#### 3、选择稍后安装操作系统
![[Pasted image 20240609143300.png]]
#### 4、选择Red Hat Enterprise Linux 9 64位
![[Pasted image 20240609143311.png]]
#### 5、设置文件路径、并将名称命名为master
![[Pasted image 20240609143419.png]]
#### 6、设置处理器核数
我自己机子比较垃圾、所以都设置成1
11th Gen Intel(R) Core(TM) i5-11400H @ 2.70GHz   2.69 GHz
RTX3050ti 笔记本
![[Pasted image 20240609143543.png]]
#### 7、分配内存
2GB即可
![[Pasted image 20240609143709.png]]
#### 8、使用NAT
![[Pasted image 20240609143751.png]]
#### 9、控制器类型
![[Pasted image 20240609143814.png]]
#### 10、磁盘类型选择NVMe
![[Pasted image 20240609143837.png]]
#### 11、创建新磁盘
![[Pasted image 20240609143857.png]]
#### 12、分配磁盘空间大小
30GB
![[Pasted image 20240609143930.png]]
#### 13、新建虚拟机向导
默认即可
![[Pasted image 20240609143956.png]]
#### 14、配置镜像
![[Pasted image 20240609145306.png]]
选择自己ISO映像文件
![[Pasted image 20240609145333.png]]
#### 15、完成
![[Pasted image 20240609152925.png]]
## 3、镜像安装
#### 1、启动刚刚创建好的虚拟机master
选择第一项 Install CentOS Stream 9
![[Pasted image 20240609153131.png]]
#### 2、等待安装完毕
![[Pasted image 20240609153226.png]]
#### 3、选择中文
![[Pasted image 20240609154839.png]]

#### 4、配置网络环境
![[Pasted image 20240609155023.png]]
#### 5、更改主机名为master
![[Pasted image 20240609155407.png]]
#### 6、配置网络环境
进入网络配置界面
->IPV4设置
->方法-手动
->添加地址：
```
192.168.128.150 
24 
192.168.128.2
```
->编写DNS服务器：4.4.4.4
->勾选需要IPV4地址完成连接
->保存
![[Pasted image 20240609160533.png]]
完成保存后
打开开关观察连接状况
![[Pasted image 20240609160756.png]]
出现上图信息成功
点击左上角完成即可
#### 7、配置虚拟NAT
![[Pasted image 20240609161539.png]]
获得管理员权限
![[Pasted image 20240609161636.png]]
按照下图步骤配置如图相同的参数
![[Pasted image 20240609161824.png]]

NAT页面
![[Pasted image 20240609161912.png]]
完成配置后**应用**并确认即可
#### 8、配置时间
![[Pasted image 20240609161045.png]]
![[Pasted image 20240609162028.png]]
配置完成后点击完成，完成配置
#### 9、存储配置
![[Pasted image 20240609162137.png]]
与图中一致即可
![[Pasted image 20240609162158.png]]
#### 10、编辑root密码
![[Pasted image 20240609162259.png]]
![[Pasted image 20240609162349.png]]
#### 11、软件选择
![[Pasted image 20240609162629.png]]
![[Pasted image 20240609162659.png]]

#### 12、完成配置，重启虚拟机
![[Pasted image 20240609162449.png]]
![[Pasted image 20240609162830.png]]
完成安装！
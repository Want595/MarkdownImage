---
title: 完全分布式高可用集群（Hadoop）
mathjax: true
date: 2024-07-22 14:56:02
tags:
categories: 大数据集群
---

# Hadoop

## 零、资源准备

- VMware workstation 16

- CentOS-Stream-9-latest-x86_64-dvd1.iso

- jdk-8u361-linux-x64.tar.gz

- Hadoop 3.3.6.tar.gz

## 一、前置准备

1. 创建虚拟机

![image-20240721140358041](完全分布式高可用集群（Hadoop）/image-20240721140358041.png)

2. 选择典型安装

![image-20240721140425249](完全分布式高可用集群（Hadoop）/image-20240721140425249.png)

2）安装来源暂时不指定

![image-20240721140440892](完全分布式高可用集群（Hadoop）/image-20240721140440892.png)

3）操作系统选择Linux

![image-20240721140457051](完全分布式高可用集群（Hadoop）/image-20240721140457051.png)

4）设置虚拟机名称和位置

注意：位置可以根据自己电脑的使用情况选择相应空闲磁盘

![image-20240721140530128](完全分布式高可用集群（Hadoop）/image-20240721140530128.png)

5）磁盘容量指定

![image-20240721140542229](完全分布式高可用集群（Hadoop）/image-20240721140542229.png)

6）完成新建

![image-20240721140552492](完全分布式高可用集群（Hadoop）/image-20240721140552492.png)

7）虚拟机设置

注意：配置内存为2G，处理器2个，可根据电脑配置适当增加

![image-20240721140625647](完全分布式高可用集群（Hadoop）/image-20240721140625647.png)


2. 安装CentOS


1）设置CentOS映像文件

ISO映像选择下载的**CentOS-Stream-9-xxxxxxx.iso**

![image-20240721140732879](完全分布式高可用集群（Hadoop）/image-20240721140732879.png)

![image-20240721140702212](完全分布式高可用集群（Hadoop）/image-20240721140702212.png)

2）启动虚拟机

![image-20240721140755596](完全分布式高可用集群（Hadoop）/image-20240721140755596.png)

3）开始安装

选择Install CentOS Stream 9进行安装

![image-20240721140817699](完全分布式高可用集群（Hadoop）/image-20240721140817699.png)

4）语言选择中文（或English）

![image-20240721140912212](完全分布式高可用集群（Hadoop）/image-20240721140912212.png)

5）安装前的配置

![image-20240721141331991](完全分布式高可用集群（Hadoop）/image-20240721141331991.png)


① 安装目的地

默认配置即可

![image-20240721141343343](完全分布式高可用集群（Hadoop）/image-20240721141343343.png)

② 软件选择

选择Server with GUI，并勾选Performance Tools

![image-20240721141402345](完全分布式高可用集群（Hadoop）/image-20240721141402345.png)

③ 时区

区域选择亚洲， 城市选择上海

![image-20240721141503285](完全分布式高可用集群（Hadoop）/image-20240721141503285.png)

④ 网络设置

确保网卡已经打开（左下角可设置主机名，也可以后续设置）

![image-20240721141754308](完全分布式高可用集群（Hadoop）/image-20240721141754308.png)

⑤ 配置root用户密码

注意勾选“允许root用户使用密码进行SSH登录”，作为练习，密码可以设置简单点，比如123456

![image-20240721141617628](完全分布式高可用集群（Hadoop）/image-20240721141617628.png)

6）等待安装完成后重启系统。

![image-20240721141825364](完全分布式高可用集群（Hadoop）/image-20240721141825364.png)

![image-20240721143839541](完全分布式高可用集群（Hadoop）/image-20240721143839541.png)

7）配置虚拟机SSH远程登录

① 启动hadoop1

进入登录界面注册用户

![image-20240721143958759](完全分布式高可用集群（Hadoop）/image-20240721143958759.png)

![image-20240721144035001](完全分布式高可用集群（Hadoop）/image-20240721144035001.png)

打开终端

![image-20240721144247390](完全分布式高可用集群（Hadoop）/image-20240721144247390.png)

![image-20240721144300119](完全分布式高可用集群（Hadoop）/image-20240721144300119.png)

切换到root用户，输入`su root`命令然后输入密码即可

②配置虚拟机SSH远程登录

**第一步，检查SSH服务是否安装和启动**

在虚拟机中，分别执行`rpm -qa | grep ssh`和`ps -ef | grep sshd`命令，查看当前虚拟机是否安装了SSH服务，以及SSH服务是否启动。

![image-20240721144528022](完全分布式高可用集群（Hadoop）/image-20240721144528022.png)

rpm（英文全拼：redhat package manager） 原本是 Red Hat Linux 发行版专门用来管理 Linux 各项套件的程序，由于它遵循 GPL 规则且功能强大方便，因而广受欢迎。逐渐受到其他发行版的采用。RPM 套件管理方式的出现，让 Linux 易于安装，升级，间接提升了 Linux 的适用度。

ps （英文全拼：process status）命令用于显示当前进程的状态，类似于 windows 的任务管理器。

grep (global regular expression) 命令用于查找文件里符合条件的字符串或正则表达式。该命令用于查找内容包含指定的范本样式的文件，如果发现某文件的内容符合所指定的范本样式，预设 grep 指令会把含有范本样式的那一列显示出来。若不指定任何文件名称，或是所给予的文件名为 -，则 grep 指令会从标准输入设备读取数据。

如果没有安装，可以使用以下命令进行安装

`yum install openssh-server openssh-clients`

**第二步，修改SSH服务配置文件**

默认情况下，CentOS Stream 9不允许用户root进行远程登录，在虚拟机hadoop1中执行`vi /etc/ssh/sshd_config`命令编辑配置文件sshd_config。将PermitRootLogin修改为yes

![image-20240721145123672](完全分布式高可用集群（Hadoop）/image-20240721145123672.png)

> 对于小白，这里介绍下vi命令的简单使用方式：使用vi命令打开文件后，输入字母i进入插入模式 => 修改相应的文件内容 => 按Esc键进入命令行模式 => 输入:进入底行模式 => 输入x或者wq保存退出。
>
> 如果文件修改后不想保存，进行底行模式后输入q!进行不保存退出。

**第三步， 重启SSH服务**

`systemctl restart sshd`

![image-20240721145430440](完全分布式高可用集群（Hadoop）/image-20240721145430440.png)

3. 克隆主机

1）关闭hadoop1

在终端使用命令`shutdown -h now`关闭hadoop1或直接关机

![image-20240721145558170](完全分布式高可用集群（Hadoop）/image-20240721145558170.png)

  2）克隆虚拟机

克隆虚拟机hadoop1、hadoop2、hadoop3，以hadoop1为例

![image-20240721145846715](完全分布式高可用集群（Hadoop）/image-20240721145846715.png)

完整克隆的虚拟机是通过复制原虚拟机创建完全独立的新虚拟机，不和原虚拟机共享任何资源，可以脱离原虚拟机独立使用。

链接克隆的虚拟机需要和原虚拟机共享同一个虚拟磁盘文件，不能脱离原虚拟机独立运行。

![image-20240721145940932](完全分布式高可用集群（Hadoop）/image-20240721145940932.png)

克隆hadoop1

![image-20240721150101913](完全分布式高可用集群（Hadoop）/image-20240721150101913.png)

克隆hadoop2（同理）

克隆hadoop3（同理）

4. 网络设置

  网络整体规划如下：

| 虚拟机名 | 主机名  | IP              |
| -------- | ------- | --------------- |
| hadoop1  | hadoop1 | 192.168.121.160 |
| hadoop2  | hadoop2 | 192.168.121.161 |
| hadoop3  | hadoop3 | 192.168.121.162 |

1）配置VMware Workstation网络

在VMware Workstation主界面，依次单击“编辑”→“虚拟网络编辑器…”选项，配置VMware Workstation网络。

![image-20240721150948899](完全分布式高可用集群（Hadoop）/image-20240721150948899.png)

![image-20240721151031381](完全分布式高可用集群（Hadoop）/image-20240721151031381.png)

![image-20240721151059435](完全分布式高可用集群（Hadoop）/image-20240721151059435.png)

2）配置静态IP

以hadoop1主机为例，类似配置hadoop2、 hadoop3（所有命令三台机器都要测试）

编辑配置文件（先切换到root用户）

`vi /etc/NetworkManager/system-connections/ens160.nmconnection`

注意每台机器的address不同，第一台是address1，ip是192.168.121.160，另两台分别是161和162

```
method=manual
address1=192.168.121.160/24,192.168.121.2
dns=114.114.114.114
```

![image-20240721151618354](完全分布式高可用集群（Hadoop）/image-20240721151618354.png)

修改uuid（只需要修改hadoop2、 hadoop3主机）

uuid的作用是使分布式系统中的所有元素都有唯一的标识码。

`sed -i '/uuid=/c\uuid='`uuidgen`'' /etc/NetworkManager/system-connections/ens160.nmconnection`

![image-20240721152046968](完全分布式高可用集群（Hadoop）/image-20240721152046968.png)

重启ens33网卡和重新加载网络配置文件

```
nmcli c reload
nmcli c up ens160
```

查看网络信息

```
ip a
```

![image-20240721152242800](完全分布式高可用集群（Hadoop）/image-20240721152242800.png)

检测网络

```
ping www.baidu.com
```

请ping百度（下图可省略）

![image-20240721152337109](完全分布式高可用集群（Hadoop）/image-20240721152337109.png)


输入ctrl+c退出检测

3）主机名

执行命令后需要重新打开终端，才会更新主机名

配置hadoop2主机名

```
hostnamectl set-hostname hadoop2
```

配置hadoop3主机名

```
hostnamectl set-hostname hadoop3
```

4）配置虚拟机SSH远程登录


配置finalshell，需下载该软件

![image-20240721155152496](完全分布式高可用集群（Hadoop）/image-20240721155152496.png)

![image-20240721155248780](完全分布式高可用集群（Hadoop）/image-20240721155248780.png)

5）修改映射文件

在虚拟机hadoop1主机执行`vi /etc/hosts`命令编辑映射文件hosts，在配置文件中添加如下内容。

```
192.168.121.160 hadoop1
192.168.121.161 hadoop2
192.168.121.162 hadoop3
```

![image-20240721154244276](完全分布式高可用集群（Hadoop）/image-20240721154244276.png)

在虚拟机hadoop1主机执行如下命令，拷贝配置到hadoop2, hadoop3

```
scp /etc/hosts root@hadoop2:/etc/hosts
scp /etc/hosts root@hadoop3:/etc/hosts
```

需要输入hadoop2和hadoop3的管理员密码，即123456

![image-20240721154353364](完全分布式高可用集群（Hadoop）/image-20240721154353364.png)


6. 关闭防火墙

  关闭虚拟机hadoop1、hadoop2和hadoop3的防火墙，分别在3台虚拟机中运行如下命令关闭防火墙并禁止防火墙开启启动

  关闭防火墙`systemctl stop firewalld`

  禁止防火墙开机启动`systemctl disable firewalld`

  ```
systemctl stop firewalld
systemctl disable firewalld
  ```

  ![image-20240721154503838](完全分布式高可用集群（Hadoop）/image-20240721154503838.png)

5. 免密登录
   在集群环境中，主节点需要频繁的访问从节点，以获取从节点的运行状态，主节点每次访问从节点时都需要通过输入密码的方式进行验证，确定密码输入正确后才建立连接，这会对集群运行的连续性造成不良影响，为主节点配置SSH免密登录功能，可以有效避免访问从节点时频繁输入密码。接下来，虚拟机hadoop1作为集群环境的主节点实现SSH免密登录。

  SSH免密登录原理（原理：非对称加密算法：公钥加密（给别人）、私钥解密给自己）

1）生成密钥

在虚拟机hadoop1中执行`ssh-keygen -t rsa`命令，生成密钥。（输入命令后一直按回车键即可）

![image-20240721155442688](完全分布式高可用集群（Hadoop）/image-20240721155442688.png)

查看秘钥文件

在虚拟机hadoop1中执行`ll /root/.ssh`命令查看密钥文件。

![image-20240721155539388](完全分布式高可用集群（Hadoop）/image-20240721155539388.png)

2）复制公钥文件

将虚拟机hadoop1生成的公钥文件复制到集群中相关联的所有虚拟机，实现通过虚拟机hadoop1可以免密登录虚拟机hadoop1、hadoop2和hadoop3。

```
ssh-copy-id hadoop1
ssh-copy-id hadoop2
ssh-copy-id hadoop3
```

![image-20240721155632699](完全分布式高可用集群（Hadoop）/image-20240721155632699.png)

![image-20240721155932799](完全分布式高可用集群（Hadoop）/image-20240721155932799.png)

3）测试免密登录

```
ssh hadoop1
ssh hadoop2
ssh hadoop3
```

![image-20240721155808525](完全分布式高可用集群（Hadoop）/image-20240721155808525.png)


6. 安装jdk

  **约定：软件安装包存放于/software，软件安装至/opt**

1）创建目录

在虚拟机hadoop1中执行`mkdir /software`

![image-20240721160154170](完全分布式高可用集群（Hadoop）/image-20240721160154170.png)

2）上传jdk

利用finalshell将jdk-8u361-linux-x64.tar.gz上传至hadoop1的/software目录（直接拖进去即可）

![image-20240721160331623](完全分布式高可用集群（Hadoop）/image-20240721160331623.png)

3）解压jdk并设置软链接

```
cd /software
tar -xvf jdk-8u361-linux-x64.tar.gz -C /opt
ln -s /opt/jdk1.8.0_361 /opt/jdk
```

![image-20240721161306342](完全分布式高可用集群（Hadoop）/image-20240721161306342.png)

4）配置jdk系统环境变量

在虚拟机hadoop1执行`vi /etc/profile`命令编辑环境变量文件profile，在该文件的底部添加配置jdk系统环境变量的内容。

```
export JAVA_HOME=/opt/jdk
export PATH=$PATH:$JAVA_HOME/bin
```

![image-20240721161425765](完全分布式高可用集群（Hadoop）/image-20240721161425765.png)

记得执行`source /etc/profile`重新加载系统环境变量

5）配置java执行程序的软链接

```
# 删除系统自带的java程序
rm -f /usr/bin/java
# 软链接我们自己安装的java程序
ln -s /opt/jdk/bin/java /usr/bin/java
```

6）验证jdk

```
java -version
```

![image-20240721161723219](完全分布式高可用集群（Hadoop）/image-20240721161723219.png)

7）同步文件

分发jdk安装目录和系统环境变量文件至hadoop2、hadoop3

```
scp -r  /opt/jdk root@hadoop2:/opt
scp  /etc/profile root@hadoop2:/etc
scp -r  /opt/jdk root@hadoop3:/opt
scp  /etc/profile root@hadoop3:/etc
```

## 二、完全分布式部署（Hadoop+MapReduce+Yarn）

基于完全分布式模式部署Hadoop，需要将Hadoop中HDFS和YARN的相关服务运行在不同的计算机中，我们使用已经部署好的3台虚拟机hadoop1、hadoop2和hadoop3。为了避免在使用过程中造成混淆，先规划HDFS和YARN的相关服务所运行的虚拟机。

| 虚拟机名 | 主机名  | IP              | 角色    | 服务                                     |
| -------- | ------- | --------------- | ------- | ---------------------------------------- |
| hadoop1  | hadoop1 | 192.168.121.160 | master  | NameNode、ResourceManager                |
| hadoop2  | hadoop2 | 192.168.121.161 | workers | SecondaryNameNode、DataNode、NodeManager |
| hadoop3  | hadoop3 | 192.168.121.162 | workers | DataNode、NodeManager                    |

1. 安装Hadoop

  利用finalshell将hadoop-3.3.6.tar.gz上传至hadoop1

  ![image-20240721162128929](完全分布式高可用集群（Hadoop）/image-20240721162128929.png)

  1）解压
  以解压方式安装Hadoop，将Hadoop安装到虚拟机hadoop1的/opt目录，并设置软链接

  ```
tar -xvf /software/hadoop-3.3.6.tar.gz -C /opt
ln -s /opt/hadoop-3.3.6 /opt/hadoop
  ```

  ![image-20240721162529355](完全分布式高可用集群（Hadoop）/image-20240721162529355.png)

  2）配置环境变量
  在hadoop1执行`vi /etc/profile`命令配置系统环境变量，在该文件的底部添加如下内容。

  ```
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
  ```

![image-20240721162651011](完全分布式高可用集群（Hadoop）/image-20240721162651011.png)

3）验证

在虚拟机hadoop1的任意目录执行`hadoop version`命令查看当前虚拟机中Hadoop的版本号。（记得先source）

![image-20240721162756797](完全分布式高可用集群（Hadoop）/image-20240721162756797.png)

2. 修改配置文件

| 配置文件        | 功能描述                                                     |
| --------------- | ------------------------------------------------------------ |
| hadoop-env.sh   | 配置Hadoop运行时的环境，确保HDFS能够正常运行NameNode、SecondaryNameNode和DataNode服务 |
| yarn-env.sh     | 配置YARN运行时的环境，确保YARN能够正常运行ResourceManager和NodeManager服务 |
| core-site.sh    | Hadoop核心配置文件                                           |
| hdfs-site.xml   | HDFS核心配置文件                                             |
| mapred-site.xml | MapReduce核心配置文件                                        |
| yarn-site.xml   | YARN核心配置文件                                             |
| workers         | 控制从节点所运行的服务器                                     |

  1）配置Hadoop运行时环境
  在Hadoop安装目录`/opt/hadoop/etc/hadoop/`目录，执行`vi hadoop-env.sh`命令，在`hadoop-env.sh`文件的底部添加如下内容。

  - 指定Hadoop使用的jdk

  - 指定管理NameNode、DataNode等服务的用户为root

  ```
export JAVA_HOME=/opt/jdk
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
  ```

  ![image-20240721163417683](完全分布式高可用集群（Hadoop）/image-20240721163417683.png)

  ![image-20240721163503167](完全分布式高可用集群（Hadoop）/image-20240721163503167.png)

2）配置hadoop

在Hadoop安装目录`/opt/hadoop/etc/hadoop/`目录，执行`vi core-site.xml`命令，在`core-site.xml`文件中添加如下内容。

```
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop1:9000</value>
</property>
<property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/data/hadoop-3.3.6</value>
</property>
<property>
   <name>hadoop.http.staticuser.user</name>
   <value>root</value>
</property>
<property>
    <name>hadoop.proxyuser.root.hosts</name>
    <value>*</value>
</property>
<property>
    <name>hadoop.proxyuser.root.groups</name>
    <value>*</value>
</property>
<property>
    <name>fs.trash.interval</name>
    <value>1440</value>
</property>
```

**注意：上面的配置项要配置到<configuration>标签中，后面的配置项类似**

![image-20240721163837014](完全分布式高可用集群（Hadoop）/image-20240721163837014.png)

> 配置项：
>
> fs.defaultFS：指定HDFS的通信地址
>
> hadoop.tmp.dir：指定Hadoop临时数据的存储目录
>
> hadoop.http.staticuser.user：指定通过Web UI访问HDFS的用户root
>
> hadoop.proxyuser.root.hosts：允许任何服务器的root用户可以向Hadoop提交任务
>
> hadoop.proxyuser.root.groups：允许任何用户组的root用户可以向Hadoop提交任务
>
> fs.trash.interval：指定HDFS中被删除文件的存活时长为1440秒
>
> 更多参数请参考官网：https://hadoop.apache.org/docs/r3.3.6/hadoop-project-dist/hadoop-common/core-default.xml

3）配置HDFS

在Hadoop安装目录`/opt/hadoop/etc/hadoop/`目录，执行`vi hdfs-site.xml`命令，在`hdfs-site.xml`文件中添加如下内容。

```
<property>
    <name>dfs.replication</name>
    <value>2</value>
</property>
<property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop2:9868</value>
</property>
```

![image-20240721164003470](完全分布式高可用集群（Hadoop）/image-20240721164003470.png)

> 配置项：
>
> dfs.replication：指定数据副本个数
>
> dfs.namenode.secondary.http-address：指定SecondaryNameNode服务的通信地址

更多参数请参考官网：https://hadoop.apache.org/docs/r3.3.6/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

4）配置MapReduce

在Hadoop安装目录`/opt/hadoop/etc/hadoop/`目录，执行`vi mapred-site.xml`命令，在`mapred-site.xml`文件中添加如下内容。

```
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
<property>
    <name>mapreduce.job.ubertask.enable</name>
    <value>true</value>
</property>
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop1:10020</value>
</property>
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop1:19888</value>
</property>
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
```

![image-20240721164135151](完全分布式高可用集群（Hadoop）/image-20240721164135151.png)

> 配置项：
>
> mapreduce.framework.name：MapReduce的执行模式，默认是本地模式，另外可以设置成classic(采用MapReduce1.0模式运行) 或 yarn（基于YARN框架运行）.
>
> mapreduce.job.ubertask.enable：是否允许开启uber模式，开启后，小作业会在一个JVM上顺序运行，而不需要额外申请资源
>
> mapreduce.jobhistory.address：指定MapReduce历史服务的通信地址
>
> mapreduce.jobhistory.webapp.address：指定通过Web UI访问MapReduce历史服务的地址
>
> yarn.app.mapreduce.am.env：指定MapReduce任务的运行环境
>
> mapreduce.map.env：指定MapReduce任务中Map阶段的运行环境
>
> mapreduce.reduce.env：指定MapReduce任务中Reduce阶段的运行环境
>
> 更多参数请参考官网：https://hadoop.apache.org/docs/r3.3.6/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml

5）配置YARN

在Hadoop安装目录`/opt/hadoop/etc/hadoop/`目录，执行`vi yarn-site.xml`命令，在`yarn-site.xml`文件中添加如下内容。

```
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop1</value>
</property>
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>
<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>
<property>
    <name>yarn.log.server.url</name>
    <value>http://hadoop1:19888/jobhistory/logs</value>
</property>
<property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
</property>
```

![image-20240721164518066](完全分布式高可用集群（Hadoop）/image-20240721164518066.png)

> 配置项：
>
> yarn.resourcemanager.hostname：指定ResourceManager服务运行的主机
>
> yarn.nodemanager.aux-services：指定NodeManager运行的附属服务
>
> yarn.nodemanager.pmem-check-enabled：指定是否启动检测每个任务使用的物理内存
>
> yarn.nodemanager.vmem-check-enabled：指定是否启动检测每个任务使用的虚拟内存
>
> yarn.log-aggregation-enable：指定是否开启日志聚合功能
>
> yarn.log.server.url：指定日志聚合的服务器
>
> yarn.log-aggregation.retain-seconds：指定日志聚合后日志保存的时间
>
> 更多参数请参考官网：https://hadoop.apache.org/docs/r3.3.6/hadoop-yarn/hadoop-yarn-common/yarn-default.xml

6）配置workers

在虚拟机hadoop1的`/opt/hadoop/etc/hadoop/`目录，执行`vi workers`命令，将`workers`文件默认的内容修改为如下内容。

```
hadoop2
hadoop3
```

![image-20240721164720509](完全分布式高可用集群（Hadoop）/image-20240721164720509.png)

3. 同步文件

  使用scp命令将虚拟机hadoop1的Hadoop安装目录分发至虚拟机hadoop2和hadoop3中存放安装程序的目录。

  ```
scp -r /opt/hadoop root@hadoop2:/opt
scp -r /opt/hadoop root@hadoop3:/opt
scp /etc/profile root@hadoop2:/etc
scp /etc/profile root@hadoop3:/etc
  ```

4. 格式化

在虚拟机hadoop1执行`hdfs namenode -format`命令，对基于完全分布式模式部署的Hadoop进行格式化HDFS文件系统的操作。

  **注意：格式化HDFS文件系统的操作只在初次启动Hadoop集群之前进行。**

5. 启动

在虚拟机hadoop1中执行命令启动Hadoop

  ```
start-dfs.sh
start-yarn.sh
  ```

6. 检测

1）jps查看进程

HDFS和YARN的相关服务运行在JVM进程中，可以执行jps命令查看当前虚拟机中运行的JVM进程。

  ![image-20240721170644728](完全分布式高可用集群（Hadoop）/image-20240721170644728.png)

  2）Web UI

① 在本地计算机的浏览器输入http://192.168.121.160:9870查看HDFS的运行状态。

  ![image-20240721170716782](完全分布式高可用集群（Hadoop）/image-20240721170716782.png)

  ② 在本地计算机的浏览器输入http://192.168.121.160:8088查看YARN的运行状态。

![image-20240721170737956](完全分布式高可用集群（Hadoop）/image-20240721170737956.png)

如果希望在本地计算机上使用 http://hadoop1:9870和http://hadoop1:8088查看Hadoop运行状态, 需要配置本机的hosts文件`C:\Windows\System32\drivers\etc\hosts`, 添加如下内容即可

  ```
192.168.121.160 hadoop1
192.168.121.161 hadoop2
192.168.121.162 hadoop3
  ```

![image-20240721153942607](完全分布式高可用集群（Hadoop）/image-20240721153942607.png)

7. Hadoop启动服务总结

下面就Hadoop的服务启动进行简单的总结:

1）整体启动和关闭

  ```
start-all.sh
stop-all.sh
  ```

2）各个服务组件逐一启动/停止

（1）分别启动/停止HDFS组件

  ```
hdfs --daemon start namenode
hdfs --daemon start datanode
hdfs --daemon start secondarynamenode
hdfs --daemon stop namenode
hdfs --daemon stop datanode
hdfs --daemon stop secondarynamenode
  ```

（2）分别启动/停止YARN组件

  ```
yarn --daemon  start resourcemanager
yarn --daemon  start nodemanager
yarn --daemon  stop resourcemanager
yarn --daemon  stop nodemanager
  ```

3） 各个模块分开启动/停止

（1）整体启动/停止HDFS

  ```
start-dfs.sh
stop-dfs.sh
  ```

（2）整体启动/停止YARN

  ```
start-yarn.sh 
stop-yarn.sh
  ```

8. 常见错误及解决办法

1）出现`command not found`错误

检查`/etc/profile`文件中是否配置了正确的PATH

如果`/etc/profile`设置正确，是否没有执行`source /etc/profile`使环境变量生效

2）所有命令都不能运行

如果你发现不止安装的程序命令，就连原系统的内置命令都使用不了（比如ls、vi、cat等），很明显，你在修改/etc/profile时，将PATH路径设置错了。最常见的是错误就是在设置PATH时，PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin，把$PATH:漏掉了，这就相当于现在的PATH路径只有两个值$HADOOP_HOME/bin和$HADOOP_HOME/sbin。

解决办法：

1）恢复默认的PATH路径：

```
export PATH=/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
```

2）使用vi命令修改`/etc/profile`文件，检查设置PATH的地方是否漏掉了$PATH:

3）不小心多次格式化

多次格式化导致DataNode 与 NameNode namespaceID不一致，导致启动HDFS失败，这里告诉最直接暴力的解决办法：

首先清空`$hadoop.tmp.dir`这个目录，以本文为例：

```
stop-all.sh
#本教程配置的hadoop.tmp.dir目录为/home/xiaobai/opt/hadoop/tmp
rm -fr /opt/data/hadoop-3.3.6
```

然后重新格式化HDFS即可

4）NameNode启动不成功

- NameNode没有格式化

- 环境变量配置错误

- Ip和hostname绑定失败，需要通过ip a查看ip地址，重新配置/etc/hosts文件，设置正确的ip和hostname

- hostname含有特殊符号如.(符号点)，会被误解析

5）万能大法

一切的错误，最好的解决办法是查看日志

Hadoop的默认日志文件目录在`$HADOOP_HOME/logs`

## 三、案例——词频统计

WordCount示例是大数据计算里的”Hello World”, 它的功能是对输入文件的单词进行统计，输出每个单词的出现次数。

1. 准备数据

1）创建文本数据

在hadoop1上使用 vi /opt/data/word.txt命令编辑如下内容：

  ```
hello world
hello hadoop
hello hdfs
hello yarn
  ```

  ![image-20240721171409897](完全分布式高可用集群（Hadoop）/image-20240721171409897.png)

2）创建目录

在HDFS创建`/wordcount/input`目录，用于存放文件`word.txt`。

```
hdfs dfs -mkdir -p /wordcount/input
```

3）在虚拟机hadoop1执行如下命令将文件word.txt上传到HDFS的`/wordcount/input`目录。

```
hdfs dfs -put /opt/data/word.txt /wordcount/input
```

![image-20240721171651809](完全分布式高可用集群（Hadoop）/image-20240721171651809.png)

4）查看文件是否上传成功

通过HDFS的Web UI（http://192.168.121.160:9870）查看文件word.txt是否上传成功。

![image-20240721171631319](完全分布式高可用集群（Hadoop）/image-20240721171631319.png)

2. 运行MapReduce程序

1）查看示例程序

进入虚拟机hadoop1的`/opt/hadoop/share/hadoop/mapreduce`目录，在该目录下执行“ll”命令，查看Hadoop提供的MapReduce程序。

2）执行程序

在MapReduce程序所在的目录执行下列命令，统计word.txt中每个单词出现的次数。

  ```
hadoop jar hadoop-mapreduce-examples-3.3.6.jar  wordcount /wordcount/input /wordcount/output
  ```

- hadoop jar：用于指定运行的MapReduce程序；也可以使用yarn jar运行

- wordcount：表示程序名称；

- wordcount/input：表示文件word.txt所在目录；

- wordcount/output：表示统计结果输出的目录

3）MapReduce程序部分运行效果。

  ![image-20240721171823822](完全分布式高可用集群（Hadoop）/image-20240721171823822.png)

  ![image-20240721171833734](完全分布式高可用集群（Hadoop）/image-20240721171833734.png)

3. 查看程序运行状态

MapReduce程序运行过程中，使用浏览器访问YARN的Web UI（http://192.168.121.160:8088）查看程序的运行状态。

在HDFS的Web UI查看统计结果。

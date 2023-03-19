---
title: Ubuntu下安装Kafka集群
author: Uphie
date: 2023-03-19 15:13:00 +0800
categories: [技术]
tags: [linux,kafka,安装]
math: true
toc: true
---

笔者有3台虚拟机机器，分别为 master、node1、node2，在这三台机器上部署 Kafka 集群。

# 配置 Java 环境

由于 Kafka 依赖 Java8+ 环境所以需要先配置好 Java 环境。

我们使用 openjdk
```console
root@master:/home/uphie# jdk
Command 'jdk' not found, did you mean:
  command 'juk' from deb juk (4:21.12.3-0ubuntu1)
  command 'jdb' from deb openjdk-11-jdk-headless (11.0.18+10-0ubuntu1~22.04)
  command 'jdb' from deb openjdk-17-jdk-headless (17.0.6+10-0ubuntu1~22.04)
  command 'jdb' from deb openjdk-18-jdk-headless (18.0.2+9-2~22.04)
  command 'jdb' from deb openjdk-19-jdk-headless (19.0.2+7-0ubuntu3~22.04)
  command 'jdb' from deb openjdk-8-jdk-headless (8u362-ga-0ubuntu1~22.04)
  command 'jd' from deb jdim (0.7.0-1)
Try: apt install <deb name>
```

在 master 上选用 `openjdk-19-jdk-headless` 安装：
```console
root@master:/home/uphie# apt install -y openjdk-19-jdk-headless
```

配置环境变量：
```console
root@master:/home/uphie# echo "JAVA_HOME=$(readlink -f /usr/bin/java | sed "``s:bin/java::``")" | tee -a /etc/profile
JAVA_HOME=/usr/lib/jvm/java-19-openjdk-amd64/
```

前面修改了 文件，使之生效：
```console
root@master:/home/uphie# source /etc/profile
```

检查下环境变量：
```console
root@master:/home/uphie# echo $JAVA_HOME
/usr/lib/jvm/java-19-openjdk-amd64/
```

按同样方法配置节点机器 node1 和 node2。

# 下载 Kafka

从官网 [https://kafka.apache.org/downloads](https://kafka.apache.org/downloads) 选择 Kafka 版本并下载，笔者选择的是编译过的 3.4.0。

```console
root@master:/home/uphie# wget https://downloads.apache.org/kafka/3.4.0/kafka_2.13-3.4.0.tgz
```

将下载的安装包复制到另外两个节点：
```
root@master:/home/uphie# scp kafka_2.13-3.4.0.tgz uphie@node1:/home/uphie/
uphie@node1's password:
kafka_2.13-3.4.0.tgz                                                                                                                              100%  101MB  72.7MB/s   00:01
root@master:/home/uphie#
root@master:/home/uphie# scp kafka_2.13-3.4.0.tgz uphie@node2:/home/uphie/
uphie@node2's password:
kafka_2.13-3.4.0.tgz                                                                                                                              100%  101MB  68.8MB/s   00:01
```

解压:
```console
root@master:/home/uphie# tar -zxf kafka_2.13-3.4.0.tgz
```

移到 /opt 目录下：
```
root@master:/home/uphie# mv /home/uphie/kafka_2.13-3.4.0 /opt/
```

给 kafka 目录添加软链接，方便以后升级（升级时只需修改软链接的指向即可）：
```console
root@master:/home/uphie# ln -s /opt/kafka_2.13-3.4.0 /opt/kafka
```

# 配置和启动服务

3.4.0 版本可以选择 zookeeper 或 kraft 启动集群，笔者选择较新的 kraft 方式。


修改 kraft 配置文件：
```cosole
root@master:/home/uphie# vim /opt/kafka/config/kraft/server.properties
```

修改以下配置，读者也可根据自己需要修改其他配置：
```
############################# Server Basics #############################

# The role of this server. Setting this puts us in KRaft mode
process.roles=broker,controller

# The node id associated with this instance's roles
# 节点 ID，每个节点的 ID 需不同
node.id=100

# The connect string for the controller quorum
# 节点选举用的连接方式
controller.quorum.voters=100@master:9093,101@node1:9093,102@node2:9093
```

生成一个 cluster ID：
```
root@master:/home/uphie# /opt/kafka/bin/kafka-storage.sh random-uuid
blibId1XQpSU_RS4_FEkow
```

将 cluster ID 更新到 kraft 配置文件中：
```
root@master:/home/uphie# /opt/kafka/bin/kafka-storage.sh format -t blibId1XQpSU_RS4_FEkow -c /opt/kafka/config/kraft/server.properties
Formatting /tmp/kraft-combined-logs with metadata.version 3.4-IV0.
```

添加一个 Kafka 服务单元，方便管理：
```console
root@master:/home/uphie# vim /etc/systemd/system/kafka.service
```

内容如下：
```
[Unit]
Description=Apache Kafka Server
Documentation=http://kafka.apache.org/documentation.html
Requires=zookeeper.service
 
[Service]
Type=simple
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/kraft/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal
 
[Install]
WantedBy=multi-user.target
```

启动验证下：
```console
root@master:/home/uphie# systemctl start kafka
root@master:/home/uphie# systemctl status kafka
● kafka.service - Apache Kafka Server
     Loaded: loaded (/etc/systemd/system/kafka.service; disabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-03-19 09:47:43 CST; 7s ago
       Docs: http://kafka.apache.org/documentation.html
   Main PID: 157894 (java)
      Tasks: 71 (limit: 2225)
     Memory: 316.2M
        CPU: 6.603s
     CGroup: /system.slice/kafka.service
             └─157894 java -Xmx1G -Xms1G -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:Initiatin>

Mar 19 09:47:48 master kafka-server-start.sh[157894]: [2023-03-19 09:47:48,537] INFO [Transaction M>
Mar 19 09:47:48 master kafka-server-start.sh[157894]: [2023-03-19 09:47:48,600] INFO [ExpirationRea>
Mar 19 09:47:48 master kafka-server-start.sh[157894]: [2023-03-19 09:47:48,765] INFO [/config/chang>
Mar 19 09:47:48 master kafka-server-start.sh[157894]: [2023-03-19 09:47:48,784] INFO [SocketServer >
Mar 19 09:47:48 master kafka-server-start.sh[157894]: [2023-03-19 09:47:48,806] INFO Kafka version:>
Mar 19 09:47:48 master kafka-server-start.sh[157894]: [2023-03-19 09:47:48,807] INFO Kafka commitId>
Mar 19 09:47:48 master kafka-server-start.sh[157894]: [2023-03-19 09:47:48,807] INFO Kafka startTim>
Mar 19 09:47:48 master kafka-server-start.sh[157894]: [2023-03-19 09:47:48,809] INFO [KafkaServer i>
Mar 19 09:47:48 master kafka-server-start.sh[157894]: [2023-03-19 09:47:48,934] INFO [BrokerToContr>
Mar 19 09:47:48 master kafka-server-start.sh[157894]: [2023-03-19 09:47:48,984] INFO [BrokerToContr>
```

按同样方法操作其他节点，笔者 master 节点的 node id为100，node1节点的为 101，node2节点的为102。

# 验证

在 master 节点创建一个测试 topic：
```console
root@master:/home/uphie# /opt/kafka/bin/kafka-topics.sh --create --replication-factor 1 --partitions 1 --topic testtopic --bootstrap-server localhost:9092
Created topic testtopic.
```

在 node1节点查看测试 topic：

```console
root@node1:/home/uphie# /opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
testtopic
```

在 node2 节点查看测试 topic：
```console
root@node2:/home/uphie# /opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
testtopic
```

自此 Kafka 集群安装部署完成。

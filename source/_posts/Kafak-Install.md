---
title: Kafak 安装
index_img: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
banner_img: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
cover: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
date: 2021-07-07 10:37:31
tags: [kafka, centos]
categories: [kafka]
toc: true
permalink: /kafka/install/
description: kafka 安装
comment: 'valine'
---

# 在 CentOS 7 中安装 kafka 详细步骤

## 一、 安装 JDK

    如jdk未安装，请参考以下连接：
    jdk安装:http://tencent.yundashi168.com/814.html

ps: [jdk-8u221-linux-x64](https://pan.baidu.com/s/1JRp5amJzJ8JD4DTIXAr_-A)
提取码：ywuc

## 二、 下载 kafka 和 zookeeper

### - 下载方式：

1. [kafka 官网](http://kafka.apache.org/downloads)

2. 百度云网盘下载

   [kafka_2.12-1.1.0](https://pan.baidu.com/s/1tsEPnnVwAHxHwLS7mUtZFQ)
   提取码：53j4

   [zookeeper-3.4.12](https://pan.baidu.com/s/1VJLMIMaQjx5X0cT5Snsq8g)
   提取码：244b

## 三、安装

1. 解压安装 kafka（我的安装目录/soft/kafka）

```
#解压
tar -zvxf kafka_2.12-1.1.0.tgz
#将解压后的文件夹移动到/soft 下
mv kafka_2.12-1.1.0.tgz /soft
#修改文件夹名称
mv kafka_2.12-1.1.0.tgz kafka
```

2. 配置 kafka

```
#进入配置文件夹
cd /soft/kakfa/config

#修改配置文件
vi server.properties
```

```xml
broker.id=0
port=9092                                                #端口号
host.name=172.30.200.98                      #服务器IP地址，修改为自己的服务器IP
log.dirs=/mnt/soft/kafka_data/log/kafka   #日志存放路径，上面创建的目录
zookeeper.connect=localhost:2181         #zookeeper地址和端口，单机配置部署，localhost:2181
listeners=PLAINTEXT://ip:9092

注：ip指的是本机ip地址
```

3、解压安装 zookeeper(我的安装目录/soft/zookeeper)

```
#解压
tar -zvxf zookeeper-3.4.12.tar.gz
#将解压后的文件夹移动到/soft 下
mv zookeeper-3.4.12 /soft
#修改文件夹名称
mv zookeeper-3.4.12 zookeeper
```

4、配置 zookeeper

```
cd /soft/kafka/config

vi zookeeper.properties
```

```
#添加一条配置
dataDir=/soft/kafka/kafka_data  #zookeeper数据目录
dataLogDir=/softkafka//kafka_data #zookeeper日志目录
```

## 四、验证

1.  使用安装包中的脚本启动单节点 Zookeeper 实例：

```
cd  /soft/kafka

bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```

2. 使用 kafka-server-start.sh 启动 kafka 服务：

```
bin/kafka-server-start.sh config/server.properties
```

3. 使用 kafka-topics.sh 创建但分区单副本的 topic test

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

4. 使用 kafka-console-producer.sh 发送消息

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

## 五、配置开机自启服务

1. 在 /lib/systemd/system/ 目录下创建 zookeeper 服务的配置文件

```
cd /lib/systemd/system/

vi zookeeper.service
```

添加以下内容

```
[Unit]
Description=Zookeeper service
After=network.target

[Service]
Type=simple
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/java/jdk1.8.0_131/bin"
User=root
Group=root
ExecStart=/soft/kafka/bin/zookeeper-server-start.sh /soft/kafka/config/zookeeper.properties
ExecStop=/soft/kafka/bin/zookeeper-server-stop.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

ps:

/usr/java/jdk1.8.0_131/bin 这里换成自己的 jdk 安装路径(jdk 路径查看命令：$JAVA_HOME)

/soft/kafka 这是 kafka 安装路径

2. 在 /lib/systemd/system/ 目录下创建和 kafka 服务的配置文件

```
[Unit]
Description=Apache Kafka server (broker)
After=network.target  zookeeper.service

[Service]
Type=simple
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/java/jdk1.8.0_131/bin"
User=root
Group=root
ExecStart=/soft/kafka/bin/kafka-server-start.sh /soft/kafka/config/server.properties
ExecStop=/soft/kafka/bin/kafka-server-stop.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

ps:

/usr/java/jdk1.8.0_131/bin 这里换成自己的 jdk 安装路径(jdk 路径查看命令：$JAVA_HOME)

/soft/kafka 这是 kafka 安装路径

3. 刷新配置。

```
systemctl daemon-reload
```

4. zookeeper、kafka 服务加入开机自启。

```
systemctl enable zookeeper

systemctl enable kafka
```

5. 使用 systemctl 启动/关闭/重启 zookeeper、kafka 服务 systemctl start/stop/restart zookeeper/kafka。

注：启动 kafka 前必须先启动 zookeeper 。

```
systemctl start zookeeper

 systemctl start kafka
```

6.  查看状态。

```
systemctl status zookeeper
```

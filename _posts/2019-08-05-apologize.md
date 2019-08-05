---
layout:     post
title:      apologize
subtitle:   ergou、fool、Spanking man、to get someone annoyed 
date:       2019-08-05
author:     skylinelin
header-img: img/machine.jpg
catalog: true
tags:
    - taiqiren
    - huogai
    - geipipidedaoqian
    - A man who was immediately abandoned
---

> Why should you apologize in English? The reason is to let the baby look at the letter of apology, by the way, learn English, but also for the purpose of postgraduate consideration.

## 1、Cause of anger

Today, the weather was very cool at noon. The two treasures were on the way back from the express delivery. At this time, the two treasures chatted with the two dogs. On the way, the two treasures talked to the two dogs many times, but the two dogs did not reply, and the two treasures asked why the two dogs did not council Erbao, the two dogs said that Taobao did not ignore the two treasures. At this time, the two dogs did not find their fault, but instead made an excuse to cover up the daze, and the two treasures were furious~

Erbao said, fuck your mother, the two dogs are very uncomfortable, obviously it is wrong, I don't know what to say, then the two treasures are super angry, smashed the two dogs for a while, the two dogs even dare not speak.



## 2、Become serious

God! Erbao has become extremely angry at this time, the two dogs are shameless, and they continue to make six calls to Erbao. The two dogs make a phone call, the two treasures become more angry, and the two dogs make six calls. Treasure is six times angry.

This time, I have repeatedly made Erbao angry. These two treasures are not angry. The two treasures have been holding back. Today, I provoked two treasures. The treasure chest of Erbao was blown up, two treasures were dissatisfied, and the anger was uncontrollable.

## 4、两种持久化的配置

**RDB持久化配置**

Redis会将数据集的快照dump到dump.rdb文件中。此外，我们也可以通过配置文件来修改Redis服务器dump快照的频率，在打开6379.conf文件之后，我们搜索save，可以看到下面的配置信息：

save 900 1              #在900秒(15分钟)之后，如果至少有1个key发生变化，则dump内存快照。

save 300 10            #在300秒(5分钟)之后，如果至少有10个key发生变化，则dump内存快照。

save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，则dump内存快照。

**AOF持久化配置**

在Redis的配置文件中存在三种同步方式，它们分别是：

appendfsync always     #每次有数据修改发生时都会写入AOF文件。

appendfsync everysec  #每秒钟同步一次，该策略为AOF的缺省策略。

appendfsync no          #从不同步。高效但是数据不会被持久化。



## 3、Redis五大数据类型

|         类型         |                          简介                          |                             特性                             |                             场景                             |
| :------------------: | :----------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|    String(字符串)    |                       二进制安全                       | 可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M |                             ---                              |
|      Hash(字典)      |            键值对集合,即编程语言中的Map类型            | 适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去) |                   存储、读取、修改用户属性                   |
|      List(列表)      |                     链表(双向链表)                     |               增删快,提供了操作某一段元素的API               |     1,最新消息排行等功能(比如朋友圈的时间线) 2,消息队列      |
|      Set(集合)       |                 哈希表实现,元素不重复                  | 1、添加、删除,查找的复杂度都是O(1) 2、为集合提供了求交集、并集、差集等操作 | 1、共同好友 2、利用唯一性,统计访问网站的所有独立ip 3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐 |
| Sorted Set(有序集合) | 将Set中的元素增加一个权重参数score,元素按score有序排列 |               数据插入集合时,已经进行天然排序                |                           1、排行                            |



## 4、Redis集群搭建

**介绍安装环境与版本**

- 用两台虚拟机模拟6个节点，一台机器3个节点，创建出3 master、3 salve 环境。

- redis 采用 redis-3.2.4 版本。

- 两台虚拟机都是 CentOS ，一台 CentOS6.5 （IP:192.168.31.245），一台 CentOS7（IP:192.168.31.210） 。

**安装过程**

**1、下载并解压**

```c
cd /root/software
wget http://download.redis.io/releases/redis-3.2.4.tar.gz
tar -zxvf redis-3.2.4.tar.gz
```

**2、编译安装**

```c
cd redis-3.2.4
make && make install
```

**3、将 redis-trib.rb 复制到 /usr/local/bin 目录下**

```c
cd src
cp redis-trib.rb /usr/local/bin/
```

**4、创建 Redis 节点，首先在 192.168.31.245 机器上 /root/software/redis-3.2.4 目录下创建 redis_cluster 目录；**

```c
mkdir redis_cluster　
```

在 redis_cluster 目录下，创建名为7000、7001、7002的目录，并将 redis.conf 拷贝到这三个目录中

```c
mkdir 7000 7001 7002
cp redis.conf redis_cluster/7000
cp redis.conf redis_cluster/7001
cp redis.conf redis_cluster/7002　
```

分别修改这三个配置文件，修改如下内容

```c
port  7000                                        //端口7000,7002,7003        
bind 本机ip                                       //默认ip为127.0.0.1 需要改为其他节点机器可访问的ip 否则创建集群时无法访问对应的端口，无法创建集群
daemonize    yes                               //redis后台运行
pidfile  /var/run/redis_7000.pid          //pidfile文件对应7000,7001,7002
cluster-enabled  yes                           //开启集群  把注释#去掉
cluster-config-file  nodes_7000.conf   //集群的配置  配置文件首次启动自动生成 7000,7001,7002
cluster-node-timeout  15000                //请求超时  默认15秒，可自行设置
appendonly  yes                           //aof日志开启  有需要就开启，它会每次写操作都记录一条日志
```

接着在另外一台机器上（192.168.31.210），的操作重复以上三步，只是把目录改为7003、7004、7005，对应的配置文件也按照这个规则修改即可

**5、启动各个节点**

```c
第一台机器上执行
redis-server redis_cluster/7000/redis.conf
redis-server redis_cluster/7001/redis.conf
redis-server redis_cluster/7002/redis.conf
 
另外一台机器上执行
redis-server redis_cluster/7003/redis.conf
redis-server redis_cluster/7004/redis.conf
redis-server redis_cluster/7005/redis.conf
```

**6、 检查 redis 启动情况**

```c
##一台机器
ps -ef | grep redis
root      61020      1  0 02:14 ?        00:00:01 redis-server 127.0.0.1:7000 [cluster]    
root      61024      1  0 02:14 ?        00:00:01 redis-server 127.0.0.1:7001 [cluster]    
root      61029      1  0 02:14 ?        00:00:01 redis-server 127.0.0.1:7002 [cluster]    
 
netstat -tnlp | grep redis
tcp        0      0 127.0.0.1:17000             0.0.0.0:*                   LISTEN      61020/redis-server 
tcp        0      0 127.0.0.1:17001             0.0.0.0:*                   LISTEN      61024/redis-server 
tcp        0      0 127.0.0.1:17002             0.0.0.0:*                   LISTEN      61029/redis-server 
tcp        0      0 127.0.0.1:7000              0.0.0.0:*                   LISTEN      61020/redis-server 
tcp        0      0 127.0.0.1:7001              0.0.0.0:*                   LISTEN      61024/redis-server 
tcp        0      0 127.0.0.1:7002              0.0.0.0:*                   LISTEN      61029/redis-server
2
4
6
8
10
12
    
##另外一台机器
ps -ef | grep redis
root       9957      1  0 02:32 ?        00:00:01 redis-server 127.0.0.1:7003 [cluster]
root       9964      1  0 02:32 ?        00:00:01 redis-server 127.0.0.1:7004 [cluster]
root       9971      1  0 02:32 ?        00:00:01 redis-server 127.0.0.1:7005 [cluster]
root      10065   4744  0 02:38 pts/0    00:00:00 grep --color=auto redis
netstat -tlnp | grep redis
tcp        0      0 127.0.0.1:17003         0.0.0.0:*               LISTEN      9957/redis-server 1
tcp        0      0 127.0.0.1:17004         0.0.0.0:*               LISTEN      9964/redis-server 1
tcp        0      0 127.0.0.1:17005         0.0.0.0:*               LISTEN      9971/redis-server 1
tcp        0      0 127.0.0.1:7003          0.0.0.0:*               LISTEN      9957/redis-server 1
tcp        0      0 127.0.0.1:7004          0.0.0.0:*               LISTEN      9964/redis-server 1
tcp        0      0 127.0.0.1:7005          0.0.0.0:*               LISTEN      9971/redis-server 1
```

**7、创建集群**

Redis 官方提供了 redis-trib.rb 这个工具，就在解压目录的 src 目录中，第三步中已将它复制到 /usr/local/bin 目录中，可以直接在命令行中使用了。使用下面这个命令即可完成安装。

```c
redis-trib.rb  create  --replicas  1  192.168.31.245:7000 192.168.31.245:7001  192.168.31.245:7002 192.168.31.210:7003  192.168.31.210:7004  192.168.31.210:7005
```

其中，前三个 ip:port 为第一台机器的节点，剩下三个为第二台机器。

等等，出错了。这个工具是用 ruby 实现的，所以需要安装 ruby。安装命令如下：

```c
yum -y ``install` `ruby ruby-devel rubygems rpm-build
gem ``install` `redis
```

> [转载网址](https://www.cnblogs.com/wuxl360/p/5920330.html)


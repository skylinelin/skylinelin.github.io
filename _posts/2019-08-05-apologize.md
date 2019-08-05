---
layout:     post
title:      apologize
subtitle:   ergou、fool、Spanking man、to get someone annoyed 
date:       2019-08-05
author:     skylinelin
header-img: img/machine.jpg
catalog: true
tags:
    - Redis集群搭建
    - Redis
    - Redis持久化
    - 持久化差异
---

> 有必要了解一个非关系型数据库Redis

## 1、Redis持久化

Redis的所有数据都是保存在内存中，然后不定期的通过异步方式保存到磁盘上(这称为“半持久化模式”)；也可以把每一次数据变化都写入到一个append only file(aof)里面(这称为“全持久化模式”)。 

RDB持久化（记录数据）

AOF持久化（记录操作）



## 2、两种持久化对比

**RDB存在哪些优势呢？**

1). 一旦采用该方式，那么你的整个Redis数据库将只包含一个文件，这对于文件备份而言是非常完美的。比如，你可能打算每个小时归档一次最近24小时的数据，同时还要每天归档一次最近30天的数据。通过这样的备份策略，一旦系统出现灾难性故障，我们可以非常容易的进行恢复。

2). 对于灾难恢复而言，RDB是非常不错的选择。因为我们可以非常轻松的将一个单独的文件压缩后再转移到其它存储介质上。

3). 性能最大化。对于Redis的服务进程而言，在开始持久化时，它唯一需要做的只是fork出子进程，之后再由子进程完成这些持久化的工作，这样就可以极大的避免服务进程执行IO操作了。

4). 相比于AOF机制，如果数据集很大，RDB的启动效率会更高。

**RDB又存在哪些劣势呢？**

1). 如果你想保证数据的高可用性，即最大限度的避免数据丢失，那么RDB将不是一个很好的选择。因为系统一旦在定时持久化之前出现宕机现象，此前没有来得及写入磁盘的数据都将丢失。

2). 由于RDB是通过fork子进程来协助完成数据持久化工作的，因此，如果当数据集较大时，可能会导致整个服务器停止服务几百毫秒，甚至是1秒钟。

**AOF的优势有哪些呢？**

1). 该机制可以带来更高的数据安全性，即数据持久性。Redis中提供了3中同步策略，即每秒同步、每修改同步和不同步。事实上，每秒同步也是异步完成的，其效率也是非常高的，所差的是一旦系统出现宕机现象，那么这一秒钟之内修改的数据将会丢失。而每修改同步，我们可以将其视为同步持久化，即每次发生的数据变化都会被立即记录到磁盘中。可以预见，这种方式在效率上是最低的。至于无同步，无需多言，我想大家都能正确的理解它。

2). 由于该机制对日志文件的写入操作采用的是append模式，因此在写入过程中即使出现宕机现象，也不会破坏日志文件中已经存在的内容。然而如果我们本次操作只是写入了一半数据就出现了系统崩溃问题，不用担心，在Redis下一次启动之前，我们可以通过redis-check-aof工具来帮助我们解决数据一致性的问题。

3). 如果日志过大，Redis可以自动启用rewrite机制。即Redis以append模式不断的将修改数据写入到老的磁盘文件中，同时Redis还会创建一个新的文件用于记录此期间有哪些修改命令被执行。因此在进行rewrite切换时可以更好的保证数据安全性。

4). AOF包含一个格式清晰、易于理解的日志文件用于记录所有的修改操作。事实上，我们也可以通过该文件完成数据的重建。

**AOF的劣势有哪些呢？**

1). 对于相同数量的数据集而言，AOF文件通常要大于RDB文件。RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

2). 根据同步策略的不同，AOF在运行效率上往往会慢于RDB。总之，每秒同步策略的效率是比较高的，同步禁用策略的效率和RDB一样高效。

二者选择的标准，就是看系统是愿意牺牲一些性能，换取更高的缓存一致性（aof），还是愿意写操作频繁的时候，不启用备份来换取更高的性能，待手动运行save的时候，再做备份（rdb）。rdb这个就更有些 eventually consistent的意思了。

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


---
layout:     post
title:      Zookeeper简述
subtitle:   Zookeeper简述、工作原理
date:       2017-03-03
author:     skylinelin
header-img: img/zookeeper.jpg
catalog: true
tags:
    - Zookeeper
    - 选举
    - Zookeeper配置
---


> zookeeper的作用是什么？选举机制、工作原理及同步流程。

## 1. Zookeeper概述

### 1.1 Zookeeper设计目的

配置多个实例共同构成一个集群对外提供服务以达到水平扩展的目的，每个服务器上的数据是相同的，每一个服务器均可以对外提供读和写的服务，这点和redis是相同的，即对客户端来讲每个服务器都是平等的。

![zookeeper01](/resource_img/zookeeper01.png)

### 1.2 Zookeeper特性
1. **最终一致性：** client不论连接到哪个Server，展示给它都是同一个视图，这是zookeeper最重要的性能。 
2. **可靠性：** 具有简单、健壮、良好的性能，如果消息被到一台服务器接受，那么它将被所有的服务器接受。 
3. **实时性：** Zookeeper保证客户端将在一个时间间隔范围内获得服务器的更新信息，或者服务器失效的信息。但由于网络延时等原因，Zookeeper不能保证两个客户端能同时得到刚更新的数据，如果需要最新数据，应该在读数据之前调用sync()接口。 
4. **等待无关（wait-free）**： 慢的或者失效的client不得干预快速的client的请求，使得每个client都能有效的等待。 
5. **原子性：** 更新只能成功或者失败，没有中间状态。 
6. **顺序性：** 包括全局有序和偏序两种：全局有序是指如果在一台服务器上消息a在消息b前发布，则在所有Server上消息a都将在消息b前被发布；偏序是指如果一个消息b在消息a后被同一个发送者发布，a必将排在b前面。 

## 2. Zookeeper中的基本概念

### 2.1 选举机制
zookeeper提供了三种方式：
1. LeaderElection
2. AuthFastLeaderElection
3. FastLeaderElection


### 2.2 Zookeeper中基本角色
 - 领导者（leader），负责进行投票的发起和决议，更新系统状态;
 - 学习者（learner），包括跟随者（follower）和观察者（observer）;
   - follower用于接受客户端请求并想客户端返回结果，在选主过程中参与投票
   - Observer可以接受客户端连接，将写请求转发给leader，但observer不参加投票过程，只同步leader的状态，observer的目的是为了扩展系统，提高读取速度
 - 客户端（client），请求发起方;
 

![zookeeper02](/resource_img/zookeeper02.png)

![zookeeper03](/resource_img/zookeeper03.png)

### 2.3 Zookeeper server属性

 - **服务器ID：** 比如有三台服务器，编号分别是1,2,3。编号越大在选择算法中的权重越大。
 - **数据ID：** 服务器中存放的最大数据ID.值越大说明数据越新，在选举算法中数据越新权重越大。

### 2.4 Zookeeper逻辑时钟

或者叫投票的次数，同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加，然后与接收到的其它服务器返回的投票信息中的数值相比，根据不同的值做出不同的判断。

### 2.5 Zookeeper server选举状态

 - **LOOKING：** 竞选状态。
 - **FOLLOWING：** 随从状态，同步leader状态，参与投票。
 - **OBSERVING：** 观察状态,同步leader状态，不参与投票。
 - **LEADING：** 领导者状态。

## 	3. Zookeeper工作原理

Zookeeper 的核心是原子广播，这个机制保证了各个Server之间的同步。实现这个机制的协议叫做Zab协议。Zab协议有两种模式，它们分别是恢复模式（选主）和广播模式（同步）。当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数Server完成了和leader的状态同步以后，恢复模式就结束了。状态同步保证了leader和Server具有相同的系统状态。

为了保证事务的顺序一致性，zookeeper采用了递增的事务id号（zxid）来标识事务。所有的提议（proposal）都在被提出的时候加上了zxid。实现中zxid是一个64位的数字，它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个新的epoch，标识当前属于那个leader的统治时期。低32位用于递增计数。


> 结尾
---
layout:     post
title:      Zookeeper选举
subtitle:   Zookeeper选举、同步流程
date:       2017-03-10
author:     skylinelin
header-img: img/zookeeper.jpg
catalog: true
tags:
    - Zookeeper
    - 选举
    - Zookeeper配置
    - Zookeeper同步流程
---

> Zookeeper选举、同步流程、深刻理解Zookeeper

## 1. Zookeeper选举过程

### 1.1	集群选举类型：
Zookeeper选举主要是针对leader而言。下述主要分析leader的选择机制，以默认的算法FastLeaderElection为例。

**选举非为两种：**
 - 全新集群选举
 - 非全新集群选举。

![zookeeper04](/resource_img/zookeeper/zookeeper04.png)

### 1.2 全新集群选举：

假设现在有5台服务器均没有数据，它们的编号分别是1，2，3，4，5，按编号依次启动。过程如下：

1. 服务器 1 启动，给自己投票，然后发投票信息给其他服务器。由于其他服务器没有启动，所以它收不到反馈信息，但是由于投票还没有到达半数（服务器 1 怎么知道一共有多少台服务器参与选举呢， 那是因为在zk配置文件中配置了集群信息，所有配置了3888端口的服务器均会参与投票，假设这5台都参与投票，则超过半数应为至少3台服务器参与投票。），所以服务器 1 的状态一直处于 LOOKING。
2. 服务器 2 启动， 给自己投票，然后与其他服务投票信息交换结果，由于服务器 2 的编号大于服务器 1，所以服务器 2 胜出，但是由于投票仍未到达半数，所以服务器 2 同样处于 LOOKING 状态。
3. 服务器 3 启动， 给自己投票，然后与其他服务投票信息交换结果，由于服务器 3 的编号大于服务器 2，1，所以服务器 3胜出， 并且此时投票数正好大于半数， 所以选举结束，服务器 3 处于LEADING 状态，服务器 1，服务器 2 处于 FOLLOWING 状态。
4. 服务器 4 启动，给自己投票，同时与之前的服务器 1 ，2，3交换信息，尽管服务器 4 的编号最大，但之前服务器 3 已经胜出，所以服务器 4 只能处于 FOLLOWING 状态。
5. 服务器 5 启动，同上。FOLLOWING状态。

### 1.3	非全新集群选举

对于运行正常的zookeeper集群，中途有机器down掉，需要重新选举时，选举过程就需要加入数据ID、服务器ID、和逻辑时钟。这样选举就变成：

1. 逻辑时钟小的选举结果被忽略，重新投票；（除去选举次数不完整的服务器）       
2. 统一逻辑时钟后，数据id大的胜出；（选出数据最新的服务器）
3. 数据id相同的情况下，服务器id大的胜出。（数据相同的情况下， 选择服务器id最大，即权重最大的服务器）

### 1.4	投票数据结构
每个投票中包含了两个最基本的信息，所推举服务器的SID和ZXID，投票（Vote）在Zookeeper中包含字段如下
		
 - **id：** 被推举的Leader的SID。
 - **zxid：** 被推举的Leader事务ID。
 - **electionEpoch：** 逻辑时钟，用来判断多个投票是否在同一轮选举周期中，该值在服务端是一个自增序列，每次进入新一轮的投票后，都会对该值进行加1操作。
 - **peerEpoch：** 被推举的Leader的epoch。
 - **state：** 当前服务器的状态。

### 1.5	每轮选举的流程

#### 服务器启动时期的Leader选举
1.  **每个Server发出一个投票。** 由于是初始情况，所有满足条件的Server都会将自己作为Leader服务器来进行投票，每次投票包含所推举的服务器的myid和ZXID，使用(myid, ZXID)来表示，然后各自将这个投票发给集群中其他机器。此时Server1的投票为(1, 0)，Server2的投票为(2, 0)，
2. **接受来自各个服务器的投票。** 集群的每个服务器收到投票后，首先判断该投票的有效性，如检查是否是本轮投票、是否来自LOOKING状态的服务器。
3. **处理投票。** 针对每一个投票，服务器都需要将别人的投票和自己的投票进行PK，PK规则如下：1.优先检查ZXID。ZXID比较大的服务器优先作为Leader。2.若ZXID相同，就比较myid。myid较大的服务器作为Leader服务器。结束pk之后服务器需要根据pk结果更新自己的投票状态。如Server1它的投票是(1, 0)，接收Server2的投票为(2, 0)，首先会比较两者的ZXID，均为0，再比较myid，此时Server2的myid最大，于是更新自己的投票为(2,0)，然后重新投票，对于Server2而言，其无须更新自己的投票，只是再次向集群中所有机器发出上一次投票信息即可。
4. **统计投票。** 每次投票后，服务器都会统计投票信息，判断是否已经有过半机器接受到相同的投票信息，对于Server1、Server2而言，都统计出集群中已经有两台机器接受了(2,0)的投票信息，此时便认为已经选出了Leader。
5. **改变服务器状态。** 一旦确定了Leader，每个服务器就会更新自己的状态，如果是Follower，那么就变更为FOLLOWING，如果是Leader，就变更为LEADING。

#### 服务器运行时期的Leader选举

1. **变更状态。** 	Leader挂后，余下的非Observer服务器都会讲自己的服务器状态变更为LOOKING，然后开始进入Leader选举过程。
2. **每个Server会发出一个投票。** 		在运行期间，每个服务器上的ZXID可能不同。发出选票与启动时期的Leader选举过程相同。
3. **重复启动时期的Leader选举过程。**

### 1.6 变更投票规则补充

每台机器发出投票后，也会收到其他机器的投票，每台机器会根据一定规则来处理收到的其他机器的投票，并以此来决定是否需要变更自己的投票，这个规则也是整个Leader选举算法的核心所在，其中术语描述如下
 - **vote_sid：** 接收到的投票中所推举Leader服务器的SID。
 - **vote_zxid：** 接收到的投票中所推举Leader服务器的ZXID。
 - **self_sid：** 当前服务器自己的SID。
 - **self_zxid：** 当前服务器自己的ZXID。
 
每次对收到的投票的处理，都是对(vote_sid,vote_zxid)和(self_sid, self_zxid)对比的过程。

- **规则一**：如果vote_zxid大于self_zxid，就认可当前收到的投票，并再次将该投票发送出去。
- **规则二**：如果vote_zxid小于self_zxid，那么坚持自己的投票，不做任何变更。
- **规则三**：如果vote_zxid等于self_zxid，那么就对比两者的SID，如果vote_sid大于self_sid，那么就认可当前收到的投票，并再次将该投票发送出去。
- **规则四**：如果vote_zxid等于self_zxid，并且vote_sid小于self_sid，那么坚持自己的投票，不做任何变更。

## 2. Zookeeper同步流程
选完Leader以后，zk就进入状态同步过程。 

1. Leader等待server连接；
2. Follower连接leader，将最大的zxid发送给leader；
3. Leader根据follower的zxid确定同步点；
4. 完成同步后通知follower 已经成为uptodate状态；
5. Follower收到uptodate消息后，又可以重新接受client的请求进行服务了。

![zo5](/resource_img/zookeeper/zookeeper05.png)


## 3. Zookeeper工作流程-Leader

1. 恢复数据； 
2. 维持与Learner的心跳，接收Learner请求并判断Learner的请求消息类型； 
3. 根据不同的消息类型，进行不同的处理。Learner的消息类型主要有PING消息、REQUEST消息、ACK消息、REVALIDATE消息：
PING 消息是指Learner的心跳信息；REQUEST消息是Follower发送的提议信息，包括写请求及同步请求；ACK消息是 Follower对提议的回复，超过半数的Follower通过，则commit该提议；REVALIDATE消息是用来延长SESSION有效时间。

![zo6](/resource_img/zookeeper/zookeeper06.png)

## 4. Zookeeper工作流程-Follower

### 4.1	Follower主要有四个功能： 
1. 向Leader发送请求（PING消息、REQUEST消息、ACK消息、REVALIDATE消息）；
2. 接收Leader消息并进行处理；
3. 接收Client的请求，如果为写请求，发送给Leader进行投票;
4. 返回Client结果。 

### 4.2	Follower的消息循环处理如下几种来自Leader的消息： 
1. PING消息： 心跳消息；
2. PROPOSAL消息：Leader发起的提案，要求Follower投票； 
3. COMMIT消息：服务器端最新一次提案的信息； 
4. UPTODATE消息：表明同步完成； 
5. REVALIDATE消息：根据Leader的REVALIDATE结果，关闭待revalidate的session还是允许其接受消息； 
6. SYNC消息：返回SYNC结果到客户端，这个消息最初由客户端发起，用来强制得到最新的更新。

![zo7](/resource_img/zookeeper/zookeeper07.png)

## 5. Zookeeper节点数据操作流程
1. 在Client向Follwer发出一个写的请求
2. Follwer把请求发送给Leader
3. Leader接收到以后开始发起投票并通知Follwer进行投票
4. Follwer把投票结果发送给Leader
5. Leader将结果汇总后如果需要写入，则开始写入同时把写入操作通知给Leader，然后commit;
6. Follwer把请求结果返回给Client

![zo8](/resource_img/zookeeper/zookeeper08.png)


> 结尾

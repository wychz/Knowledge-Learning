# 1. 集群实现方式

1. 实现基础——分区
   * 分区是分割数据到多个Redis实例的处理过程，因此每个实例只保存key的一个子集。
   * 通过利用多台计算机内存的和值，允许我们构造更大的数据库。
   * 通过多核和多台计算机，允许我们扩展计算能力；通过多台计算机和网络适配器，允许我们扩展网络带宽。

2. 集群的几种实现方式
   * 客户端分片
   * 基于代理的分片
   * 路由查询

### 1. 客户端分片

- 由客户端决定key写入或者读取的节点。
- 包括jedis在内的一些客户端，实现了客户端分片机制。

原理如下所示：

 

![img](https://upload-images.jianshu.io/upload_images/1521743-5490a7b34deae42c.png)

 

**特性**

- 优点
  - 简单，性能高。
- 缺点
  - 业务逻辑与数据存储逻辑耦合
  - 可运维性差
  - 多业务各自使用redis，集群资源难以管理
  - 不支持动态增删节点

### 2. 基于代理的分片

- 客户端发送请求到一个代理，代理解析客户端的数据，将请求转发至正确的节点，然后将结果回复给客户端。
- 开源方案
  - Twemproxy
  - codis

基本原理如下所示：

 

![img](https://upload-images.jianshu.io/upload_images/1521743-5b8e35bb04543259.png)

 

 

**特性**

- 透明接入Proxy 的逻辑和存储的逻辑是隔离的。
  - 业务程序不用关心后端Redis实例，切换成本低。
- 代理层多了一次转发，性能有所损耗。

### 3. 路由查询

- 将请求发送到任意节点，接收到请求的节点会将查询请求发送到正确的节点上执行。
- 开源方案
  - Redis-cluster

基本原理如下所示：



 

![img](https://upload-images.jianshu.io/upload_images/1521743-204c4cc057006efa.png)

 

### 集群的挑战

- 涉及多个key的操作通常是不被支持的。涉及多个key的redis事务不能使用。
  - 举例来说，当两个set映射到不同的redis实例上时，你就不能对这两个set执行交集操作。
- 不能保证集群内的数据均衡。
  - 分区的粒度是key，如果某个key的值是巨大的set、list，无法进行拆分。
- 增加或删除容量也比较复杂。
  - redis集群需要支持在运行时增加、删除节点的透明数据平衡的能力。

# 2. Redis集群各种方案原理

## 1. Twemproxy

- Proxy-based
- twtter开源，C语言编写，单线程。
- 支持 Redis 或 Memcached 作为后端存储。

 

![img](https://upload-images.jianshu.io/upload_images/1521743-4dc3262674558d8f.png)

 

### **Twemproxy特性**

- 支持失败节点自动删除
  - 与redis的长连接，连接复用，连接数可配置
- 自动分片到后端多个redis实例上支持redis pipelining 操作
  - 多种hash算法：能够使用不同的分片策略和散列函数
  - 支持一致性hash，但是使用DHT之后，从集群中摘除节点时，不会进行rehash操作
  - 可以设置后端实例的权重
- 支持状态监控
- 支持select切换数据库

### **Twemproxy不足**

- 性能低：代理层损耗 && 本身效率低下
- Redis功能支持不完善
  - 不支持针对多个值的操作，比如取sets的子交并补等（MGET 和 DEL 除外）
  - 不支持Redis的事务操作
  - 出错提示还不够完善
- 集群功能不够完善
  - 仅作为代理层使用
  - 本身不提供动态扩容，透明数据迁移等功能
- 失去维护
  - 最近一次提交在一年之前。Twitter内部已经不再使用。

## 2. Redis Cluster

### 3.1 节点

Redis Cluster是分布式架构：即Redis Cluster中有多个节点，每个节点都负责进行数据读写操作

每个节点之间会进行通信。

### 3.2 meet操作

节点之间会相互通信

meet操作是节点之间完成相互通信的基础，meet操作有一定的频率和规则

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027173608325-1809944413.png)

### 3.3 分配槽

把16384个槽平均分配给节点进行管理，每个节点只能对自己负责的槽进行读写操作

由于每个节点之间都彼此通信，每个节点都知道另外节点负责管理的槽范围

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027173506943-656215359.png)

客户端访问任意节点时，对数据key按照CRC16规则进行hash运算，然后对运算结果对16383进行取作，如果余数在当前访问的节点管理的槽范围内，则直接返回对应的数据
如果不在当前节点负责管理的槽范围内，则会告诉客户端去哪个节点获取数据，由客户端去正确的节点获取数据

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027173628442-1759433146.png)

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027173637265-984512111.png)

### 3.4 复制

保证高可用，每个主节点都有一个从节点，当主节点故障，Cluster会按照规则实现主备的高可用性

对于节点来说，有一个配置项：cluster-enabled，即是否以集群模式启动

### 3.5 客户端路由

#### 3.5.1 moved重定向

```
1.每个节点通过通信都会共享Redis Cluster中槽和集群中对应节点的关系
2.客户端向Redis Cluster的任意节点发送命令，接收命令的节点会根据CRC16规则进行hash运算与16383取余，计算自己的槽和对应节点
3.如果保存数据的槽被分配给当前节点，则去槽中执行命令，并把命令执行结果返回给客户端
4.如果保存数据的槽不在当前节点的管理范围内，则向客户端返回moved重定向异常
5.客户端接收到节点返回的结果，如果是moved异常，则从moved异常中获取目标节点的信息
6.客户端向目标节点发送命令，获取命令执行结果
```

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027173651666-1863525873.png)

需要注意的是：客户端不会自动找到目标节点执行命令

槽命中：直接返回

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027173702243-1050038981.png)

```
[root@mysql ~]# redis-cli -p 9002 cluster keyslot hello
(integer) 866
```

槽不命中：moved异常

```
[root@mysql ~]# redis-cli -p 9002 cluster keyslot php
(integer) 9244
```

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027173713886-2024280701.png)

```
[root@mysql ~]# redis-cli -c -p 9002
127.0.0.1:9002> cluster keyslot hello
(integer) 866
127.0.0.1:9002> set hello world
-> Redirected to slot [866] located at 192.168.81.100:9003
OK
192.168.81.100:9003> cluster keyslot python
(integer) 7252
192.168.81.100:9003> set python best
-> Redirected to slot [7252] located at 192.168.81.101:9002
OK
192.168.81.101:9002> get python
"best"
192.168.81.101:9002> get hello
-> Redirected to slot [866] located at 192.168.81.100:9003
"world"
192.168.81.100:9003> exit
[root@mysql ~]# redis-cli -p 9002
127.0.0.1:9002> cluster keyslot python
(integer) 7252
127.0.0.1:9002> set python best
OK
127.0.0.1:9002> set hello world
(error) MOVED 866 192.168.81.100:9003
127.0.0.1:9002> exit
[root@mysql ~]# 
```

#### 3.5.2 ask重定向

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027173742168-2092868404.png)

在对集群进行扩容和缩容时，需要对槽及槽中数据进行迁移

当客户端向某个节点发送命令，节点向客户端返回moved异常，告诉客户端数据对应的槽的节点信息

如果此时正在进行集群扩展或者缩空操作，当客户端向正确的节点发送命令时，槽及槽中数据已经被迁移到别的节点了，就会返回ask，这就是ask重定向机制

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027173806627-1735185965.png)

步骤：

```
1.客户端向目标节点发送命令，目标节点中的槽已经迁移支别的节点上了，此时目标节点会返回ask转向给客户端
2.客户端向新的节点发送Asking命令给新的节点，然后再次向新节点发送命令
3.新节点执行命令，把命令执行结果返回给客户端
```

moved异常与ask异常的相同点和不同点

```
两者都是客户端重定向
moved异常：槽已经确定迁移，即槽已经不在当前节点
ask异常：槽还在迁移中
```

## 3. Codis

Codis是一个豌豆荚团队开源的使用Go语言编写的Redis Proxy使用方法和普通的redis没有任何区别，设置好下属的多个redis实例后就可以了，使用时在本需要连接redis的地方改为连接codis，它会以一个代理的身份接收请求 并使用一致性hash算法，将请求转接到具体redis，将结果再返回codis，和之前比较流行的twitter开源的Twemproxy功能类似

下面我们看下如果使用Coids作为缓存集群方案的架构图，简单画了这么个架构图，这个架构是codis保证HA的前提下的最小级，从这张架构图可以看到我们最少需要8台机器，其中一台机器是codis的dashboard用于通过web界面可视化的配置codis group和proxy，也可以查看各个节点的状态；还有两台是用于codis的proxy代理节点，两个节点之间通过pipeline主从互备；还需要至少配置一台zk用于保存slot状态信息，也可以通过etcd存储这些状态信息，方便client请求的路由，也可以配置多台保证高可用；最后就是要配置数据节点来存储数据了，在codis中需要将数据节点都放在codis group中进行管理，每个group至少保留一个节点，该架构图中，为了保证HA，我们每个group都配置了一个master一个slave节点，这里配置了两个group，如果一个group中的master挂了，那么同一个group中的slave节点通过选举算法选出新的master节点，并通知到proxy，如果为了较好的高可用可以增加group的个数和每个group中slave节点的个数。

![img](https:////upload-images.jianshu.io/upload_images/4720632-3659cd151ffaf7db..png?imageMogr2/auto-orient/strip|imageView2/2/w/498/format/webp)

codis方案推出的时间比较长，而且国内很多互联网公司都已经使用了该集群方案，所以该方案还是比较适合大型互联网系统使用的，毕竟成功案例比较多，但是codis因为要实现slot切片，所以修改了redis-server的源码，对于后续的更新升级也会存在一定的隐患。，但是codis的稳定性和高可用确实是目前做的最好的，只要有足够多的机器能够做到非常好的高可用缓存系统。

# 集群原理详解

https://www.cnblogs.com/kismetv/p/9853040.html
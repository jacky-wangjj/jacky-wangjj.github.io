---
layout: post
title: 大数据面试之Zookeeper知识点
date: 2019-07-26
tags: interview
---
### zookeeper基础知识
#### 什么是zookeeper
zookeeper是一套高吞吐的分布式协调系统。

1） zookeeper的主要作用是为分布式系统提供协调服务，包括但不限于：分布式锁，统一命名服务，配置管理，负载均衡，主控服务器选举以及主从切换等。

2） zookeeper是分布式服务，可提供高可用的特性。客户端通过tcp协议连接任意一个服务端节点请求zookeeper集群提供服务，而集群内部如何通信以及如何保持分布式数据一致性等细节对客户端透明。只要zookeeper集群中超过一半的节点存活，便可正常对外提供服务。

3）zookeeper是以高吞吐量为目标进行设计的，故而在读多写少(10:1)的场合有非常好的性能表现。

**zookeeper具有高吞吐特性的主要原因：**

1）zookeeper集群的任意一个服务端节点都可以直接响应客户端的读请求（写请求不一样），并且可以通过增加节点进行横向扩展。这是其吞吐量高的主要原因。

2）zookeeper将全量数据存储于内存中,从内存中读取数据不需要进行磁盘IO,速度要快得多。

3）zookeeper放松了对分布式数据的强一致性要求,即不保证数据实时一致,允许分布式数据经过一个时间窗口达到最终一致,这也在一定程度上提高了其吞吐量。

#### zookeeper集群角色
在Zookeeper集群中,分别有Leader,Follower和Observer三种类型的服务器角色。

**Leader**

Leader服务器在整个正常运行期间有且仅有一台,集群会通过选举的方式选举出Leader服务器,由它同统一处理集群的事务性请求以及集群内各服务器的调度。

**Follower**

Follower的主要职责有以下几点:

1.参与Leader选举投票

2.参与事务请求Proposal的投票

3.处理客户端非事务请求(读),并转发事务请求(写)给Leader服务器。

**Observer**

Observer是弱化版的Follower。其像Follower一样能够处理非事务也就是读请求,并转发事务请求给Leader服务器,但是其不参与任何形式的投票,不管是Leader选举投票还是事务请求Proposal的投票。引入这个角色主要是为了在不影响集群事务处理能力的前提下提升集群的非事务处理的吞吐量。

**Znode的类型**

1） 持久节点（PERSISTENT）

2） 持久顺序节点（PERSISTENT_SEQUENTIAL）

3） 临时节点（EPHEMERAL）

4） 临时顺序节点（EPHEMERAL_SEQUENTIAL）

**一致性保证**

1） 顺序一致性：来自客户端的更新请求将按顺序进行操作；

2） 原子性：更新要么成功、要么失败，没有中间结果；

3） 单系统镜像：无论连接那个服务器，客户端都将看到相同的服务视图；

4） 可靠性：一旦更新，它将保持直到下一次被客户端更新；

5） 及时性：系统的客户端视图保证在一定的时间范围内是最新的。

zookeeper采用ZAB(Zookeeper Atomic Broadcast)协议来保证分布式数据一致性。ZAB协议包括两种基本模式：崩溃恢复模式和消息广播模式。崩溃恢复模式主要用来在集群启动过程或Leader服务器崩溃退出后进行新的Leader服务器的选举以及数据同步；消息广播模式主要用来进行事务请求的处理。

ZAB协议详解：https://www.jianshu.com/p/2bceacd60b8a

### zookeeper选举机制
1.集群刚启动时

2.集群运行时，但Leader服务器因故退出

选票主要有两个信息组成：一是所推举的Leader服务器的ID（即配置在myid文件中的数字）。二是该服务器的事务ID，事务表示对服务器状态变更的操作，一个服务器的事务ID越大，则其数据越新。过程如下：

a.Follower服务器投出选票（SID,ZXID），第一次每个Follower都会推选自己为Leader服务器，也就是说每个Follower第一次投出的选票是自己服务器ID和事务ID。

b.每个Follower都会接收来自于其他Follower的选票，它会基于如下规则重新生成一张选票：比较收到的选票和自己的ZXID的大小，选取其中最大的；若ZXID一样则选取SID即服务器ID最大的。最终每个服务器都会重新生成一张选票，并将该选票投出去。

c.经过多轮投票后，如果某一台服务器得到了超过半数的选票，则其将当前选为Leader。

### zookeeper的应用场景
（1）统一命名服务

（2）配置管理（同步更新）
具体实现：可将配置信息写到Zookeeper的一个znode上，各个节点监听这个znode，一旦znode中的数据修改，Zookeeper将通知各个节点。

（3）集群管理  
典型应用场景：Hbase中master状态监听与选举

（4）分布式通知与协调

（5）分布式锁

（6）分布式队列

### zookeeper容错
zookeeper通过事务日志和数据快照来避免因为服务器故障导致的数据丢失。
- 事务日志是指服务器在更新内存数据前先将事务操作以日志的方式写入磁盘，Leader和Follower都会记录事务日志。

- 数据快照是指周期性通过深度遍历的方式将内存中的树形结构数据转入外存快照中。

### zookeeper data model
https://zookeeper.apache.org/doc/r3.5.5/zookeeperProgrammers.html
- znode
Znodes maintain a stat structure that includes version numbers for data changes, ACL changes, and timestamps, to allow cache validations and coordinated updates. Each time a znode's data changes, the version number increases. For instance, whenever a client retrieves data it also receives the version of the data.

- watches
Clients can set watches on znodes. Changes to that znode trigger the watch and then clear the watch. When a watch triggers, ZooKeeper sends the client a notification.

- data access
Each node has an Access Control List (ACL) that restricts who can do what.

- ephemeral nodes
ZooKeeper also has the notion of ephemeral nodes. These znodes exists as long as the session that created the znode is active. When the session ends the znode is deleted. Because of this behavior ephemeral znodes are not allowed to have children.

- Sequence nodes
When creating a znode you can also request that ZooKeeper append a monotonically increasing counter to the end of path. This counter is unique to the parent znode. The counter has a format of %010d -- that is 10 digits with 0 (zero) padding (the counter is formatted in this way to simplify sorting), i.e. "0000000001".

- Container nodes
ZooKeeper has the notion of container znodes. Container znodes are special purpose znodes useful for recipes such as leader, lock, etc. When the last child of a container is deleted, the container becomes a candidate to be deleted by the server at some point in the future.
Given this property, you should be prepared to get KeeperException.NoNodeException when creating children inside of container znodes. i.e. when creating child znodes inside of container znodes always check for KeeperException.NoNodeException and recreate the container znode when it occurs.

#### zookeeper指定字段解释
- Zxid：Zookeeper Transaction Id

- Version numbers
Every change to a node will cause an increase to one of the version numbers of that node. The three version numbers are version (number of changes to the data of a znode), cversion (number of changes to the children of a znode), and aversion (number of changes to the ACL of a znode).

#### zookeeper stat structure
The Stat structure for each znode in ZooKeeper is made up of the following fields:
- **czxid** The zxid of the change that caused this znode to be created.
- **mzxid** The zxid of the change that last modified this znode.
- **pzxid** The zxid of the change that last modified children of this znode.
- **ctime** The time in milliseconds from epoch when this znode was created.
- **mtime** The time in milliseconds from epoch when this znode was last modified.
- **version** The number of changes to the data of this znode.
- **cversion** The number of changes to the children of this znode.
- **aversion** The number of changes to the ACL of this znode.
- **ephemeralOwner** The session id of the owner of this znode if the znode is an ephemeral node. If it is not an ephemeral node, it will be zero.
- **dataLength** The length of the data field of this znode.
- **numChildren** The number of children of this znode.

#### zookeeper watches
All of the read operations in ZooKeeper - getData(), getChildren(), and exists() - have the option of setting a watch as a side effect. Here is ZooKeeper's definition of a watch: a watch event is one-time trigger, sent to the client that set the watch, which occurs when the data for which the watch was set changes.
We can set watches with the three calls that read the state of ZooKeeper: exists, getData, and getChildren. The following list details the events that a watch can trigger and the calls that enable them:
- Created event: Enabled with a call to exists.
- Deleted event: Enabled with a call to exists, getData, and getChildren.
- Changed event: Enabled with a call to exists and getData.
- Child event: Enabled with a call to getChildren.

#### ACL Permissions
ZooKeeper supports the following permissions:

- CREATE: you can create a child node
- READ: you can get data from a node and list its children.
- WRITE: you can set data for a node
- DELETE: you can delete a child node
- ADMIN: you can set permissions

### zookeeper shell cmd
```shell
## 连接zookeeper
$ bin/zkCli.sh -server 127.0.0.1:2181

## 查看帮助
[zkshell: 0] help
ZooKeeper host:port cmd args
    get path [watch]
    ls path [watch]
    set path data [version]
    delquota [-n|-b] path
    quit
    printwatches on|off
    create path data acl
    stat path [watch]
    listquota path
    history
    setAcl path acl
    getAcl path
    sync path
    redo cmdno
    addauth scheme auth
    delete path [version]
    deleteall path
    setquota -n|-b val path

## 查看特定znode的子节点
[zkshell: 0] ls /

## 创建新的znode
[zkshell: 0] create /zk_test my_data

## 获取znode的数据
[zkshell: 0] get /zk_test

## 修改znode的数据
[zkshell: 0] set /zk_test new_data

## 删除znode
[zkshell: 0] delete /zk_test
```

### zookeeper工作流
![](https://jacky-wangjj.github.io/images/blog/interview/2019-07-26-Interview_Zookeeper_Summary-44562768.png#pic_center)

|               组件                |                                                        描述                                                         |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
|           写入（write）           | 写入过程由leader节点处理。leader将写入请求转发到所有znode，并等待znode的回复。如果一半的znode回复，则写入过程完成。 |
|           读取（read）            |                             读取由特定连接的znode在内部执行，因此不需要与集群进行交互。                             |
| 复制数据库（replicated database） |        它用于在zookeeper中存储数据。每个znode都有自己的数据库，每个znode在一致性的帮助下每次都有相同的数据。        |
|              Leader               |                                          Leader是负责处理写入请求的Znode。                                          |
|             Follower              |                             follower从客户端接收写入请求，并将它们转发到leader znode。                              |
|  请求处理器（request processor）  |                               只存在于leader节点。它管理来自follower节点的写入请求。                                |
|   原子广播（atomic broadcasts）   |                                     负责广播从leader节点到follower节点的变化。                                      |

### zookeeper API
create : creates a node at a location in the tree

delete : deletes a node

exists : tests if a node exists at a location

get data : reads the data from a node

set data : writes data to a node

get children : retrieves a list of children of a node

sync : waits for data to be propagated

[示例代码](https://www.yiibai.com/zookeeper/zookeeper_api.html)

### zookeeper应用程序
- [Barrier Queue](https://zookeeper.apache.org/doc/r3.5.5/zookeeperTutorial.html)

- [watcher接口实现示例、集群监控器示例](https://www.jianshu.com/p/44c3d73dc30d)

- 分布式锁：
为了使用ZooKeeper实现分布式锁，我们使用可排序的znode来实现进程对锁的竞争。思路其实很简单：首先，我们需要一个表示锁的znode，获得锁的进程就表示被这把锁给锁定了（命名为，/leader）。然后，client为了获得锁，就需要在锁的znode下创建ephemeral类型的子znode。在任何时间点上，只有排序序号最小的znode的client获得锁，即被锁定。例如，如果两个client同时创建znode /leader/lock-1和/leader/lock-2，所以创建/leader/lock-1的client获得锁，因为他的排序序号最小。ZooKeeper服务被看作是排序的权威管理者，因为是由他来安排排序的序号的。 锁可能因为删除了/leader/lock-1znode而被简单的释放。另外，如果相应的客户端死掉，使用ephemeral znode的价值就在这里，znode可以被自动删除掉。创建/leader/lock-2的client就获得了锁，因为他的序号现在最小。当然客户端需要启动观察模式，在znode被删除时才能获得通知：此时他已经获得了锁。 获得锁的伪代码如下：
1）在lock的znode下创建名字为lock-的ephemeral类型znode，并记录下创建的znode的path（会在创建函数中返回）。
2）获取lock znode的子节点列表，并开启对lock的子节点的watch模式。
3）如果创建的子节点的序号最小，则再执行一次第2步，那么就表示已经获得锁了。退出。
等待第2步的观察模式的通知，如果获得通知，则再执行第2步。

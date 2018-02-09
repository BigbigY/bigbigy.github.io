---
title: Zookeeper基本原理
tags: zookeeper
categories: 分布式
type: "categories"
---
![](http://ocppiicaw.bkt.clouddn.com/zookeeperzookeeper1.jpg)
<!--more-->
# Zookeeper基本原理 #

## 1.1.1    Zookeeper的保证 ##


顺序性，client的updates请求都会根据它发出的顺序被顺序的处理；

原子性,  一个update操作要么成功要么失败，没有其他可能的结果；

一致的镜像，client不论连接到哪个server，展示给它都是同一个视图；

可靠性，一旦一个update被应用就被持久化了，除非另一个update请求更新了当前值

实时性，对于每个client它的系统视图都是最新的

## 1.1.2    Zookeeper server角色 ##

领导者（Leader) : 领导者不接受client的请求，负责进行投票的发起和决议，最终更新状态。

跟随者（Follower）: Follower用于接收客户请求并返回客户结果。参与Leader发起的投票。

观察者（observer）: Oberserver可以接收客户端连接，将写请求转发给leader节点。但是Observer不参加投票过程，只是同步leader的状态。Observer为系统扩展提供了一种方法。

学习者 ( Learner ) : 和leader进行状态同步的server统称Learner，上述Follower和Observer都是Learner。

## 1.1.3    Zookeeper集群 ##

 

通常Zookeeper由2n+1台servers组成，每个server都知道彼此的存在。每个server都维护的内存状态镜像以及持久化存储的事务日志和快照。对于2n+1台server，只要有n+1台（大多数）server可用，整个系统保持可用。

系统启动时，集群中的server会选举出一台server为Leader，其它的就作为follower（这里先不考虑observer角色）。接着由follower来服务client的请求，对于不改变系统一致性状态的读操作，由follower的本地内存数据库直接给client返回结果；对于会改变系统状态的更新操作，则交由Leader进行提议投票，超过半数通过后返回结果给client。

# 二．Zookeeper server工作原理 #

Zookeeper的核心是原子广播，这个机制保证了各个server之间的同步。实现这个机制的协议叫做Zab协议。Zab协议有两种模式，它们分别是恢复模式和广播模式。当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数server的完成了和leader的状态同步以后，恢复模式就结束了。状态同步保证了leader和server具有相同的系统状态。

一旦leader已经和多数的follower进行了状态同步后，他就可以开始广播消息了，即进入广播状态。这时候当一个server加入zookeeper服务中，它会在恢复模式下启动，发现leader，并和leader进行状态同步。待到同步结束，它也参与消息广播。Zookeeper服务一直维持在Broadcast状态，直到leader崩溃了或者leader失去了大部分的followers支持。

Broadcast模式极其类似于分布式事务中的2pc（two-phrase commit 两阶段提交）：即leader提起一个决议，由followers进行投票，leader对投票结果进行计算决定是否通过该决议，如果通过执行该决议（事务），否则什么也不做。

广播模式需要保证proposal被按顺序处理，因此zk采用了递增的事务id号(zxid)来保证。所有的提议(proposal)都在被提出的时候加上了zxid。实现中zxid是一个64为的数字，它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个新的epoch。低32位是个递增计数。

当leader崩溃或者leader失去大多数的follower，这时候zk进入恢复模式，恢复模式需要重新选举出一个新的leader，让所有的server都恢复到一个正确的状态。

首先看一下选举的过程，zk的实现中用了基于paxos算法（主要是fastpaxos）的实现。具体如下：

1.每个Server启动以后都询问其它的Server它要投票给谁。

2.对于其他server的询问，server每次根据自己的状态都回复自己推荐的leader的id和上一次处理事务的zxid（系统启动时每个server都会推荐自己）

3.收到所有Server回复以后，就计算出zxid最大的哪个Server，并将这个Server相关信息设置成下一次要投票的Server。

4.计算这过程中获得票数最多的的sever为获胜者，如果获胜者的票数超过半数，则改server被选为leader。否则，继续这个过程，直到leader被选举出来。

此外恢复模式下，如果是重新刚从崩溃状态恢复的或者刚启动的的server还会从磁盘快照中恢复数据和会话信息。（zk会记录事务日志并定期进行快照，方便在恢复时进行状态恢复）

选完leader以后，zk就进入状态同步过程。

1.leader就会开始等待server连接

2.Follower连接leader，将最大的zxid发送给leader

3.Leader根据follower的zxid确定同步点

4.完成同步后通知follower 已经成为uptodate状态

5.Follower收到uptodate消息后，又可以重新接受client的请求进行服务了。

# 三. ZookeeperServer工作流程 #

 

### 3.1.1 主线程的工作： ###

1. 刚开始时各个Server处于一个平等的状态peer

2. 主线程加载配置后启动。

3. 主线程启动QuorumPeer线程，该线程负责管理多数协议（Quorum），并根据表决结果进行角色的状态转换。

4. 然后主线程等待QuorumPeer线程。

### 3.1.2  QuorumPeer线程 ###

1. 首先会从磁盘恢复zkdatabase（内存数据库），并进行快照回复。

2. 然后启动server的通信线程，准备接收client的请求。

3. 紧接着该线程进行选举leader准备，选择选举算法，启动response线程（根据自身状态）向其他server回复推荐的leaer。

4. 刚开始的时候server都处于looking状态，进行选举根据选举结果设置自己的状态和角色。

### 3.1.3 quorumPeer有几种状态 ###

1. Looking: 寻找状态，这个状态不知道谁是leader，会发起leader选举

2. Observing: 观察状态，这时候observer会观察leader是否有改变，然后同步leader的状态

3. Following:  跟随状态，接收leader的proposal ，进行投票。并和leader进行状态同步

4. Leading:    领导状态，对Follower的投票进行决议，将状态和follower进行同步

当一个Server发现选举的结果自己是Leader把自己的状态改成Leading，如果Server推荐了其他人为Server它将自己的状态改成Following。做Leader的server如果发现拥有的follower少于半数时，它重新进入looking状态，重新进行leader选举过程。（Observing状态是根据配置设置的）。

## 3.2 Leader的工作流程： ##

 

### 3.2.1 Leader主线程： ###

1.首先leader开始恢复数据和清除session

启动zk实例，建立请求处理链(Leader的请求处理链)：PrepRequestProcessor->ProposalRequestProcessor->CommitProcessor->Leader.ToBeAppliedRequestProcessor ->FinalRequestProcessor

2.得到一个新的epoch，标识一个新的leader , 并获得最大zxid（方便进行数据同步）

3.建立一个学习者接受线程（来接受新的followers的连接，follower连接后确定followers的zxvid号，来确定是需要对follower进行什么同步措施，比如是差异同步(diff)，还是截断（truncate）同步，还是快照同步）

4. 向follower建立一个握手过程leader->follower NEWLEADER消息，并等待直到多数server发送了ack

5. Leader不断的查看已经同步了的follower数量，如果同步数量少于半数，则回到looking状态重新进行leaderElection过程，否则继续step5.

### 3.2.2 LearnerCnxAcceptor线程 ###

1.该线程监听Learner的连接

2.接受Learner请求，并为每个Learner创建一个LearnerHandler来服务

3.2.3 LearnerHandler线程的服务流程

1.检查server来的第一个包是否为follower.info或者observer.info，如果不是则无法建立握手。

2. 得到Learner的zxvid，对比自身的zxvid，确定同步点

3.和Learner建立第二次握手，向Learner发送NEWLEADER消息

4.与server进行数据同步。

5.同步结束，知会server同步已经ok，可以接收client的请求。

6. 不断读取follower消息判断消息类型

i.           如果是LEADER.ACK,记录follower的ack消息，超过半数ack，将proposal提交(Commit)

ii.         如果是LEADER.PING，则维持session（延长session失效时间）

iii.        如果是LEADER.REQEST，则将request放入请求链进行处理–Leader写请求发起proposal，然后根据follower回复的结果来确定是否commit的。最后由FinallRequestProcessor来实际进行持久化，并回复信息给相应的response给server

## 3.3 Follower的工作流程: ##

1.启动zk实例，建立请求处理链:FollowerRequestProcessor->CommitProcessor->FinalProcessor

2.follower首先会连接leader，并将zxid和id发给leader

3.接收NEWLEADER消息，完成握手过程。

4.同leader进行状态同步

5.完成同步后，follower可以接收client的连接

5.接收到client的请求,根据请求类型

对于写操作, FollowerRequestProcessor会将该操作作为LEADER.REQEST发给LEADER由LEADER发起投票。

对于读操作，则通过请求处理链的最后一环FinalProcessor将结果返回给客户端

对于observer的流程不再赘述，observer流程和Follower的唯一不同的地方就是observer不会参加leader发起的投票。

# 三．关于Zookeeper的扩展 #

为了提高吞吐量通常我们只要增加服务器到Zookeeper集群中。但是当服务器增加到一定程度，会导致投票的压力增大从而使得吞吐量降低。因此我们引出了一个角色：Observer。

Observers 的需求源于 ZooKeeper  follower服务器在上述工作流程中实际扮演了两个角色。它们从客户端接受连接与操作请求，之后对操作结果进行投票。这两个职能在 ZooKeeper集群扩展的时候彼此制约。如果我们希望增加 ZooKeeper 集群服务的客户数量（我们经常考虑到有上万个客户端的情况），那么我们必须增加服务器的数量，来支持这么多的客户端。然而，从一致性协议的描述可以看到，增加服务器的数量增加了对协议的投票部分的压力。领导节点必须等待集群中过半数的服务器响应投票。于是，节点的增加使得部分计算机运行较慢，从而拖慢整个投票过程的可能性也随之提高，投票操作的会随之下降。这正是我们在实际操作中看到的问题——随着 ZooKeeper 集群变大，投票操作的吞吐量会下降。

所以需要增加客户节点数量的期望和我们希望保持较好吞吐性能的期望间进行权衡。要打破这一耦合关系，引入了不参与投票的服务器，称为 Observers。 Observers 可以接受客户端的连接，将写请求转发给领导节点。但是，领导节点不会要求 Observers 参加投票。相反，Observers 不参与投票过程，仅仅和其他服务节点一起得到投票结果。

这个简单的扩展给 ZooKeeper 的可伸缩性带来了全新的镜像。我们现在可以加入很多 Observers 节点，而无须担心严重影响写吞吐量。规模伸缩并非无懈可击——协议中的一歩（通知阶段）仍然与服务器的数量呈线性关系。但是，这里的穿行开销非常低。因此可以认为在通知服务器阶段的开销无法成为主要瓶颈。

上图显示了一个简单评测的结果。纵轴是从一个单一的客户端发出的每秒钟同步写操作的数量。横轴是 ZooKeeper 集群的尺寸。蓝色的是每个服务器都是 voting 服务器的情况，而绿色的则只有三个是 voting 服务器，其它都是 Observers。图中看到，扩充 Observers，写性能几乎可以保持不变，但如果同时扩展 voting 节点的数量的话，性能会明显下降。显然 Observers 是有效的。因此Observer可以用于提高Zookeeper的伸缩性。

此外Observer还可以成为特定场景下，广域网部署的一种方案。原因有三点：1.为了获得更好的读性能，需要让客户端足够近，但如果将投票服务器分布在两个数据中心，投票的延迟太大会大幅降低吞吐，是不可取的。因此希望能够不影响投票过程，将投票服务器放在同一个IDC进行部署，Observer可以跨IDC部署。2. 投票过程中，Observer和leader之间的消息、要远小于投票服务器和server的消息，这样远程部署对带宽要求就较小。3.由于Observers即使失效也不会影响到投票集群，这样如果数据中心间链路发生故障，不会影响到服务本身的可用性。这种故障的发生概率要远高于一个数据中心中机架间的连接的故障概率，所以不依赖于这种链路是个优点。

 

Znodes

每个节点在zookeeper中用znode表示。znodes 包含数据变更和acl变更的版本号。znode同样包含时间戳。版本号和时间戳用来帮助zookeeper验证缓存或者协调更新。每次znode数据发生变化都会使版本号增加。例如，每次client接受数据时都会接收到数据的版本号。当client更新或者删除数据时必须给znode提供数据的版本号。如果提供的版本号与实际的版本号不匹配，更新操作会失败。

znode是程序访问的主要实体类。包含如下特性：

Watches 

  clients 可以为znode设置watch。znode发生改变将会触发watch。当一个watch触发，zookeeper会向client发送通知。

 

Data Access

  在namespace中存储在每个znode上的数据发生的读写操作都是原子性的。读一个znode上的全部数据或者替换掉全部数据都是原子性的。每个znode都有一个Access Contron List（ACL）用来约束哪些人可以执行相应操作。

    Zookeeper不是用来做数据库或者存贮大对象的。相反，它只负责协调数据。数据可以来自配置表单、结构化信息等等。这些数据的有一个共同的特点那就是都很小：以Kb为测量单位。Zookeeper的client和server的实现类都会验证znode存储的数据是否小于1M，但是数据应该比平均值小的多。操作大数据将会触发一些消耗时间的额外操作并且影响潜在的操作，因为需要额外的时间在网络和存储介质上转移数据。如果有大数据需要存储，通常的办法是把这些数据存储在专门的大型文件系统上，例如NFS或者HDFS，然后把存储介质的位置存在zookeeper上。

 

Ephemeral Nodes

zookeeper有一种znode是ephemeral nodes。这些znode只在session存在期间有效。当session结束的时候这些ephemeral nodes被删除。所以ephemeral znodes不能有子节点。

 

Sequence Nodes -- Unique Naming

当创建一个znode时候，你也可以要求zookeeper在path的结尾单调递增。计数器对每一个znode来说都是唯一的。计数器使用%010d格式化--例如<path>0000000001。注意：计数器使用一个singed int(4bytes)来存储下一个序列值。所以计数器达到2147483647 后会溢出。

 

zookeeper的每个节点可以有如下三种角色：

1.leader和follower

 

      ZooKeeper需要在所有的服务（可以理解为服务器）中选举出一个Leader，然后让这个Leader来负责管理集群。此时，集群中的其它服务器则成为此Leader的Follower。并且，当Leader故障的时候，需要ZooKeeper能够快速地在Follower中选举出下一个Leader。这就是ZooKeeper的Leader机制，下面我们将简单介绍在ZooKeeper中，Leader选举（Leader Election）是如何实现的。

此操作实现的核心思想是：首先创建一个EPHEMERAL目录节点，例如“/election”。然后。每一个ZooKeeper服务器在此目录下创建一个SEQUENCE|EPHEMERAL类型的节点，例如“/election/n_”。在SEQUENCE标志下，ZooKeeper将自动地为每一个ZooKeeper服务器分配一个比前一个分配的序号要大的序号。此时创建节点的ZooKeeper服务器中拥有最小序号编号的服务器将成为Leader。

在实际的操作中，还需要保障：当Leader服务器发生故障的时候，系统能够快速地选出下一个ZooKeeper服务器作为Leader。一个简单的解决方案是，让所有的follower监视leader所对应的节点。当Leader发生故障时，Leader所对应的临时节点将会自动地被删除，此操作将会触发所有监视Leader的服务器的watch。这样这些服务器将会收到Leader故障的消息，并进而进行下一次的Leader选举操作。但是，这种操作将会导致“从众效应”的发生，尤其当集群中服务器众多并且带宽延迟比较大的时候，此种情况更为明显。

在Zookeeper中，为了避免从众效应的发生，它是这样来实现的：每一个follower对follower集群中对应的比自己节点序号小一号的节点（也就是所有序号比自己小的节点中的序号最大的节点）设置一个watch。只有当follower所设置的watch被触发的时候，它才进行Leader选举操作，一般情况下它将成为集群中的下一个Leader。很明显，此Leader选举操作的速度是很快的。因为，每一次Leader选举几乎只涉及单个follower的操作。

 

 

2.Observer

      observer的行为在大多数情况下与follower完全一致, 但是他们不参加选举和投票, 而仅仅接受(observing)选举和投票的结果.
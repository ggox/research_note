### ZAB协议（zookeeper原子广播）

#### 名词解释

* electionEpoch：每执行一次leader选举，electionEpoch就会自增，用来标记leader选举的轮次

* peerEpoch：每次leader选举完成之后，都会选举出一个新的peerEpoch，用来标记事务请求所属的轮次

* zxid：事务请求的唯一标记，由leader服务器负责进行分配。由2部分构成，高32位是上述的peerEpoch，低32位是请求的计数，从0开始。所以由zxid我们就可以知道该请求是哪个轮次的，并且是该轮次的第几个请求

* lastProcessedZxid：最后一次commit的事务请求的zxid

#### 流程

* Leader election

  leader选举过程，electionEpoch自增，在选举的时候lastProcessedZxid越大，越有可能成为leader

* Discovery：

  第一：leader收集follower的lastProcessedZxid，这个主要用来通过和leader的lastProcessedZxid对比来确认follower需要同步的数据范围

  第二：选举出一个新的peerEpoch，主要用于防止旧的leader来进行提交操作（旧leader向follower发送命令的时候，follower发现zxid所在的peerEpoch比现在的小，则直接拒绝，防止出现不一致性）

* Synchronization：

  follower中的事务日志和leader保持一致的过程，就是依据follower和leader之间的lastProcessedZxid进行，follower多的话则删除掉多余部分，follower少的话则补充，一旦对应不上则follower删除掉对不上的zxid及其之后的部分然后再从leader同步该部分之后的数据

* Broadcast

  正常处理客户端请求的过程。leader针对客户端的事务请求，然后提出一个议案，发给所有的follower，一旦过半的follower回复OK的话，leader就可以将该议案进行提交了，向所有follower发送提交该议案的请求，leader同时返回OK响应给客户端

*注意：这是zab paper的内容，和zookeeper源码实现还是有出入的*

[zookeeper zab协议源码实现参考](https://my.oschina.net/pingpangkuangmo/blog/778927)


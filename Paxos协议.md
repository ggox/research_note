### Paxos协议

#### 两个原则

* 安全原则--保证不做错事
  1. 只能有一个值被批准，不能出现第二个值把第一个覆盖的情况
  2. 一个值被批准了，其他服务器最终会学习到这个值
* 存活原则--只要多数服务器存活并且彼此间可以通信则最终都要做到的事
  1. 最终会批准某个被提议的值
  2. 一个值被批准了，其他服务器最终会学习到这个值

#### 角色

* Proposer(提议者)：提议发起者，处理客户端请求，将客户端的请求发送到集群中，以便决定这个值是否可以被批准
* Acceptor(决策者)：提议批准者，负责处理接收到的提议，他们的回复就是一次投票。会存储一些状态来决定是否接收一个值

#### 流程

* prepare阶段
  1. P1a：Proposer 发送 Prepare
  
     Proposer 生成全局唯一且递增的提案 ID（Proposalid，以高位时间戳 + 低位机器 IP 可以保证唯一性和递增性），向 Paxos 集群的所有机器发送 PrepareRequest，这里无需携带提案内容，只携带 Proposalid 即可。
  
  2. P1b：Acceptor 应答 Prepare
  
     Acceptor 收到 PrepareRequest 后，做出“两个承诺，一个应答”
  
     **两个承诺**
  
     * 第一，不再应答 Proposalid 小于等于（注意：这里是 <= ）当前请求的 PrepareRequest
     * 第二，不再应答 Proposalid 小于（注意：这里是 < ）当前请求的 AcceptRequest
  
     **一个应答**
  
     * 返回自己已经 Accept 过的提案中 Proposalid 最大的那个提案的内容，如果没有则返回空值
  
     *注意，这“两个承诺”中，蕴含了两个要点：*
  
     1. 就是应答当前请求前，也要按照“两个承诺”检查是否会违背之前处理 PrepareRequest 时做出的承诺
     2. 应答前要在本地持久化当前 Propsalid
* accept阶段
  1. P2a：Proposer 发送 Accept
  
     “提案生成规则”：Proposer 收集到多数派应答的 PrepareResponse 后，从中选择Proposalid最大的提案内容，作为要发起 Accept 的提案，如果这个提案为空值，则可以自己随意决定提案内容。然后携带上当前 Proposalid，向 Paxos 集群的所有机器发送 AccpetRequest
  
  2. P2b：Acceptor 应答 Accept
  
     Accpetor 收到 AccpetRequest 后，检查不违背自己之前作出的“两个承诺”情况下，持久化当前 Proposalid 和提案内容。最后 Proposer 收集到多数派应答的 AcceptResponse 后，形成决议

#### Paxos的证明

*请参考：*[如何浅显易懂地解说 Paxos 的算法？ - 朱一聪的回答 - 知乎](https://www.zhihu.com/question/19787937/answer/82340987)












#第十期
##zookeeper
**1、zookeeper的zab协议详解**
    
    1、Zab（zookeeper atomic broadcast）zookeeper原子广播，通过zab协议实现最终一致性。
    2、Zab特别为zookeeper设计的支持崩溃恢复的原子广播协议，基于该协议，zookeeper实现了一种主备模型的系统
    架构，保证集群中个副本的数据一致性。
    3、zab协议核心：
        在zookeeper中只有一个leader，并且只有leader处理操作的事务请求，并将其转换成一个事务proposal（写操作）
        然后Leader服务器在将事务proposal操作的数据同步到所有follower（原子广播/复制）.
        效果：只要有台服务器提交了proposal，就要确保所有服务器最终都能正确的提交Proposal。
**2 、ZAB模式：崩溃恢复模式，消息广播模式**
    
   1、消息广播协议
   ![](/image/zookeeper01.png)
    
    1、客户端发起一个写操作请求
    2、leader处理客户端请求并转换为proposal，同时为每个proposal分配全局唯一ID，即ZXID
    3、leader与每个follower 之间都有一个队列，leader将此消息发送到队列
    4、follower机器从队列中取出消息处理完（写入本地内存）后，向leader发送ACk确认
    5、leader服务器收到半数以上的follower的ACK，即认为可以发送commit
    6、leader向follwer服务器发送commit消息
   2、崩溃恢复模式
   
    1、当整个集群刚刚启动，或者leader服务器宕机、重启、或者网络故障导致不存在过半的服务器与leader保持正常通信，就会进入崩溃恢复模式
    2、为保证顺序执行，只能在leader服务器上处理写操作，其他服务接收到写请求也会转发至leader服务器进行处理
    3、崩溃恢复需满足:(1)、已经被leader提交的proposal必须最终被所有的follower服务器提交
                   (2)、确保丢弃了被leader提出但没有的提交的proposal
    4、选举时将每个follower中的ZXID作为重要依据
    5、Leader选举算法
      (1)、通过配置项设置Zookeeper,选举leader的算法：
       <1>、0：基于UDP的leaderElection
       <2>、1：基于udp的fastLeaderElection
       <3>、2：基于UDP和认证的FastLeaderElction
       <4>、3：基于TCP的fastLeaderElection
      (2)、基于TCP的FastLeaderElection原理
        <1>、myid
        每个zookeeper服务器都有一个名为myid的文件，该文件包含整个zookeeper集群唯一id
   ![](/image/zookeeper02.png)
   
        <2>、ZXID
        事务id，用于标识proposalID，为保证顺序性，ZXID为单调递增，因此zookeeper使用64位数来表示
        高32位leader的epoch（标识:唐宋元明清）,低32位自增的序号，每次epoch更新，序号重置
      (3)、服务器状态
        <1>、  Looking：无leader，要选举          
        <2>、  Leading：leader角色
        <3>、  following：跟随者
        <4>、  Observing：观察者
      (4)、选票结构
      logicClock，每个服务器维护的自增整数，名为logicClock，他表示该服务器的选举轮次
      state：当前服务器状态
      self_id:当前myid
      self_zxid:当前事务最大的Zxid
      vote_id:被推举的服务器的myid
      vote_zxid:被推举的事务最大的ZXid
      (5)、选举过程
       <1>、自增选举轮次
       <2>、初始化选票：清空自身票箱
       <3>、发送初始化选票：投自己
       <4>、接受外部推举选票：确定与其他服务器建立连接
       <5>、判断选举轮次：外部大于自己则更新为推举的选票和轮次，小于则直接忽视，继续处理下一轮选票
       <6>、如果一致，则比较二者的vote_zxid，外部大，则更新vote_myid和vote_zxid,并广播出去，将收到的票
       和自己更新的票放到自己的票箱，如果ZXID一致则比较vote_myid，处理方式与上面相同
       <7>、统计计票，如果超过半数服务器的推选服务器相同则终止投票
       <8>、更新状态，被推举的服务器更新成leading状态，其他参与选举的Folloing
**3、zookeeper实现分布式锁原理**

    1、原理：zookeeper是基于临时顺序节点以及Watcher监听器机制实现分布式锁
    2、在每个节点下面创建临时顺序节点（ephemeral_sequential），新子节点后面会加上一个次序编号，并且是上一个次序编号加一
    3、创建一个持久父节点（persistent），每个要获得锁的线程都在这个节点下创建一个临时顺序节点，该节点按照创建的次序，依次递增
    4、判断最小节点获得锁
    5、zookeeper节点监听机制，保障占有锁的传递有序而且高效。每个线程抢占锁的时候会先创建临时顺序子节点，创建成功后，如果不是
    最小的节点，就处于通知等待，每一个等通知的节点，都需要监视序号在自己前面的那个Znode，只要上一个节点删除，就再次判断自己
    是不是最小的节点，如果是，自己就获得锁。不断地通知后面的Znode节点。
    java中的curator：
      InterProcessMutex：分布式可重入排它锁
      InterProcessSemaphoreMutex：分布式排它锁
      InterProcessReadWriteLock：分布式读写锁


               
                
           
       
   

      

    
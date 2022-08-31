#第十一期
##MQ中间件
**1、kafka**
    
    1、高吞吐量、分布式、基于发布订阅的消息系统。
    2、broker：kafka服务器，负责消息的存储和转发
    3、topic：消息类别，kafka按照topic来分类消息
    4、offset：消息在日志中的位置，可以理解是消息在partition上的偏移量，也是代表该消息的唯一序号
    5、producer：消息生产者
    6、consumer：消息消费者
    7、consumerGroup：消费者组，每个conseumer必须属于一个group
    8、Zookper：保存着集群broker、topic、partition等meta数据，另外，还负责broker故障发现，partitionleader选举，负载均衡功能
![](/image/kafka1.png)
**2、kafka维护消息状态跟踪方法**  

    1、大部分是维护消息状态，broker已经发送，或是consumer已处理后，但是都会有相应的问题
    2、kafka通过offset一个简单的整数，标记消息消费的位置，通过调整offset的值可以去消费一个较老的值
**3、数据传输的事务定义有哪三种**

    1、最多一次
    2、最少一次（并对消息设置唯一主键进行区分避免重复消费）
    3、就一次
**4、ack机制**  

    1、0：生产者不等待broker的ack，延迟低但是安全性差
    2、1：服务端等待ack的值，leader副本确认接收到消息后，发送ack
    3、-1：服务端等所有follwer的副本收到数据后才会收到leader发出的ack，保证数据不丢失
**5、手动提交偏移量**    
    
    将 auto.commit.offset 设为 false，然后在处理一批消息后 commitSync() 或者异步提交 commitAsync()  
**6、如何保证消息不被重复消费**     
     
    幂等性：用户对一个请求发起多次结果是一样的
    保证幂等：通过对消息设置唯一id进行判断再决定是否消费
**7、rabbitMQ**

    
    
    

              
                
           
       
   

      

    
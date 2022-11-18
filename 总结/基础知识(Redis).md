#第二期
##基础
###1、reids持久化一
rdb:

    1、触发，执行bgsave
    2、redis进程fork子进程进行持久化，等持久化完成替换就得rdb文件
    3、是经过压缩的二进制文件，占用的空间比较小
    4、在恢复大的数据集时，速度比aof快
    save 10 6000 ：设置10秒内6000次操作则持久化数据
aof:

    1、aof如果没有初始化，如果存在就读取
    2、redis接收指令，每个指令添加aof文件，当文件大小达到一定大小时rewrite
    或者手动调用bgRewriteaOf
    3、fork出一个子进程，而父进程继续接受命令，现在所写的指令放在aof_rewrite_buf_block缓冲中
    4、当子进程rewrite结束后，父进程收到子进程的信号，将缓冲中的内容添加到文件中，切换aof文件的fd。（先写临时文件最后rename替换）
    5、规则配置
      auto-aof-rewrite-percentage 100 设置文件超过最小基准值的xx百分比那就重写，在这里也就是超过64mb的100%就发生重写，也就是到达128mb就重写。
    6、使用redis-check-aof，对原来的文件进行修复  
   ![](/image/redis4.png) 
     
###2、redis跳表
是一种有序的数据结构，通过每个节点指向其他节点的指针，打到快速访问节点的目的
![跳表](/image/skipList.png)
###3、bitmap类型
由0和1状态表现的二进制数组(常用于统计)，二进制安全。
setbit key offset value

    （1）、使用userid和时间作为key，日期作为offset
     eg（统计每月签到）：setbit sign:userid:202101 18 1
     获得某一天的状态：getbit key offset
     bitcount key：统计userid在202101签到次数
    （2）、strlen：统计字节长度超过8位加1
    （3）、bitfield（连续签到）：实现连续签到多少天： 
    签到方式：setbit（userid+date，日期减一，true）
    bitfield k3 get u3（u+数量） 0(从第0位开始)
    问题：人名做key所以随着用户的增加压力变大
    （4）、bitop（做统计使用）：
                AND: 与运算符（&） 两个同时为1，结果为1，否则为0  
                OR:  或运算（|） 一个为1，其值为1
                NOT: 取反(0110 0001 NOT: 1001 1110)
                XOR: 异或运算,值不同为1，否则为0
          实现连续登录，活跃度统计
    签到方式：时间作为key，用户名作为偏移量  
###4、redis 缓存穿透，缓存击穿，缓存雪崩
        
    （1）缓存穿透：用户查询数据，在redis中查不到，然后去数据库查也查不到
        解决：
        a、布隆过滤器
        思路：当一个查询请求过来时，先通过布隆过滤器进行查，如是判断存在再继续查询操作，如是判断不存在，那肯定是不存
        在直接滚犊子
        原理：在一个bit数组中，采用多个hash函数计算元素位置，不同的元素可能hash值相同，判断存在的时候有一定的误差，
        在数组中不存在的一定不存在
        实际操作：redission
![](/image/redis1.png) 
                        
        b、缓存空对象
        (1)、空对象过多
        (2)、空对象过期了还是同样为题  
    （2）、缓存击穿：数据量太大，缓存过期
          解决：将热点数据设置为不过期，不设置过期时间
               加锁：对同一个key加分布式锁，锁的压力会特别大
    （3）、缓存雪崩：大量的热点数据失效，或者redis宕机
          解决：
          a、做成高可用redis
          b、限流降级，在缓存失效的时候，通过锁限制访问数据数据，
          c、通过预热，对热点数据设置不同过期时间
###5、redis过期策略和内存淘汰机制

    (1)、过期策略：定期删除和惰性删除
        a、定期删除：在一定的时间内（100ms），随机抽取key如果过期则删除（不是所有）25%
        b、惰性删除：又过期的key不会直接删除，再次获取key的时候进行判断
        依然存在问题
    (2)、内存淘汰： 分为两类，一是设置了过期时间的，二是所有的
        a、设置了过期时间的：
        volatile-LRU：设置了超时时长里最近使用最少的
        volatile-ttl:设置了超时时长里将要过期的
        volatile-random:从设置了超时时长的数据中随机
        allkeys-lru：从所有数据集中选择最近使用最少的
        allkeys-random：从所有数据集中随机选择
        no-enviction：不删除，再添加则报错
    (3)、设置最大清除内存：maxmemory
         设置淘汰机制：maxmemory-policy
         设置统计采样数：maxmemory-samples在LRU的基础下
![](/image/redis2.png)
###6、redis基本类型及基本操作
    String
    （1）、set： set key value ex（key过期时间s） px（key的过期时间ms） nx （不存在赋值） xx（存在赋值）
    （2）、append： append key value 给key的值添加vallue
    （3）、incr、incrby、incrbyfloat：计算添加1、给定值、给定浮点类型
    （4）、getrange、setrange：范围内取，范围内覆盖
    （5）、strLen key： 长度 
    map
    （1）、hset/hget/hmset/hmget/hgetall： hset key field value 
    （2）、hkeys/hvals: hkeys key
    （3）、hincrby/hincrbyflaot： hincrby key filed int
    （4）、hexists:hexists key filed 是否存在
    （5）、hdel：hdel key field [field] 删除一个或多个属性
    list
    （1）、lpush/rpush/lpop/rpop:lpush key value 正反向实现队列/栈
    （2）、blpop、brpop：blpop key timeout 阻塞单播队列，fifo
    （3）、lindex/lset ：Lindex key index /lset key index value 模拟数组操作
    （4）、linsert/ltrim:linsert key before/after value 在第一个出现的元素之前之后插数
          ltrim key start stop ；删除规定范围内的值
    （5）、lrem：lrem key count（-1、0、1） value 
          -1的时候从后面删除给定的数的值，1前面，0，所有
    set
    （1）、sadd/srem/scard/smembers: sadd key member 添加，移除,统计,查询
    （2）、sdiff/sdiffstore/sinter/sinterstore/sunion/sunionstore:
           差集存储/交集存储/并集存储
    （3）、srandmember： srandmember key count （抽奖）
            count正数，不会重复可能不够，负数一定满足数量，但是有可能重复
    zset
    （1）、zadd：zadd key[nx:xx] score member 添加元素
    （2）、zcard：zcard key 统计key的数量
    （3）、zcount：zcount min max 查询范围内的数量
    （4）、zincrby：zincrby key num member 给元素增加评分
    （5）、zinterstore/zunionstore：zinterstore  newkey key的数量 key1 key2
     weights 权重1 权重2 aggregate sum
    （6）、zrange/zrevrange： zrange key start end 正向反向查询（排行榜）
    pub/sub
    （1）、subscribe/publish:subscribe topic /publish topic info订阅/推送
    （2）、psubscribe：psubscribe 规则 订阅符合条件的多个
    
   ![](/image/redis3.png)
###7、redis事务
    
    1、multi：开启事务
    2、exec：执行事务 如果开启了两个事务，则谁先exec 先执行谁
    3、discard：取消事务
    4、watch：cas实现乐观锁，watch可以监视key，如果至少有一个被监视的键被修改
    事务取消  
###8、磁盘持久化二
    
    1、都开启时默认使用appendonly.aof进行数据恢复
    2、备份redis数据:
    （1）、在运行时随时可以对RBD文件进行复制：RDB文件一旦被创建就不会
    被修改，当创建新的RDB文件时，现将文件的内容保存到临时文件夹，当写入完毕时程序才
    使用rename（2）原子的用临时文件替换原有文件
    （2）、创建定时任务按所需时间将rdb文件备份到文件夹
    （3）、每天至少次，将rdb备份到数据中心机器外的机器上，保证容错性
###9、redis调用for（）函数进行持久化原理

    1、redis的work（工作线程）是单线程的，如果将持久化也放在这个工作线程，一旦持久化大量数据的时候
    将会影响用户使用，因此单开子进程专门做持久化
    2、redis内存中的数据，其实是redis中虚拟地址指向实际物理地址，这时要持久化，就是将真实数据的物理地址复制
![](/image/redis5.png)    
    
    3、将真实数据的物理地址copy一份到持久化空间
![](/image/redis6.png)
    
    4、当用户进行数据修改时，父进程中的虚拟内存映射到物理内存中的新的位置，不会影响子进程进行持久化
![](/image/redis7.png)

    5、原理：linux中父子进程
        （1）、读时共享，写时复制，降低内存消耗、减少复制所使用的时间
###10、redis持久化二
    1、dump.rdb
    优点：体积小，大数据集恢复快
    缺点：不安全，不能拉链就一个
    触发：save，bgsave，配置文件条件达到，主从复制，flushALL,shutdown
    2、appendonly.aof
    优点：数据安全，体积过大时可以重写
    缺点：相同的数据集aof文件体积大
    触发：bgrewrite，auto-aof-rewrite-min-size 64mb,auto-aof-rewrite-percentage 100
    appendfsync选项
    always： 每执行一个事件就把aof缓冲区的内容强制性的写入硬盘上的aof文件里，保证了数据持久化的完整性，
    
    everysec：每隔一秒才会进行一次文件同步，把内存缓冲区里的aof缓存数据真正写入aof文件，极端情况下会丢失一秒数据
    
    no：默认系统的缓存区写入磁盘机制，不做程序强制，安全性和完整性相对差一点
    
    3、redis4.0之前rewrite就是合并重复指令，删除无效指令 
       redis4.0之后新型混合持久化：对已有的数据进行rdb处理 ，后续继续补充指令
    4、数据恢复，只有某一种，就按照某一种，同时存在两种则按aof恢复
###11、redis小札
    1、一个String能存512mb
    
     
       
   
         

    




               
                
           
       
   

      

    

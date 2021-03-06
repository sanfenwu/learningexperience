#第二期
##基础
###1、reids持久化
rdb:

    1、触发，执行bgsave
    2、redis进程fork子进程进行持久化，等持久化完成替换就得rdb文件
    3、是经过压缩的二进制文件，占用的空间比较小
    4、在恢复大的数据集时，速度比aof快
aof:

    1、aof如果没有初始化，如果存在就读取
    2、redis接收指令，每个指令添加aof文件，当文件大小达到一定大小时rewrite
    或者手动调用bgRewriteaOf
    3、fork出一个子进程，而父进程继续接受命令，现在所写的指令放在aof_rewrite_buf_block缓冲中
    4、当子进程rewrite结束后，父进程收到子进程的信号，将缓冲中的内容添加到文件中，切换aof文件的fd。（先写临时文件最后rename替换）
###2、redis跳表
是一种有序的数据结构，通过每个节点指向其他节点的指针，打到快速访问节点的目的
![跳表](/image/skipList.png)
###3、bitmap类型
由0和1状态表现的二进制数组。常用与统计
setbit key offset value

    （1）、使用userid和时间作为key，日期作为offset
     eg（统计每月签到）：setbit sign:userid:202101 18 1
     获得某一天的状态：getbit key offset
     bitcount key：统计userid在202101签到次数
    （2）、strlen：统计字节长度超过8位加1
    （3）、bitfield（连续签到）：实现连续签到多少天： 
    签到方式：setbit（userid+date，日期减一，true）
    bitfield k3 get u3（u+数量） 0(从第0位开始)
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
        a、定期删除：在一定的时间内（100ms），随机抽取key如果过期则删除（不是所有）
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
         
   
         

    




               
                
           
       
   

      

    
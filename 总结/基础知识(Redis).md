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
    4、当子进程rewrite结束后，父进程今收到子进程的信号，将缓冲中的内容添加到文件中，切换aof文件的fd。（先写临时文件最后rename替换）
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
        

    




               
                
           
       
   

      

    
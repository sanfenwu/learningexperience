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

    eg（统计每月签到）：setbit sign:userid:202101 18 1




               
                
           
       
   

      

    
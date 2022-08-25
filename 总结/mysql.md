#第九期
##mysql
**1、mysql内部逻辑图**

![](/image/mysql1.png)  
      
        （1）、连接器：和客户端建立连接，获取权限、维持管理连接
        （2）、查询缓存(默认关闭)：key（sql语句）-value，先在缓存中查没有在查数据库，但是条件苛刻，8.0已经没了
        （3）、解析器/分析器：主要对要执行的sql语句进行语法分析，获得抽象语法树，在判断语法树中的表和字段是否存在        
        （4）、优化器：主要将sql经过解析得到的语法树，分析得到最优的执行计划
        （5）、执行器：根据执行计划，调用存储引擎提供的api操作数据，完成sql的执行
**2、mysql指令**
    
    （1）、set autocommit=0设置自动提交
    （2）、set Max_Execution_time命令，设置最长的执行时间
**3、索引覆盖、最左原则、索引下推**
      
      针对联合索引
    （1）、将需要查询出来的字段作为聚合索引，索引覆盖从而减去回表过程，业务优化减少执行时间
    （2）、通过最左原则，可以减少索引的维护，聚合索引的最左字段，普通索引的最左字符，聚合索引不支持最左
          复合索引，如果某一个列没出现或者进行范围查询，此列及后面的所有索引失效，索引的底层是b+tree
![](/image/mysql2.png)
          
    （3）、索引下推（5.6），在聚合索引的时候会对依次根据索引进行过滤减少回表次数
            版本之前，先按前面索引查出来然后回表查出所有，再根据后面的索引条件判断符合要求的数据
            版本之后，在根据前面索引查询的时候，会根据后面索引条件进行过滤，减少回表行数
          
**4、索引重建**

    alter table 表明 engine=innoDB；
**5、操作语言**

    1.DDL（DataDefinitionLanguage）：数据定义语言，用来定义数据库对象：库、表、列等；
    -- 【alter table add column】  新增一列
    alter table t_user add column t_birthday datetime null comment '用户生日';
    
    -- 【alter table modify column】 修改字段的属性
    alter table t_user modify column t_birthday datetime not null comment '用户生日';
   
    -- 【alter table change column newcolumn】 修改字段的属性或名字
    alter table t_user change  t_birthday  t_birth datetime not null comment '用户生日';
   
    -- 【alter table drop column】  删除字段
    alter table t_user drop t_birth;
  
    -- 【show create table】  
    show  create table t_user;
   
    -- 【rename table to newrtable】
    rename table t_user to my_user;
    rename table my_user to t_user;
  
    2.DML（DataManipulationLanguage）：数据操作语言，用来定义数据库记录（数据）；
    
    3.DQL（DataQueryLanguage）：数据查询语言，用来查询记录（数据）；
    
    4.DCL（DataControlLanguage）：数据控制语言，用来定义访问权限和安全级别。
**6、全局锁加锁**      

    （1）、对整个数据库加锁：Flush table with read lock 
    （2）、场景：做全库逻辑备份
    （3）、mysqldump工具备份 设置-single transaction
    （4）、设置全库只读：set global readonly=true
**7、表级锁**   
    
    （1）、lock tables ... read/write;unlock tables 主动释放锁此语法除了限制其他线程
    还会限制自己线程
    （2）、MDL（metadata lock）， 新增字段时，会等当前表的事务提交后在释放锁，所以会阻塞后续操作
        1：将事务做掉
        2：设置等待时间里能拿到锁则修改，拿不到也不阻塞后续操作，然后重试此操作
**8、行级锁**

    （1）、解决死锁：innodb_lock_wait_timeout来设置超时时间
          主动死锁检查：innodb_deadlock_detect 设置为on开启逻辑，发生死锁会先回滚一个事务
**9、索引选择**
    
    （1）、索引不准确，可以用analyze table 统计分析索引信息
          force index 强行指定索引
    字符串索引选择
     （1）、完整索引，比较占用空间
     （2）、前缀索引，节省空间，会增加扫描次数，不能使用覆盖索引
     （3）、倒序+前缀 不支持范围查找
     （4）、hash字段索引，查询性能稳定，有额外的存储和计算消耗，不支持范围扫描
**10、重建表后空间没有变小**     

    alter table 表明 engine =innodb
    （1）、刚做了重建，并且此时是onlineDDl 还有其他的操作进来，所以变大了
    （2）、每次重建都不会吧数据页填满，会预留1/16，有可能这部分被占了，然后重建表反而变大了
**11、count（*）**   
    
    先新增，在修改，并发情况下，减少锁等待
**12、索引字段做函数操作，索引失效**
    1、索引字段函数计算
    
    （1）、B+树提供快速定位的能力，来源于同一层兄弟节点的有序性，
    （2）、对索引字段做函数操作，可能会破坏索引值的有序性，因此优化器放弃走树搜索的功能
    
   2、隐世类型转换
   
    （1）、mysql中字符串和数字作比较的话，是将字符串转化成数字，对于优化器来说，相当于执行了函数转换
   3、隐式字符编码转换
    
    （1）、 当两个不同编码格式的表做关联查询，如果有字段的交互时，会默认进行编码的函数转换
**13、mysql操作指令**

    1、show processList 查看状态
**14、查询慢**
  1、被锁阻塞
  
    1、等待MDL写锁：查询语句被MDL阻塞
    2、等flush：flush语句被其他sql语句阻塞，我们的查询语句被flush阻塞
    3、等待行锁
  2、慢查询
    
    1、快照读的时候，检索undo log
**15、间隙锁分析**
    
    原则：
    （1）、加锁的基本单位是next-key lock ，前开后闭
    （2）、查找过程中访问到的对象才会加锁（以此分析针对唯一字段，普通字段作为条件查询时，加锁范围）
    优化：
    （1）、索引上等值查询，给唯一索引加锁的时候，next-key lock退化为行锁
    （2）、索引上等值查询，向右遍历时最后一个值不满足等值条件的时候，退化为间隙锁
**16、数据可靠性**    

    binlog:一个事务的binlog是不能分开的，系统会给每一个事务分配一个binlog cache 内存，每个线程一个
            通过binlog_cache_size来控制大小，先把binlog日志写在这个binlog_cache里面，如果超过这个大小先暂存磁盘
            事务提交的时候，执行器吧binlog_cache中的binlog写入到binlog中，并清空binlog_cache.
    sync_binlog=0/1/n:每次提交事务都会write，不fsync/每次都fsync/每次都write在经过n在fsync（类比aof）
**17、count（\*）和count（1）**

    count(*),经过优化器优化取行数，
    count（1），也是取行数并赋个1；
    实际上效率是差不多的
   

          


               
                
           
       
   

      

    
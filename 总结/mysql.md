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
    
    （1）、将需要查询出来的字段作为聚合索引，索引覆盖从而减去回表过程，业务优化减少执行时间
    （2）、通过最左原则，可以减少索引的维护
    （3）、索引下推，在聚合索引的时候会对依次根据索引进行过滤减少回表次数
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
    
    
    
    

          


               
                
           
       
   

      

    
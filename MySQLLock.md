
# MySQL锁机制
## 参考文档： https://juejin.im/post/5b82e0196fb9a019f47d1823  https://mp.weixin.qq.com/s/WgcLdEfjFE3CsWV4FgOefw

# 锁的级别

    MyISAM存储主要就简单的表级别锁
    
    InnoDB不仅支持行级别的锁，也支持表级别的锁, 默认为行级锁

## 1. 表级锁：
    
    开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高 ，并发度最低。
    
    锁的是整张表，限制的是整张表的数据读写。
    
## 2. 行级锁：
    
    开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。
    
    锁的是表中对应的行，只限制当前行的读写
    
    具体的锁：
        
        共享锁（S锁或者读锁）和排它锁（X锁或者写锁）
### 使用行级锁的条件
    
    行级锁都是基于索引的，如果一条SQL语句用不到索引是不会使用行级锁的，会使用表级锁。     
     
## 3. 页面锁：
    
    开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。
    
    锁的是整张表中某页数据，限制的是某页的数据读写。
    

# 乐观锁和悲观锁

    乐观锁和悲观锁是泛指，不是具体的锁。
    
    表锁和行锁也是泛指，不是具体的锁
    
## 1. 乐悲锁
    
    乐观并发控制，它总是乐观的认为用户在并发事务处理时不会影响彼此的数据。
    
### 乐观锁的实现：    
    
    用数据版本（Version）记录机制实现，即为数据增加一个版本标识，一般是通过为数据库表增加一个数字类型的 “version” 字段来实现。
    当读取数据时，将version字段的值一同读出，数据每更新一次，对此version值加1。当我们提交更新的时候，
    判断数据库表对应记录的当前版本信息与第一次取出来的version值进行比对，如果数据库表当前版本号与第一次取出来的version值相等
    ，则予以更新，否则认为是过期数据
    
    每次更新表中的value字段时，为了防止发生冲突，需要这样操作
        update TABLE  set value=xx,version=version+1  where id=id and version=version


    
## 2. 悲观锁

    认为每次去拿的数据都会被别人修改。所以每次在拿数据的时候都会上锁，从而屏蔽可能违反数据完整性的操作。
    
    悲观锁需要耗费较多的时间。另外与乐观锁相对应的，悲观锁是由数据库自己实现了的，要用的时候，我们直接调用数据库的相关语句就可以了。
    
    间隙锁、临键锁都属于悲观锁。
    
### 悲观锁的实现
    
    悲观锁的实现：首先实现悲观锁时，我们必须先使用



    set autocommit=0; 关闭mysql的autoCommit属性。
    
    
    
    因为我们查询出数据之后就要将该数据锁定。
    
    
    
    关闭自动提交后，我们需要手动开启事务。
    
    //1.开始事务
    
    begin; 或者 start transaction;
    
    //2.查询出商品信息，然后通过for update锁定数据防止其他事务修改
    
    select status from t_goods where id=1 for update;
    
    //3.根据商品信息生成订单
    
    insert into t_orders (id,goods_id) values (null,1);
    
    //4.修改商品status为2
    
    update t_goods set status=2;
    
    //5.提交事务
    
    commit; --执行完毕，提交事务
    
    
    
    上述就实现了悲观锁，悲观锁就是悲观主义者，它会认为我们在事务A中操作数据1的时候，一定会有事务B来修改数据1,所以，在第2步我们将数据查询出来后直接加上排它锁（X）锁，防止别的事务来修改事务1，直到我们commit后，才释放了排它锁。
    
    
    
    优点：保证了数据处理时的安全性。
    
    
    
    缺点：加锁造成了开销增加，并且增加了死锁的机会。降低了并发性。



# 共享锁(S锁)和排他锁(X锁)
    
    InnoDB引擎实现了标准的行级别锁：分别是共享锁和排他锁。
    
    对于普通 SELECT 语句，InnoDB 不会加任何锁；
    对于 UPDATE、 DELETE 和 INSERT 语句  InnoDB会自动给涉及数据集加排他锁（X)；
    
## 共享锁
    
    共享锁又称读锁 (read lock)，是读取操作创建的锁。其他用户可以并发读取数据，但任何事务都不能对数据进行修改，
    直到已释放所有共享锁。如果事务对读锁进行修改操作，很可能会造成死锁
    
    加上共享锁后，对于update，insert，delete语句会自动加排它锁
    
   ![avatar](https://user-gold-cdn.xitu.io/2018/8/31/1658f05aa3240058?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
    
### 使用说明
    
    若事务A对数据对象1加上S锁，则事务A可以读数据对象1但不能修改，其他事务只能再对数据对象1加S锁，而不能加X锁，
    直到事务A释放数据对象1上的S锁。这保证了其他事务可以读数据对象1，但在事务A释放数据对象1上的S锁之前不能对数据对象1做任何修改。 
    如果当前事务需要对该记录进行更新操作，则很有可能造成死锁。
    
### 用法：

    select ... lock in share mode;
    
    设置共享锁：select * from user where id = 1 LOCK IN SHARE MODE;
    
    ----共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改

## 排他锁
    
    排他锁（也叫writer lock）又称写锁，独占锁。排它锁会阻塞所有的排它锁和共享锁
    
   ![avatar](https://user-gold-cdn.xitu.io/2018/8/31/1658f053ad8d212e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
    
### 使用说明：
    
    若事务A对数据对象1加上X锁，事务A可以读数据对象1也可以修改数据对象1，其他事务不能再对数据对象1加任何锁，
    直到事务A释放数据对象1上的锁。
    
### 用法：
    
    select ... for update
    
    设置排他锁：select * from user where id = 1 FOR UPDATE;
    
    ----排他锁就是不能与其他所并存，如一个事务获取了一个数据行的排他锁，其他事务就不能再获取该行的其他锁
    其他 事务 可以查询该记录，但是不能对该记录加共享锁或排他锁，而是等待获得锁

# 意向锁

    为了允许行锁和表锁共存，实现多粒度锁机制，InnoDB 还有两种内部使用的意向锁
    这两种意向锁都是表锁
    
    意向锁是有数据引擎自己维护的，用户无法手动操作意向锁，在为数据行加共享 / 排他锁之前，InooDB 会先获取该数据行所在在数据表的对应意向锁。
    意向锁不会与行级的共享 / 排他锁互斥！！！
    
## 意向共享锁（IS）
    
    事务打算给数据行加共享锁, 必须先在表上加意向共享锁
    
    -- 事务要获取某些行的 S 锁，必须先获得表的 IS 锁。
    SELECT column FROM table ... LOCK IN SHARE MODE;

## 意向排他锁（IX)
    
    事务打算给数据行加排他锁,必须先在表上加意向排他锁
    
    -- 事务要获取某些行的 X 锁，必须先获得表的 IX 锁。
    SELECT column FROM table ... FOR UPDATE;
    
## 意向锁的兼容互斥性

                    意向共享锁（IS）       	意向排他锁（IX）
    意向共享锁（IS）	    兼容	                    兼容
    意向排他锁（IX）	    兼容	                    兼容
    
    意向锁之间是互相兼容的
    
    意向锁和自家兄弟互相兼容，但是它会与普通的排他 / 共享锁互斥
    
    	            意向共享锁（IS）	            意向排他锁（IX）
    共享锁（S）	        兼容	                    互斥
    排他锁（X）	        互斥	                    互斥
    
    这里的排他 / 共享锁指的都是表锁！！！意向锁不会与行级的共享 / 排他锁互斥！！！

## 意向锁的作用

    如果另一个任务试图在该表级别上应用共享或排它锁，则受到由第一个任务控制的表级别意向锁的阻塞。第二个任务在锁定该表前不必检查各个页或行锁，
    而只需检查表上的意向锁
    
    例：
        事务 A 获取了某一行的排他锁，并未提交
            SELECT * FROM users WHERE id = 6 FOR UPDATE;
            
        此时 users 表存在两把锁：users 表上的意向排他锁与 id 为 6 的数据行上的排他锁
        
        事务 B 想要获取 users 表的共享锁：
            LOCK TABLES users READ;
            
        此时事务 B 检测事务 A 持有 users 表的意向排他锁，就可以得知事务 A 必然持有该表中某些数据行的排他锁，
        那么事务 B 对 users 表的加锁请求就会被排斥（阻塞），而无需去检测表中的每一行数据是否存在排他锁。
        
        事务 C 也想获取 users 表中某一行的排他锁
            
            SELECT * FROM users WHERE id = 5 FOR UPDATE;
            
            1. 事务 C 申请 users 表的意向排他锁。
            2. 事务 C 检测到事务 A 持有 users 表的意向排他锁。为意向锁之间并不互斥，所以事务 C 获取到了 users 表的意向排他锁。
            3. 因为id 为 5 的数据行上不存在任何排他锁，最终事务 C 成功获取到了该数据行上的排他锁。


# 间隙锁
    
    用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，
    叫做“间隙（GAP)”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key锁）
    
    InnoDB除了通过范围条件加锁时使用间隙锁外，如果使用相等条件请求给一个不存在的记录加锁，InnoDB也会使用间隙锁！
    
## 示例：
    
    假如emp表中只有101条记录，其empid的值分别是 1,2,...,100,101，下面的SQL：
    
    Select * from  emp where empid > 100 for update;
    
    InnoDB不仅会对符合条件的empid值为101的记录加锁，也会对empid大于101（这些记录并不存在）的“间隙”加锁
    

## InnoDB使用间隙锁的目的

    1. 防止幻读，以满足相关隔离级别的要求；
    
    2. 满足恢复和复制的需要：
        
        MySQL 通过 BINLOG 录入执行成功的 INSERT、UPDATE、DELETE 等更新数据的 SQL 语句，并由此实现 MySQL 数据库的恢复和主从复制。
        
        MySQL 的恢复机制（复制其实就是在 Slave Mysql 不断做基于 BINLOG 的恢复）有以下特点：

            一是 MySQL 的恢复是 SQL 语句级的，也就是重新执行 BINLOG 中的 SQL 语句。
            
            二是 MySQL 的 Binlog 是按照事务提交的先后顺序记录的， 恢复也是按这个顺序进行的。

        由此可见，MySQL 的恢复机制要求：在一个事务未提交前，其他并发事务不能插入满足其锁定条件的任何记录，也就是不允许出现幻读。


# 表级别加锁

## MyISAM表级锁模式：
    
    共享读锁 （Table Read Lock）：不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；
    
    独占写锁 （Table Write Lock）：会阻塞其他用户对同一表的读和写操作；
        会阻塞其他进程对同一表的读和写操作，只有当写锁释放后，才会执行其他进程的读写操作。
        
    MyISAM 在执行查询语句(SELECT)前,会自动给涉及的所有表加读锁,
    在执行更新操作 (UPDATE、DELETE、INSERT 等)前，会自动给涉及的表加写锁，
    
    这个过程并不需要用户干预，因此，用户一般不需要直接用LOCK TABLE命令给MyISAM表显式加锁。
    
  ![avatar](https://user-gold-cdn.xitu.io/2018/8/31/1658f07bf9dbb8bf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
  ![avatar](https://user-gold-cdn.xitu.io/2018/8/31/1658f07ff7213d75?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 对表加读锁和写锁

    InnoDB表和MyISAM都可以
    
    1. 对表test_table增加读锁：       
        
        LOCK TABLES test_table READ 
    
    2. 对表test_table增加写锁：
        
        LOCK TABLES test_table WRITE
    
    3. 解锁表：
        UNLOCK TABLES 
 
## LOCK TABLES 注意项
    
    当使用 LOCK TABLES 时，不仅需要一次锁定用到的所有表,而且,同一个表在 SQL 语句中出现多少次，就要通过与 SQL 语句中相同的别名锁定多少次，否则也会出错！
    
    比如如下SQL语句：
        select a.first_name,b.first_name, from actor a,actor b where a.first_name = b.first_name;
    
    该Sql语句中，actor表以别名的方式出现了两次，分别是a,b，这时如果要在该Sql执行之前加锁就要使用以下Sql:
        lock table actor as a read,actor as b read;
        
## 使用LOCK TABLES的场景：

    第一种情况：全表更新。事务需要更新大部分或全部数据，且表又比较大。若使用行锁，会导致事务执行效率低，从而可能造成其他事务长时间锁等待和更多的锁冲突
    
    第二种情况：多表查询。事务涉及多个表

    例如，有一个订单表 orders，其中记录有各订单的总金额 total，同时还有一个 订单明细表 order_detail，其中记录有各订单每一产品的金额小计 subtotal，假设我们需要检 查这两个表的金额合计是否相符，可能就需要执行如下两条 SQL：
    
        Select sum(total) from orders; 
        Select sum(subtotal) from order_detail; 
    
    这时，如果不先给两个表加锁，就可能产生错误的结果，因为第一条语句执行过程中，
    order_detail 表可能已经发生了改变。因此，正确的方法应该是：
    
        Lock tables orders read local, order_detail read local; 
        Select sum(total) from orders; 
        Select sum(subtotal) from order_detail; 
        Unlock tables;
                
## MyISAM的锁调度
    
    MyISAM 存储引擎的读锁和写锁是互斥的，读写操作是串行的。那么，一个进程请求某个 MyISAM 表的读锁，同时另一个进程也请求同一表的写锁，MySQL 如何处理呢?
    
    写进程先获得锁
    
    即使读请求先到锁等待队列，写请求后到，写锁也会插到读锁请求之前！这是因为 MySQL 认为写请求一般比读请求要重要。
    这也正是 MyISAM 表不太适合于有大量更新操作和查询操作应用的原因，因为大量的更新操作会造成查询操作很难获得读锁，从而可能永远阻塞
 
##  MyISAM 表不会出现死锁
    
    在自动加锁的情况下，MyISAM 总是一次获得 SQL 语句所需要的全部锁，这也正是 MyISAM 表不会出现死锁（Deadlock Free）的原因。
    

    
# 死锁
    
    在两个或多个并发进程中因争抢锁资源而造成的互相等待的现象
    
    
## 产生死锁的四个必要条件：

    互斥条件：
        
        一个资源每次只能被一个进程使用。
   
    请求与保持条件：
        
        一个进程因请求资源而阻塞时，对已获得的资源保持不放。
    
    不剥夺条件:
        
        进程已获得的资源，在末使用完之前，不能强行剥夺。
    
    循环等待条件:
        
        若干进程之间形成一种头尾相接的循环等待资源关系。
    
## 示例：
    
    T1:begin transelect * from table lock in share mode update table set column1='hello'
			
	T2:begin transelect * from table lock in share mode update table set column1='world'
    
    假设 T1 和 T2 同时达到 select，T1 对 table 加共享锁，T2 也对 table 加共享锁当 T1 的 select 执行完，准备执行 update 时，根据锁机制，T1 的共享锁需要升级到排他锁才能执行接下来的 update.在升级排他锁前，
	必须等 table 上的其它共享锁（T2）释放，同理，T2 也在等 T1 的共享锁释放。于是死锁产生了。


## 死锁的解决

    1.等待事务超时，主动回滚。
    
    2.进行死锁检查，主动回滚某条事务，让别的事务能继续走下去。
        
        查看当前的事务
            SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;
        查看当前锁定的事务
            SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;
        查看当前等锁的事务
            SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
        杀死进程
            kill 进程ID
            
            

# 锁的释放
    
    非事务(Transaction) 中，语句执行完毕，便释放锁。
    
    事务 (Transaction) 中，只有等到当前的事务Transaction 进行了 commit 或 roll back，锁才能释放。

   
    InnoDB引擎 默认的修改数据类SQL语句，update,delete,insert等，都会自动给涉及到的数据加上排他锁


# 锁的优化

## 查看加锁情况

    1表示加锁，0表示未加锁。
    
    mysql> show open tables where in_use > 0;
    +----------+-------------+--------+-------------+
    | Database | Table       | In_use | Name_locked |
    +----------+-------------+--------+-------------+
    | lock     | myisam_lock |      1 |           0 |
    +----------+-------------+--------+-------------+


## 查询表级锁争用情况：
    
    
    mysql> show status like 'table_locks%';
    +----------------------------+-------+
    | Variable_name              | Value |
    +----------------------------+-------+
    | Table_locks_immediate      | 104   |
    | Table_locks_waited         | 0     |
    +----------------------------+-------+


    通过检查 table_locks_waited 和 table_locks_immediate 状态变量来分析系统上的表锁的争夺，
    如果 Table_locks_waited 的值比较高，则说明存在着较严重的表级锁争用情况
    
    
## 获取 InnoDB 行锁争用情况：
    可以通过检查 InnoDB_row_lock 状态变量来分析系统上的行锁的争夺情况：
    
    mysql> show status like 'innodb_row_lock%'; 
    +-------------------------------+-------+ 
    | Variable_name | Value | 
    +-------------------------------+-------+ 
    | InnoDB_row_lock_current_waits | 0 | 
    | InnoDB_row_lock_time | 0 | 
    | InnoDB_row_lock_time_avg | 0 | 
    | InnoDB_row_lock_time_max | 0 | 
    | InnoDB_row_lock_waits | 0 | 
    +-------------------------------+-------+ 
    
    innodb_row_lock_current_waits: 当前正在等待锁定的数量
    innodb_row_lock_time: 从系统启动到现在锁定总时间长度；非常重要的参数，
    innodb_row_lock_time_avg: 每次等待所花平均时间；非常重要的参数，
    innodb_row_lock_time_max: 从系统启动到现在等待最常的一次所花的时间；
    innodb_row_lock_waits: 系统启动后到现在总共等待的次数；非常重要的参数。直接决定优化的方向和策略。
    
## 行锁优化

    1. 尽可能让所有数据检索都通过索引来完成，避免无索引行或索引失效导致行锁升级为表锁。
    2. 尽可能避免间隙锁带来的性能下降，减少或使用合理的检索范围。
    3. 尽可能减少事务的粒度，比如控制事务大小，而从减少锁定资源量和时间长度，从而减少锁的竞争等，提供性能。
    4. 尽可能低级别事务隔离，隔离级别越高，并发的处理能力越低。

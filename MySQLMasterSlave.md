# MySQL主从复制
## 1. 主从复制的原理

    Mysql的主从复制中主要有三个线程：master（binlog dump thread）、slave（I/O thread 、SQL thread），Master一条线程和Slave中的两条线程。
    
    总的来说就是主mysql的DML(数据操作语言)和DDL(数据定义语言)在从mysql中执行一遍。 

    过程：
            1.  主mysql打开bin-log文件,打开后就会把主mysql执行过的所有DML,DDL记录到bin-log文件中，
    
                Master有多少个slave就会创建多少个binlog dump线程， Master节点的binlog发生变化时，binlog dump 线程会
                通知所有的salve节点，并将相应的binlog内容推送给slave节点
            
            2.   从mysql的 I/O thread线程在Slave中创建，该线程用于请求Master，Master会返回binlog的名称以及当前数据更新的位置、
                 binlog文件位置的副本。然后，将binlog保存在 「relay log（中继日志）」 中，中继日志也是记录数据更新的信息。
            
            3.   SQL线程也是在Slave中创建的，当Slave检测到中继日志有更新，根据DML,DDL， 执行相同的SQL语句， 这样就保证了主从的数据的同步
            
## 2. 策略方式

    1. 「同步策略」：Master会等待所有的Slave都回应后才会提交，这个主从的同步的性能会严重的影响。
    
    2. 「半同步策略」：Master至少会等待一个Slave回应后提交。
    
    3. 「异步策略」：Master不用等待Slave回应就可以提交。
    
    4. 「延迟策略」：Slave要落后于Master指定的时间
    
  
## 3. Mysql主从的优点


    1. 高性能方面：主从复制通过水平扩展的方式，解决了原来单点故障的问题，并且原来的并发都集中到了一台Mysql服务器中，
       现在将单点负载分散到了多台机器上，实现读写分离，不会因为写操作过长锁表而导致读服务不能进行的问题，提高了服务器的整体性能。

    2. 可靠性方面：主从在对外提供服务的时候，若是主库挂了，会有通过主从切换，选择其中的一台Slave作为Master；
       若是Slave挂了，还有其它的Slave提供读服务，提高了系统的可靠性和稳定性。


## 4. 若是主从复制，达到了写性能的瓶颈如何处理：
    
    设计上进行解决采用分库分表的形式，对于业务数据比较大的数据库可以采用分表，使得数据表的存储的数据量达到一个合理的状态
    
    
 ## 5. 主从复制的过程有数据延迟怎么办？导致Slave被读取到的数据并不是最新数据：
 
     查看同步延时时间：
        
        通过 MySQL 命令：show status   查看 Seconds_Behind_Master
 
     参考文档： https://mp.weixin.qq.com/s?__biz=Mzg3MjA4MTExMw==&mid=2247498199&idx=2&sn=3083e59355aacdb1683f038e40764913
     
     缓存路由：
     
     1. 写请求发往主库，同时缓存记录操作的 key，缓存的失效时间设置为主从的延时
    
     2. 读请求首先判断缓存是否存在
        
        若存在，代表刚发生过写操作，读请求操作主库
        
        若不存在，代表近期没发生写操作，读请求操作从库
        
# MySQL主从配置：

   参考文档：https://zhuanlan.zhihu.com/p/199217698
 
 

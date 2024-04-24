# InnoDB

### 一.简介

InnoDB是目前MySQL的默认存储引擎，MyISAM是MySQL早期的默认存储引擎。

InnoDB引擎与MyISAM引擎的区别：

* InnoDB引擎支持事务，MyISAM不支持。
* InnoDB引擎支持行锁和表锁，MyISAM仅支持表锁，不支持行锁。
* InnoDB引擎支持外键，MyISAM是不支持。

### 二.逻辑存储结构

逻辑存储结构从上到下：

* TableSpace（表空间），表空间中管理多个段。
* Segment（段），有数据段、索引段、回滚段。段管理多个区。
* Extend（区），每个区大小1M，区管理多个页。
* Page（页），每个页默认大小16K，页管理多个行。
* Row（行），数据按行存放。

### 三.架构图

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption><p>InnoDB架构图</p></figcaption></figure>

架构主要分为如下部分：

* 内存架构
  * Buffer Pool
  * Change Buffer
  * Adaptive Hash Index
  * Log Buffer
* 磁盘架构
  * System Tablespace
  * File-Per-Table Tablespaces
  * General Tablespaces
  * Undo Tablespaces
  * Temporary Tablespaces
  * Doublewrite Buffer Files
  * Redo Log

### 四.内存架构

#### 1.Buffer Pool

InnoDB的缓冲池中不仅缓存了索引页和数据页，还包含了undo页、插入缓存、自适应哈希索引以及 InnoDB的锁信息等。

执行增删改查操作时，先操作缓冲池中的数据（若缓冲池没有数据，则从磁盘加载并缓存），然后再以一定频率刷新到磁盘，从而减少磁盘IO，加快处理速度。

#### 2.Change Buffer

在执行DML语句时，如果这些数据Page没有在Buffer Pool中，不会直接操作磁盘，而会将数据变更存在更改缓冲区 Change Buffer 中，在未来数据被读取时，再将数据合并恢复到Buffer Pool中，再将合并后的数据刷新到磁盘中。

目的是为了减少磁盘IO。

#### 3.Adaptive Hash Index

自适应hash索引，用于优化对Buffer Pool数据的查询。

#### 4.Log Buffer

日志缓冲区，用来保存要写入到磁盘中的log日志数据（redo log 、undo log）。默认日志在每次事务提交时写入并刷新到磁盘。

### 五.磁盘架构

#### 1.System Tablespace

Change Buffer的存储区域。

#### 2.File-Per-Table Tablespaces

如果开启了innodb\_file\_per\_table开关（默认开启） ，则每个表的文件表空间包含单个InnoDB表的数据和索引 ，并存储在文件系统上的单个数据文件中。

#### 3.General Tablespaces

通用表空间，需要通过 CREATE TABLESPACE 语法创建通用表空间，在创建表时，可以指定该表空间。

#### 4.Undo Tablespaces

MySQL实例在初始化时会自动创建两个默认的undo表空间（初始大小16M），用于存储 undo log日志。

#### 5.Temporary Tablespaces

存储用户创建的临时表等数据。

#### 6.Doublewrite Buffer Files

双写缓冲区，InnoDB引擎将数据页从Buffer Pool刷新到磁盘前，先将数据页写入双写缓冲区文件中，便于系统异常时恢复数据。

#### 7.Redo Log

重做日志，是用来实现事务的持久性。

该日志文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件（redo log），前者是在内存中，后者在磁盘中。

当事务提交之后会把所有修改信息都会存到该日志中，用于在刷新脏页到磁盘时或者发生错误时，进行数据恢复。

### 六.后台线程

InnoDB的后台线程中，分为4类：

* Master Thread，核心后台线程，负责调度其他线程，负责将缓冲池中的数据异步刷新到磁盘中, 保持数据的一致性。
* IO Thread，一系列负责读写操作的IO线程，如刷新日志等。
* Purge Thread，回收事务已经提交了的undo log，在事务提交之后，undo log可能不用了，就用它来回 收。
* Page Cleaner Thread，协助 Master Thread 刷新脏页到磁盘的线程。

### 七.事务原理

#### 1.原子性

通过undo log，可以做到回滚数据，保证事务要么全部成功，要么全部失败。

#### 2.一致性

通过redo log和undo log，保证所有数据都是一致的。

#### 3.隔离性

通过锁和MVCC（Multi-Version Concurrency Control，多版本并发控制）来保证事务不受外部并发操作影响。

#### 4.持久性

通过redo log，能够恢复数据，保证事务的更改是永久的。

### 八.MVCC

全称 Multi-Version Concurrency Control，多版本并发控制。指维护一个数据的多个版本，使得读写操作没有冲突。

#### 1.读方式

MySQL有两种读方式：

* 当前读，读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。select ... lock in share mode(共享锁)，select ... for update、update、insert、delete(排他锁)都是一种当前读。
* 快照读，简单的select（不加锁）就是快照读，读取的是记录数据的可见版本，有可能是历史数据，快照读是非阻塞的。

注意：

* Read Committed：每次select，都生成一个快照读。
* Repeatable Read：开启事务后第一个select语句才是快照读的地方。
* Serializable：快照读会退化为当前读。

#### 2.隐藏字段

InnoDB还会自动的给表添加三个隐藏字段：

<table><thead><tr><th width="190">字段</th><th>含义</th></tr></thead><tbody><tr><td>DB_TRX_ID</td><td>最近修改事务ID，记录插入这条记录或最后一次修改该记录的事务ID。</td></tr><tr><td>DB_ROLL_PTR</td><td>回滚指针，指向这条记录的上一个版本，用于配合undo log，指向上一个版本。</td></tr><tr><td>DB_ROW_ID</td><td>隐藏主键，如果表结构没有指定主键，将会生成该隐藏字段。</td></tr></tbody></table>

借助DB\_TRX\_ID和DB\_ROLL\_PTR，不同事务或相同事务对同一条记录进行修改，会导致该记录的undolog生成一条`记录版本链表`，链表的头部是最新的旧记录，链表尾部是最早的旧记录。

### 3.ReadView

ReadView（读视图）是快照读SQL执行时MVCC提取数据的依据，记录并维护系统当前活跃的事务（未提交的）id。

ReadView中包含了四个核心字段：

<table><thead><tr><th width="217">字段</th><th>含义</th></tr></thead><tbody><tr><td>m_ids</td><td>当前活跃的事务ID集合。</td></tr><tr><td>min_trx_id</td><td>最小活跃事务ID。</td></tr><tr><td>max_trx_id</td><td>当前最大事务ID+1。</td></tr><tr><td>creator_trx_id</td><td>ReadView创建者的事务ID。</td></tr></tbody></table>

通过这些事务ID可以判断是否可以访问undo log里的某一个版本，原理是比较DB\_TRX\_ID和表中的字段。

不同的隔离级别，生成ReadView的时机不同：

* RC：在事务中每一次执行快照读时生成ReadView。
* RR：仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView。因为用了同一个ReadView，所以后续快照读的内容是一致的，避免了不可重复读的问题发生。

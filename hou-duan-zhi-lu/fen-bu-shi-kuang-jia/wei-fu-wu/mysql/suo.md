# 锁

### 一.分类

MySQL中的锁，按照锁的粒度分，分为以下三类：&#x20;

* 全局锁：锁定数据库中的所有表。&#x20;
* 表级锁：每次操作锁住整张表。&#x20;
* 行级锁：每次操作锁住对应的行数据。

### 二.全局锁

全局锁会阻塞所有DDL、DML语句，但是可以执行DQL数据。

### 三.表级锁

对于表级锁，主要分为以下三类：

* 表锁
* 元数据锁（meta data lock，MDL）
* 意向锁

#### 1.表锁

表锁分为两类：&#x20;

* 表共享读锁（read lock）
* 表独占写锁（write lock）

读锁不会阻塞其他客户端的读，但是会阻塞写。写锁既会阻塞其他客户端的读，又会阻塞其他客户端的写。

语法如下：lock tables 表名... read/write;

```sql
lock tables 表名 read; # 读锁
lock tables 表名 write # 写锁

# 解锁，如果客户端断开连接也会解锁
unlock tables;
```

#### 2.元数据锁

MDL加锁过程是系统自动控制，无需显式使用，在访问一张表的时候会自动加上。

MDL锁主要作用是维护表元数据的数据一致性，在表上有活动事务的时候，不可以对元数据进行写入操作。

常见sql操作，添加的元数据锁如下：

| SQL                                          | 锁类型                                          | 说明                                            |
| -------------------------------------------- | -------------------------------------------- | --------------------------------------------- |
| lock tables xxx read/write                   | SHARED\_READ\_ONLY / SHARED\_NO\_READ\_WRITE |                                               |
| select ... lock in share mode、select         | SHARED\_READ                                 | 与SHARED\_READ、 SHARED\_WRITE兼容，与 EXCLUSIVE互斥。 |
| insert 、update、 delete、select ... for update | SHARED\_WRITE                                | 与SHARED\_READ、 SHARED\_WRITE兼容，与 EXCLUSIVE互斥。 |
| alter table ...                              | EXCLUSIVE                                    | 与其他的MDL都互斥。                                   |

可以看出来元数据锁可以避免DML与 DDL冲突，保证读写的正确性。

#### 3.意向锁

为了避免DML在执行时，加的行锁与表锁的冲突，在InnoDB中引入了意向锁，使得表锁不用检查每行数据是否加锁，使用意向锁来减少表锁的检查。

常见sql操作，添加的意向锁如下：

| SQL                                      | 锁类型       | 说明                                    |
| ---------------------------------------- | --------- | ------------------------------------- |
| select ... lock in share mode            | 意向共享锁(IS) | 表锁共享锁 (read)兼容，与表锁排他锁(write)互斥。       |
| insert、update、delete、select...for update | 意向排他锁(IX) | 与表锁共享锁(read)及排他锁(write)都互斥，意向锁之间不会互斥。 |

注意：

* 一旦事务提交了，意向共享锁、意向排他锁，都会自动释放。

### 四.行级锁

InnoDB的数据是基于索引组织的，行锁是通过对索引上的索引项加锁来实现的，而不是对记录加锁。

对于行级锁，主要分为以下三类：

* 行锁（Record Lock）：锁定单个行记录的锁，防止其他事务对此行进行update和delete。在 RC、RR隔离级别下都支持。
* 间隙锁（Gap Lock）：锁定索引记录间隙（不含该记录），确保索引记录间隙不变，防止其他事务在这个间隙进行insert，产生幻读。在RR隔离级别下都支持。
* 临键锁（Next-Key Lock）：行锁和间隙锁组合，同时锁住数据，并锁住数据前面的间隙Gap。 在RR隔离级别下支持。

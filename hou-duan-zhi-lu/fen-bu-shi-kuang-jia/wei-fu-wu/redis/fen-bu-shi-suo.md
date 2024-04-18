# 分布式锁

### 一.简单分布式锁

下面命令实现了一个简单的分布式锁：

```
SET resource-name anystring NX EX max-lock-time
```

如果上述命令返回OK，说明获取了锁。如果返回NIL，则稍后重试。

注意：

* anystring应该使用一个随机的猜测不到的大字符串，称之为token。
* 不要使用DEL释放锁，而是使用脚本判断anystring相等才释放锁。
* 以上两点是防止一种情况：A设置了分布式锁lock且超时时间60s，在60s的时候A仍然没有执行完业务逻辑，此时lock超时删除。在61s的时候B重新设置了分布式锁lock，但是A此时业务逻辑执行完成，删除了lock。所以anystring既然随机，删除的时候也要判断这个anystring。

以下是删除脚本，使用方式为`EVAL ...script... 1 resource-name token-value`：

```
if redis.call("get",KEYS[1]) == ARGV[1]
then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

### 二.RedLock

RedLock是一个更规范的分布式锁算法：[https://redis.io/docs/latest/develop/use/patterns/distributed-locks/](https://redis.io/docs/latest/develop/use/patterns/distributed-locks/)

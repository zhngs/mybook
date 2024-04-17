# 通用命令

#### 1.KEYS

```
KEYS pattern
```

该命令返回redis中所有满足pattern的key，时间复杂度是O(n)，n是redis中所有key。

返回值：

* 所有匹配的key的list。

注意：

* 虽然时间复杂度是O(n)，但是执行速度很快。
* 在生产环境中不建议使用，有可能对大型数据库破坏性能。

```
redis> MSET firstname Jack lastname Stuntman age 35
"OK"
redis> KEYS *name*
1) "lastname"
2) "firstname"
redis> KEYS a??
1) "age"
redis> KEYS *
1) "age"
2) "lastname"
3) "firstname"
```

#### 2.EXISTS

```
EXISTS key [key ...]
```

该命令可用于判断key是否存在，时间复杂度是O(n)，n是EXISTS后key的个数。

返回值是指定的key中存在的key的个数。

注意：

* EXISTS后可以写两个相同的key，此时这个key会被计算两次。

```
redis> SET key1 "Hello"
"OK"
redis> EXISTS key1
(integer) 1
redis> EXISTS nosuchkey
(integer) 0
redis> SET key2 "World"
"OK"
redis> EXISTS key1 key2 nosuchkey
(integer) 2
```

#### 3.TYPE

```
TYPE key
```

该命令用来查询key的类型，时间复杂度是O(1)。

返回值是类型的字符串，如果不存在返回none。

```
redis> SET key1 "value"
"OK"
redis> LPUSH key2 "value"
(integer) 1
redis> SADD key3 "value"
(integer) 1
redis> TYPE key1
"string"
redis> TYPE key2
"list"
redis> TYPE key3
"set"
```

#### 3.DEL

```
DEL key [key ...]
```

该命令用来删除key，如果key类型是string，时间复杂度是O(n)，n是要删除的key的个数，如果key类型是list这种复合类型，时间复杂度是O(m)，m是list中值的个数。

返回值是删除成功的key的个数。

注意：

* 如果key不存在，会被忽略。

```
redis> SET key1 "Hello"
"OK"
redis> SET key2 "World"
"OK"
redis> DEL key1 key2 key3
(integer) 2
```

#### 3.UNLINK

```
UNLINK key [key ...]
```

该命令用来异步删除key，首先会在keyspace中将该key删除，然后实际的内存回收会在别的线程执行。时间复杂度是O(1)，实际的内存回收线程会有O(n)的时间复杂度，n是key中实际值的个数。

注意：

* 如果key不存在，会被忽略。
* UNLINK是非阻塞的，DEL是阻塞的。

#### 4.TTL

```
TTL key
```

该命令用来获取设置过超时的key的存活时间，时间级别是秒。时间复杂度是O(1)。

返回值：

* key存在且设置超时，返回秒级的生存时间。
* 如果key存在但是没有设置超时，返回-1。
* 如果key不存在，返回-2。

```
redis> SET mykey "Hello"
"OK"
redis> EXPIRE mykey 10
(integer) 1
redis> TTL mykey
(integer) 10
```

#### 5.EXPIRE

```
EXPIRE key seconds [NX | XX | GT | LT]
```

给key设置超时时间，时间复杂度是O(1)。设置过超时的key，被称之为`volatile`。如果成功设置超时，会返回1，如果没有成功设置超时，会返回0（比如key不存在）。

选项含义如下：

* `NX` ，只有当key没有超时的情况，才会设置超时。
* `XX` ，只有当key已经设置过超时，才会设置超时。
* `GT` ，只有当新的超时比key当前的超时大，才会设置超时。
* `LT` ，只有当新的超时比key当前的超时小，才会设置超时。
* 对于GT和LT说，非volatile的key，ttl会被认为是无限的。GT、LT和NX是互斥的。

注意：

* key过期之后，会被自动删除。
* 只有删除key或者覆盖key的值会清理超时，包括DEL、SET、GETSET和所有的存储命令。
* 改变在key上的value而不是用新值替换，不会清理超时。比如INCR自增，LPUSH向list增加新值，HSET改变哈希字段，都不会影响超时。
* 使用PERSIST命令可以清理超时。
* 如果使用RENAME重命名一个key，超时时间会转移到新的key上。如果key\_a原本存在，key\_b被RENAME成key\_a，那么新的key\_a，会继承key\_b所有特征，而旧的key\_a则等同于被删除。
* 如果指定过期时间是负数，会直接删除key。
* 对已经设置超时的key，再次EXPIRE会刷新key的存活时间。

redis超时原理：

* 被动超时：用户访问key的时候，超时key会被删除。
* redis每秒会执行10次主动超时：
  * 在所有设置超时的key中随机取20个。
  * 删除所有超时key。
  * 如果超时key的数量大于25%，则再随机取20个，重复上述步骤。

```
redis> SET mykey "Hello"
"OK"
redis> EXPIRE mykey 10
(integer) 1
redis> TTL mykey
(integer) 10
redis> SET mykey "Hello World"
"OK"
redis> TTL mykey
(integer) -1
redis> EXPIRE mykey 10 XX
(integer) 0
redis> TTL mykey
(integer) -1
redis> EXPIRE mykey 10 NX
(integer) 1
redis> TTL mykey
(integer) 10
```

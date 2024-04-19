# Hash

Hash是string类型的field和value的映射表，底层数据结构是哈希表。

### 1.HSET

```
HSET key field value [field value ...]
```

该命令设置key上field和value的映射表。如果key不存在会创建一个hash，如果key存在则覆盖指定的field和value。

返回值：

* 返回新添加的field的个数（integer）。

```
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HGET myhash field1
"Hello"
redis> HSET myhash field2 "Hi" field3 "World"
(integer) 2
redis> HGET myhash field2
"Hi"
redis> HGET myhash field3
"World"
redis> HGETALL myhash
1) "field1"
2) "Hello"
3) "field2"
4) "Hi"
5) "field3"
6) "World"
```

### 2.HGET

```
HGET key field
```

返回field上的value。

RESP2返回值：

* field上的value（bulk string）。
* 如果key不存在或者field不存在，返回Nil。

RESP3返回值：

* field上的value（bulk string）。
* 如果key不存在或者field不存在，返回Null。

```
redis> HSET myhash field1 "foo"
(integer) 1
redis> HGET myhash field1
"foo"
redis> HGET myhash field2
(nil)
```

### 3.HEXISTS

```
HEXISTS key field
```

判断field是否存在。

返回值：

* 如果key不存在或者field不存在，返回0。
* 如果field存在返回1。

```
redis> HSET myhash field1 "foo"
(integer) 1
redis> HEXISTS myhash field1
(integer) 1
redis> HEXISTS myhash field2
(integer) 0
```

### 4.HKEYS

```
HKEYS key
```

返回hash中所有的field。

返回值：

* field列表（array）。

注意：

* HVALS返回所有value，其他地方和HKEYS相同。

```
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HSET myhash field2 "World"
(integer) 1
redis> HKEYS myhash
1) "field1"
2) "field2"
```

### 5.HINCRBY

```
HINCRBY key field increment
```

给field上的数字加increment，可以是负数。

返回值：

* field的value。

```
redis> HSET myhash field 5
(integer) 1
redis> HINCRBY myhash field 1
(integer) 6
redis> HINCRBY myhash field -1
(integer) 5
redis> HINCRBY myhash field -10
(integer) -5
```

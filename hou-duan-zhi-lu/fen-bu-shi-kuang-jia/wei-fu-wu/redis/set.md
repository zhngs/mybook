# Set

Set是无序集合，可以自动去重，底层数据结构是value为null的Hash。

### 1.SADD

```
SADD key member [member ...]
```

向Set中添加成员。如果key不存在则自动创建Set。如果member已经存在则忽略。

返回值：

* 返回新添加成员的数量（integer）。

```
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SADD myset "World"
(integer) 0
redis> SMEMBERS myset
1) "Hello"
2) "World"
```

### 2.SMEMBERS

```
SMEMBERS key
```

返回key上Set的所有成员。

返回值：

* Set上的成员列表（array）。

```
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SADD myset "World"
(integer) 0
redis> SMEMBERS myset
1) "Hello"
2) "World"
```

### 3.SISMEMBER

```
SISMEMBER key member
```

判断Set里是否有member。

返回值：

* 如果key不存在或者没有member，返回0。
* 如果member存在返回1。

```
redis> SADD myset "one"
(integer) 1
redis> SISMEMBER myset "one"
(integer) 1
redis> SISMEMBER myset "two"
(integer) 0
```

### 4.SCARD

```
SCARD key
```

返回Set中成员的数量。

返回值：

* 如果key不存在，返回0。
* 返回key上Set的成员数量。

```
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SCARD myset
(integer) 2
```

### 5.SREM

```
SREM key member [member ...]
```

删除Set中特定的member。如果member不存在则忽略，如果key不存在也忽略。

返回值：

* 返回删除的member的数量（integer）。

```
redis> SADD myset "one"
(integer) 1
redis> SADD myset "two"
(integer) 1
redis> SADD myset "three"
(integer) 1
redis> SREM myset "one"
(integer) 1
redis> SREM myset "four"
(integer) 0
redis> SMEMBERS myset
1) "two"
2) "three"
```

### 6.SPOP

```
SPOP key [count]
```

删除并返回最多count个随机member。

RESP2返回值：

* 如果key不存在返回Nil。
* 如果没有指定count，返回被删除的member（bulk string）。
* 如果指定了count，返回删除的列表（array）。

RESP3返回值：

* 如果key不存在返回Null。
* 如果没有指定count，返回被删除的member（bulk string）。
* 如果指定了count，返回删除的列表（array）。

```
redis> SADD myset "one"
(integer) 1
redis> SADD myset "two"
(integer) 1
redis> SADD myset "three"
(integer) 1
redis> SPOP myset
"three"
redis> SMEMBERS myset
1) "one"
2) "two"
redis> SADD myset "four"
(integer) 1
redis> SADD myset "five"
(integer) 1
redis> SPOP myset 3
1) "two"
2) "four"
3) "five"
redis> SMEMBERS myset
1) "one"
```

### 7.SRANDMEMBER

```
SRANDMEMBER key [count]
```

返回count个member，但不删除。

RESP2返回值：

* 如果不指定count，随机返回一个member（bulk string）。
* 如果不指定count，且key不存在，返回Nil。
* 如果count为正数，返回不重复member列表，至多count个。
* 如果count为负数，返回重复member列表，即一个member可以被选多次，数量是count的绝对值。
* 如果指定了count，但是count不存在，返回空列表（array）。

RESP3返回值：

* 如果不指定count，随机返回一个member（bulk string）。
* 如果不指定count，且key不存在，返回Null。
* 如果count为正数，返回不重复member列表，至多count个。
* 如果count为负数，返回重复member列表，即一个member可以被选多次，数量是count的绝对值。
* 如果指定了count，但是count不存在，返回空列表（array）。

```
redis> SADD myset one two three
(integer) 3
redis> SRANDMEMBER myset
"one"
redis> SRANDMEMBER myset 2
1) "one"
2) "two"
redis> SRANDMEMBER myset -5
1) "two"
2) "three"
3) "one"
4) "three"
5) "two"
```

### 8.SMOVE

```
SMOVE source destination member
```

将source中的member移动到destination中。

返回值：

* 如果member成功被move，返回1。
* 如果source中没有member或者source不存在，则什么都不会发生，返回0。

```
redis> SADD myset "one"
(integer) 1
redis> SADD myset "two"
(integer) 1
redis> SADD myotherset "three"
(integer) 1
redis> SMOVE myset myotherset "two"
(integer) 1
redis> SMEMBERS myset
1) "one"
redis> SMEMBERS myotherset
1) "three"
2) "two"
```

### 9.SINTER

```
SINTER key [key ...]
```

返回多个Set的交集。如果key不存在，则认为是空Set。

返回值：

* 交集列表（array）。

注意：

* SUNION返回并集，SDIFF返回第一个Set和后面Set的差集。

```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SINTER key1 key2
1) "c"
```


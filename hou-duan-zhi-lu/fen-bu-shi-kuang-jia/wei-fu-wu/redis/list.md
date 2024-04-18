# List

List底层是双向链表。

### 1.LPUSH

```
LPUSH key element [element ...]
```

在列表头部插入数据，如果key不存在，会首先创建一个空List再执行插入操作。时间复杂度是O(n)，n是要插入元素的个数。

返回值：

* List的长度（integer）。

注意：

* RPUSH和LPUSH类似，只是RPUSH是在列表尾部插入数据。

```
redis> LPUSH mylist "world"
(integer) 1
redis> LPUSH mylist "hello"
(integer) 2
redis> LRANGE mylist 0 -1
1) "hello"
2) "world"
```

### 2.LPOP

```
LPOP key [count]
```

删除List头部count个元素，count默认为1，如果count大于List中元素个数，count等同于List长度。时间复杂度是O(n)，n是实际删除的元素个数。

RESP2返回值：

* 如果key不存在，返回Nil。
* 如果不带count，返回删除的value的值（bulk string）。
* 如果带了count，返回删除的value列表（array）。

RESP3返回值：

* 如果key不存在，返回Null。
* 如果不带count，返回删除的value的值（bulk string）。
* 如果带了count，返回删除的value列表（array）。

注意：

* RPOP和LPOP类似，只是RPOP是在列表尾部删除数据。

```
redis> RPUSH mylist "one" "two" "three" "four" "five"
(integer) 5
redis> LPOP mylist
"one"
redis> LPOP mylist 2
1) "two"
2) "three"
redis> LRANGE mylist 0 -1
1) "four"
2) "five"
```

### 3.LRANGE

```
LRANGE key start stop
```

返回List中start和stop之间的元素。

返回值：

* List中start和stop范围内元素组成的List（array）。
* 如果key不存在或者start超出范围，返回空List。

注意：

* 0表示第一个元素，1表示第二个元素，-1表示最后一个元素，-2表示倒数第二个元素，以此类推。
* 0到10的元素个数是11个，即start和stop上的元素都是包含的，在使用这个特性时，最好在特定语言的库中确认一下。
* start和stop超出List范围不会报错。如果start超过List最后一个元素，会返回空List。如果stop超过List最后一个元素，会将其当作最后一个元素看待。

```
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LRANGE mylist 0 0
1) "one"
redis> LRANGE mylist -3 2
1) "one"
2) "two"
3) "three"
redis> LRANGE mylist -100 100
1) "one"
2) "two"
3) "three"
redis> LRANGE mylist 5 10
(empty array)
```

### 4.LINDEX

```
LINDEX key index
```

返回链表中下标为index的值。时间复杂度是O(n)，n是找到index需要的步数。

RESP2返回值：

* 返回链表中下标为index的值（bulk string）。
* 如果index超出范围，返回Nil。

RESP3返回值：

* 返回链表中下标为index的值（bulk string）。
* 如果index超出范围，返回Null。

```
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSH mylist "Hello"
(integer) 2
redis> LINDEX mylist 0
"Hello"
redis> LINDEX mylist -1
"World"
redis> LINDEX mylist 3
(nil)
```

### 5.LLEN

```
LLEN key
```

返回列表的长度，时间复杂度是O(1)。

返回值：

* 返回列表长度（integer）。
* 如果key不存在，返回0。

```
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSH mylist "Hello"
(integer) 2
redis> LLEN mylist
(integer) 2
```

### 6.LINSERT

```
LINSERT key <BEFORE | AFTER> pivot element
```

向pivot值的前或后插入element。

返回值：

* 如果成功插入，返回list长度（integer）。
* 如果key不存在，返回0。
* 如果pivot没找到，返回-1。

```
redis> RPUSH mylist "Hello"
(integer) 1
redis> RPUSH mylist "World"
(integer) 2
redis> LINSERT mylist BEFORE "World" "There"
(integer) 3
redis> LRANGE mylist 0 -1
1) "Hello"
2) "There"
3) "World"
```

### 7.LSET

```
LSET key index element
```

将index上的值设置为element。

返回值：

* 成功设置返回OK（simple string）
* 超出范围返回错误。

```
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LSET mylist 0 "four"
"OK"
redis> LSET mylist -2 "five"
"OK"
redis> LRANGE mylist 0 -1
1) "four"
2) "five"
3) "three"
```

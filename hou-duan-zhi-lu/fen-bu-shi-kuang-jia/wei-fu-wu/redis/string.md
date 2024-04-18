# String

#### 1.SET

```
SET key value [NX | XX] [GET] [EX seconds | PX milliseconds | EXAT unix-time-seconds | PXAT unix-time-milliseconds | KEEPTTL]
```

该命令给key设置字符串值，时间复杂度是O(1)。

RESP2返回值：

* 如果GET不存在，且设置成功，返回OK（simple string）。
* 如果GET不存在，且和NX或XX冲突，导致SET失败，返回Nil。
* 如果GET存在，且key之前不存在，返回Nil。
* 如果GET存在，且key之前存在，返回key之前的value值（bulk string）。
* 如果GET存在，且key之前存在，但是旧key类型不是string，则返回错误，且SET无效。

RESP3返回值：

* 如果GET不存在，且设置成功，返回OK（simple string）。
* 如果GET不存在，且和NX或XX冲突，导致SET失败，返回Null。
* 如果GET存在，且key之前不存在，返回Null。
* 如果GET存在，且key之前存在，返回key之前的value值（bulk string）。
* 如果GET存在，且key之前存在，但是旧key类型不是string，则返回错误，且SET无效。

注意：

* 即使key已经存在，不管key是什么类型，SET都会将其覆盖。
* SET会清除key上的超时时间。

选项：

* NX，只有key不存在的时候才设置。
* XX，只有key存在的时候才设置。
* EX设置秒级超时时间，PX毫秒级，EXAT秒级unix时间，PXAT毫秒级unix时间。
* KEEPTTL，保留和key关联的超时时间。
* GET，如果key存在，返回旧的value值。如果key不存在，返回nil。如果key存在但是value不是string，那么会返回错误并且SET无效。

例子：

```
redis> SET mykey "Hello"
"OK"
redis> GET mykey
"Hello"
redis> SET anotherkey "will expire in a minute" EX 60
"OK"
```

#### 2.GET

```
GET key
```

返回key存储的字符串value，时间复杂度是O(1)。

RESP2返回值：

* 如果key存在，返回key上存储的值（bulk string）。
* 如果key不存在，返回Nil。
* 如果key的类型不是string，返回错误。

RESP3返回值：

* 如果key存在，返回key上存储的值（bulk string）。
* 如果key不存在，返回Null。
* 如果key的类型不是string，返回错误。

#### 3.APPEND

```
APPEND key value
```

如果key存在且类型是string，会在原字符串上追加value。如果key不存在，APPEND等同于SET。时间复杂度是O(1)。

返回值：

* 返回追加后的字符串长度（integer）。

```
redis> EXISTS mykey
(integer) 0
redis> APPEND mykey "Hello"
(integer) 5
redis> APPEND mykey " World"
(integer) 11
redis> GET mykey
"Hello World"
```

#### 4.STRLEN

```
STRLEN key
```

返回key上string的长度，时间复杂度是O(1)。

返回值：

* key存在，返回key上string的长度（integer）。
* key不存在，返回0（integer）。
* key不是string类型，返回错误。

```
redis> SET mykey "Hello world"
"OK"
redis> STRLEN mykey
(integer) 11
redis> STRLEN nonexisting
(integer) 0
```

#### 5.INCR

```
INCR key
```

使key上可以转换成数字的字符串加1。如果key不存在，会首先执行`SET key 0`，再执行INCR。时间复杂度是O(1)。

返回值：

* 返回key上的整数（integer）。
* 如果key的类型不是string或者key上的值不能转成字符串，返回错误。

注意：

* key上数字的范围是64位有符号数字。在redis中，使用字符串表示整数没有额外开销。
* INCR可以用来做计数器和限频器。

```
redis> SET mykey "10"
"OK"
redis> INCR mykey
(integer) 11
redis> GET mykey
"11"
```

#### 6.DECR

使key上可以转换成数字的字符串减1。如果key不存在，会首先执行`SET key 0`，再执行DECR。时间复杂度是O(1)。

```
redis> SET mykey "10"
"OK"
redis> DECR mykey
(integer) 9
redis> SET mykey "234293482390480948029348230948"
"OK"
redis> DECR mykey
(error) value is not an integer or out of range
```

返回值：

* 返回key上的整数（integer）。
* 如果key的类型不是string或者key上的值不能转成字符串，返回错误。

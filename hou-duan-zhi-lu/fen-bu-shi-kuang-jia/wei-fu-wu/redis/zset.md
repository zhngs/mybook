# Zset

Zset是一个没有重复元素的字符串集合，每个元素都有一个评分，用来排序。Zset底层是哈希表和跳表。

### 1.ZADD

```
ZADD key [NX | XX] [GT | LT] [CH] [INCR] score member [score member   ...]
```

添加多个指定分数的member。如果member已经存在，则score会被更新，并保证插入到正确的位置。如果key不存在，则看作是空集合，然后进行操作。

RESP2返回值：

* 如果NX、XX、GT、LT选项没有满足，返回Nil。
* 如果没有指定CH，返回新增加的元素个数（integer）。
* 如果指定了CH，返回新增加的和更新的元素个数（integer）。
* 如果指定了INCR，返回member的score（bulk string）。

RESP3返回值：

* 如果NX、XX、GT、LT选项没有满足，返回Null。
* 如果没有指定CH，返回新增加的元素个数（integer）。
* 如果指定了CH，返回新增加的和更新的元素个数（integer）。
* 如果指定了INCR，返回member的score（double）。

选项：

* XX只更新存在的元素，NX只添加新元素。
* GT只更新新score比旧score更大的元素，LT只更新新score比旧core更小的元素。这个标志不影响增加新元素。
* CH如果不指定，返回值是新增加的元素个数。如果指定，返回值是新增加的元素和更改的元素个数和。
* 当指定了INCR，类似于ZINCRBY，此时只能有一个score、number对。

注意：

* score是双精度浮点数，`+inf` 和 `-inf`也是有效值。
* GT、LT和NX是互斥的。

```
(integer) 1
redis> ZADD myzset 2 "two" 3 "three"
(integer) 2
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "one"
2) "1"
3) "uno"
4) "1"
5) "two"
6) "2"
7) "three"
8) "3"
```

### 2.ZRANGE

```
ZRANGE key start stop [BYSCORE | BYLEX] [REV] [LIMIT offset count]   [WITHSCORES]
```

返回特定范围内的元素。ZRANGE可以按照下标，分数，字典序来遍历。Zset中元素的顺序是按照score从小到大进行排序，如果score相同则按照字典序。

返回值：

* 满足条件的元素列表（array）。

选项：

* 默认情况是按照元素下标进行查找，比如0代表第一个元素，1代表第二个，-1代表倒数第一个，-2代表倒数第二个。start或者stop超出范围不会出错，start超出范围会返回空列表，stop超出范围则使用最后一个元素下标替代stop。
* 指定BYSCORE后，使用score查找元素，范围是\[start, stop]，注意这里是闭区间，如果想要使用开区间，则在start或者stop前加上左小括号即可，如`ZRANGE zset (5 (10 BYSCORE`。可以使用`-inf`和`+inf`来表示无穷大。
* 指定BYLEX后，需要注意必须满足所有元素的score都一样，如果score不一样则结果未知。start和stop必须以左小括号`(`或左中括号`[`开头，表示是否包含。可以使用加号`+`来表示最大的字符串，减号`-`表示最小的字符串。
* REV选项会将排序集完全反过来，最高分在第一位，最低分在最后一位。此时如果指定了BYSCORE或BYLEX，start必须大于等于stop才能保证结果不为空列表。
* LIMIT选项类似于SQL中select limit offset count。offset如果是负数，则认为是全部元素。
* WITHSCORES选项会在返回值中将score也返回。

注意：

* 从6.2版本开始，ZREVRANGE、ZRANGEBYSCORE、ZREVRANGEBYSCORE、ZRANGEBYLEX、ZREVRANGEBYLEX可以被ZRANGE替代。

```
redis> ZADD myzset 1 "one" 2 "two" 3 "three"
(integer) 3
redis> ZRANGE myzset 0 -1
1) "one"
2) "two"
3) "three"
redis> ZRANGE myzset 2 3
1) "three"
redis> ZRANGE myzset -2 -1
1) "two"
2) "three"
```

### 3.ZINCRBY

```
ZINCRBY key increment member
```

增加member上的分数，增量是increment，increment是一个双精度浮点字符串，可以是负数。如果member不存在，则创建且认为socre是0。如果key不存在，则创建一个Zset。

RESP2返回值：

* 增加后member的socre（bulk string）。

RESP3返回值：

* 增加后member的socre（double）。

```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZINCRBY myzset 2 "one"
"3"
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "two"
2) "2"
3) "one"
4) "3"
```

### 4.ZREM

```
ZREM key member [member ...]
```

删除member成员，如果member不存在，则忽略。

返回值：

* 删除的成员的数量（integer）。

```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZREM myzset "two"
(integer) 1
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "one"
2) "1"
3) "three"
4) "3"
```

### 5.ZCOUNT

```
ZCOUNT key min max
```

返回socre在min和max之间的元素数量。

返回值：

* 符合范围的元素数量（integer）。

```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZCOUNT myzset -inf +inf
(integer) 3
redis> ZCOUNT myzset (1 3
(integer) 2
```

### 6.ZRANK

```
ZRANK key member [WITHSCORE]
```

返回member在Zset中的index。

选项：

* WITHSCORE选项使返回值额外多一个score。

RESP2返回值：

* 如果key不存在或者member不存在，返回Nil。
* 如果WITHSCORE没指定，返回member的index（integer）。
* 如果指定了WITHSCORE，返回member的index和score（array）。

RESP3返回值：

* 如果key不存在或者member不存在，返回Null。
* 如果WITHSCORE没指定，返回member的index（integer）。
* 如果指定了WITHSCORE，返回member的index和score（array）。

```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZRANK myzset "three"
(integer) 2
redis> ZRANK myzset "four"
(nil)
redis> ZRANK myzset "three" WITHSCORE
1) (integer) 2
2) "3"
redis> ZRANK myzset "four" WITHSCORE
(nil)
```

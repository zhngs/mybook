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

# 概述

### 一.用途

Redis可以用做数据库，缓存，流引擎和消息代理等，所有数据都存储在内存中，可以支持海量并发读写。

### 二.常用数据类型

Redis中常用数据类型如下：

* String
* List
* Set
* Hash
* Zset

### 三.其他

* Transaction，事务。
* Stream，消息队列。
* Script，lua脚本。
* Function，Redis7.0新功能，类似lua脚本。
* Pub/Sub，发布订阅。
* HyperLogLog，基数估算，估算在一批数据中，不重复元素的个数有多少。
* Geospatial，存储地理位置。
* Bitmap，位图，连续的二进制数组。
* Bloom Filter，布隆过滤器，判断某个值是否存在。
* Cuckoo Filter，布谷鸟过滤器，解决了Bloom Filter不能删除和存在误判的问题。
* Count-Min Sketch，计算某个值出现多少次。
* Top-k，最频繁使用的k个值是什么。
* t-digest，百分位数算法。
* JSON，支持操作json。
* FT，全文搜索。
* TS，时间序列。

# 最佳实践

### 一.proto

* 不要重用field number。
* 将删除的field number设置为reserved，建议将name也设置为reserved，防止重用。
* 将删除的枚举值设置为reserved，建议将name也设置为reserved，防止重用。
* 不要改变字段类型。
* 不要使用required字段。
* message中field不要超过100个，因为每一个field在代码中都会占内存。
* 枚举中应该将第一个值定义为未知字段，比如命名为`FOO_UNSPECIFIED`。
* 不要将c/c++的宏常量赋值给枚举。
* 使用更广为人知的类型（[https://protobuf.dev/programming-guides/dos-donts/#well-known-common](https://protobuf.dev/programming-guides/dos-donts/#well-known-common)）
* 在单独的文件里定义广泛使用的类型。
* 不要改变field默认值，有可能造成客户端和服务的不一致。
* 不要把repeated变成标量类型，会丢失数据。
* 遵守风格指南。
* 不要使用文本格式的消息进行交换。
* 不要将名字定义成语言关键字。

### 二.api

* 为message和field写准确、简洁的文档。
* 对网络传输和存储在硬盘上的proto使用不同的message，主要目的是将两者隔离，方便后续的修改。
* 对于更改，支持部分更改和追加更改，而不是全部替换。
* 不要在顶层的request或response中包含标量类型，因为可能随时会改变。
* 不要使用bool表示以后可能会出现超过两种值的字段。
* 使用字符串来表示id，而不是整数类型。
* 不要让客户端将消息按照特定格式编码到string中或者从string中解析。
* 对于一些不透明的字段，比如token，应该使用web安全的编码序列化后，写入到string中。
* 不要包含client用不到的字段，会增加理解成本。
* 将相关联的字段定义成新消息，让字段高聚合。
* 在读请求中包含FieldMask。
* 包含version字段允许一致性读取。
* 对于相同返回值的RPC，使用同一个请求。
* 一个请求可能有多个操作，然后会有局部故障，可以考虑将RPC返回置为失败，在status中描述成功和失败的详细信息。
* 服务端提供操作小型数据的方法，期望客户端通过组合这些方法来达到批处理的效果。
* 对于客户端需要两次串行往返的RPC，最好整合成单次RPC。
* repeated的字段最好是message，而不是标量或枚举。
* api的设计应该具有幂等性，简单的方式是客户端提供一个id，根据这个id判断是否是相同请求。
* 让service名保持唯一。
* 确保每个RPC有超时时间。
* 限定请求和响应的大小。
* 小心传播状态码，错误往往比无用更糟糕，因为错误会造成混乱。
* 为每个RPC方法创建独立的请求和响应proto。

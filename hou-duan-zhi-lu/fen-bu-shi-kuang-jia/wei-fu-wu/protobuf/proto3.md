---
description: 官方文档：https://protobuf.dev/overview/
---

# proto3

### 一.简介

protobuf是语言无关、平台无关、可拓展的序列化结构数据的机制。protobuf可以提取写好proto文件，使通信更加规范，并且支持后向兼容。

### 二.语法

下面是一个简单的proto文件：

<pre class="language-protobuf"><code class="lang-protobuf"><strong>package foo.bar; // 可选的包说明符，防止消息命名冲突
</strong><strong>syntax = "proto3"; // 不写默认是proto2
</strong>
// 使用message定义一个结构
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
}
</code></pre>

这里需要注意，每一个具体的字段都有如下的属性：

* 可选性：使用optional关键字指定。
* 重复性：使用repeated关键字指定，默认不是重复的。
* 字段类型：包括标量类型、枚举类型、复合类型。
* 字段号：每一个字段都会被打上一个固定的字段号，message里的字段号不能重复，且范围是`1` 到 `536,870,911`，一旦字段号定义了，就不允许再更改，不然会出现兼容性问题。`19,000` 到 `19,999`是预分配给协议内部实现用的，自己用编译器会报错。

### 三.类型

#### 1.标量类型

不同语言在protobuf中对应的标量表见：[https://protobuf.dev/programming-guides/proto3/#scalar](https://protobuf.dev/programming-guides/proto3/#scalar)

#### 2.枚举类型

如下是一个定义枚举的例子：

```protobuf
enum Corpus {
  CORPUS_UNSPECIFIED = 0;
  CORPUS_UNIVERSAL = 1;
  CORPUS_WEB = 2;
  CORPUS_IMAGES = 3;
  CORPUS_LOCAL = 4;
  CORPUS_NEWS = 5;
  CORPUS_PRODUCTS = 6;
  CORPUS_VIDEO = 7;
}

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
  Corpus corpus = 4;
}
```

枚举这里需要注意：

* 第一个枚举值必须是0，这里是为了考虑默认值。
* 枚举值必须在32位整数内，因为枚举是变长编码，所以不建议使用负数，因为比较低效。

#### 3.消息类型

message就是定义消息类型，类似json中的对象，并且message支持嵌套写法。

```protobuf
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

#### 4.重复类型

repeated类似于json中的列表，代表将该值重复零次或多次。

```protobuf
message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

#### 5.oneof类型

oneof类型类似c++中的union，主要是为了节省内存。其中最多只能有一个字段被设置，后面设置的字段会覆盖前面设置的字段。

```protobuf
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

#### 6.map类型

map算是一个快捷的提供key和value关联的语法。

```protobuf
map<key_type, value_type> map_field = N;
```

key\_type是除了浮点类型和bytes类型意外任何的标量，value\_type是除了map的任何一个类型。

map后向兼容如下的定义：

```protobuf
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}
repeated MapFieldEntry map_field = N;
```

map有如下特性：

* map不能是repeated的。
* map的元素是乱序的。
* map在生成文本消息时是按照key排序的。

### 四.默认值

不同类型有自己的默认值：

* string的默认值是空字符串。
* bytes的默认值是空bytes。
* bool的默认值是false。
* number类型的标量默认值是0。
* 枚举类型的默认值是第一个枚举值，并且这个值必须是0。
* repeated类型的值默认为空，在具体语言中通常是空列表。

### 五.import

protobuf支持导入其他的proto文件：

```protobuf
import "myproject/other_protos.proto";
```

编译器使用 `-I`/`--proto_path`来查找proto文件，如果没有设置该标志，则默认在编译器执行的目录下找。推荐将路径设置项目的根路径。

proto3可以导入proto2的文件，但是不能使用proto2的枚举。

proto2同样可以导入proto3文件。

### 六.最佳实践

* 不要更改任何字段号，否则会造成协议不兼容。
* 添加新字段后，当新服务收到旧协议，旧协议中没有的字段会被当成默认值，需要处理好默认值引发的行文，不要让默认值造成不必要的问题。当旧服务收到新协议，会将相关字段当成未知字段。
* 如果确定某个字段不再被使用，可以删除该字段。你可以重命名该字段，比如加上"OBSOLETE\_"前缀，或者将该字段设置为reserved。
* `int32`, `uint32`, `int64`, `uint64`, 和 `bool`是兼容的，这意味着这几种类型相互替代，不会造成兼容性问题。
* `sint32` 和 `sint64`兼容，但和其他数字类型不兼容。
* 如果`bytes`是有效的utf-8字符串，`string` 和 `bytes`兼容。
* 如果`bytes`是一个编码过的`message`，那么`bytes`和嵌套的`message`兼容。
* `fixed32` 和 `sfixed32`兼容。
* `fixed64` 和 `sfixed64`兼容。
* 对于`string`、`bytes`、`message`来说，`optional`和`repeated`是兼容的。兼容方式是对于基础字段，optional只取repeated的最后一个值；对于message字段，则合并所有字段，_此处对于如何合并暂不清楚_。
* `enum` 和 `int32`, `uint32`, `int64`和`uint64`在wire格式上兼容，但是要注意可能会出现截断情况，反序列化时的具体表现取决于语言。

### 七.service

service的作用是让你在RPC调用中可以使用定义的message，如grpc就使用了protobuf。如果不使用grpc，同样也可以在你自己定义的RPC中使用protobuf。

```protobuf
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

### 八.json映射

protobuf和json的映射见：[https://protobuf.dev/programming-guides/proto3/#json](https://protobuf.dev/programming-guides/proto3/#json)

有几点需要注意：

* json转protobuf时，如果字段为空，则解释为默认值。
* protobuf转json时，如果字段为默认值，则默认不会输出该字段到json中。

### 九.option

option是对proto文件中一些声明的额外注释，option有文件级别的，也有字段级别的，同样可以自定义option。


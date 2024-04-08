# 编码格式

### 一.简介

编码主要是讲protobuf在网络上发送的格式是什么，以及占据磁盘的空间有多少。

### 二.格式

protobuf的编码是一系列的tag-value对，tag是field number和wire type，value是payload。wire type可以告诉解码器payload有多大，从而将不同的key-value对分开。

下表是不同wire type对应的ID和适用于编码哪些数据类型：

<table><thead><tr><th width="148">ID</th><th width="188">Wire Type</th><th>Used For</th></tr></thead><tbody><tr><td>0</td><td>VARINT</td><td>int32, int64, uint32, uint64, sint32, sint64, bool, enum</td></tr><tr><td>1</td><td>I64</td><td>fixed64, sfixed64, double</td></tr><tr><td>2</td><td>LEN</td><td>string, bytes, embedded messages, packed repeated fields</td></tr><tr><td>3</td><td>SGROUP</td><td>group start (deprecated)</td></tr><tr><td>4</td><td>EGROUP</td><td>group end (deprecated)</td></tr><tr><td>5</td><td>I32</td><td>fixed32, sfixed32, float</td></tr></tbody></table>

#### 1.tag解析

tag的解析是通过Base 128 Varints算法来解析，Base 128 Varints算法核心是如果字节最高位是1，说明下一个字节仍然需要解析，如果是0，说明这个字节就是tag的结束。

通过Base 128 Varints拿到的数字，低3位代表wire\_type，其他位代表field number。

```
0 0001 000
根据Base 128 Varints最高位的0首先可以判断本次tag是一个字节
然后找到最低的3位，值为0表示wire type是VARINT
然后剩下的部分是field number，一字节的field number最大值是15，这也是为什么最常用的字段最好field number小于15
```

#### 2.value解析

VARINT的value是变长的数字，I64固定是8字节，I32固定是4字节。

LEN类型的value包含两个值，第一个值是标识长度的变长数字，第二个值是真正的payload。

```protobuf
message Test2 {
  optional string b = 2;
}

// 将b赋值为"testing" 如下是编码情况
12 07 [74 65 73 74 69 6e 67]
12 = 00010 010 意思是wire type是LEN，field number是2
07 表示长度是7
```

#### 3.message编码

```protobuf
message Test1 {
  optional int32 a = 1;
}

message Test3 {
  optional Test1 c = 3;
}

// 假设c.a的值是150
1a 03 [08 96 01]
1a = 00011 010 表示field number是3，wire type是2
03 表示负载的长度是3
08 = 00001 000 表示field number是1，wire type是0
96 01 是真正的负载，使用Base 128 Varints表示，为150
```

#### 4.repeated编码

```protobuf
message Test4 {
  optional string d = 4;
  repeated int32 e = 5;
}

// 假设d是"hello"，e是[1,2,3]，以下是编码情况，冒号前表示field number，冒号后是payload
4: {"hello"}
5: 1
5: 2
5: 3

// 可以发现repeated的编码实际就是相同的field number的多个tag-value对
// 不同field之间没必要保证顺序，只需同一field保证顺序，如下也是正确的编码
5: 1
5: 2
4: {"hello"}
5: 3
```

#### 5.map编码

map实际上是repeated字段的一种简写，如下两个Test6的效果是相同的。

```protobuf
message Test6 {
  map<string, int32> g = 7;
}

message Test6 {
  message g_Entry {
    optional string key = 1;
    optional int32 value = 2;
  }
  repeated g_Entry g = 7;
}
```

### 三.参考表

下面是protobuf编码的一个简易顺序表，从前往后读较为清晰：

```gdscript3
message    := (tag value)*

tag        := (field << 3) bit-or wire_type;
                encoded as uint32 varint
value      := varint      for wire_type == VARINT,
              i32         for wire_type == I32,
              i64         for wire_type == I64,
              len-prefix  for wire_type == LEN,
              <empty>     for wire_type == SGROUP or EGROUP

varint     := int32 | int64 | uint32 | uint64 | bool | enum | sint32 | sint64;
                encoded as varints (sintN are ZigZag-encoded first)
i32        := sfixed32 | fixed32 | float;
                encoded as 4-byte little-endian;
                memcpy of the equivalent C types (u?int32_t, float)
i64        := sfixed64 | fixed64 | double;
                encoded as 8-byte little-endian;
                memcpy of the equivalent C types (u?int64_t, double)

len-prefix := size (message | string | bytes | packed);
                size encoded as int32 varint
string     := valid UTF-8 string (e.g. ASCII);
                max 2GB of bytes
bytes      := any sequence of 8-bit bytes;
                max 2GB of bytes
packed     := varint* | i32* | i64*,
                consecutive values of the type specified in `.proto`
```

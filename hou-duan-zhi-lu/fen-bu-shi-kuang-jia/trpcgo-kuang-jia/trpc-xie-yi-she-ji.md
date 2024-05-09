# trpc协议设计

### 一.为什么要设计trpc协议

下面的表格是tRPC对通信协议的能力和诉求：

| 能力           | 诉求                                          |
| ------------ | ------------------------------------------- |
| RPC          | 远程调用像调本地接口一样                                |
| 兼容性          | 协议设计和实现上应具有向前兼容                             |
| 高性能          | 协议设计上需要考虑高性能问题                              |
| 流式传输         | 协议设计上应该能支持大数据包传输和边请求边应答的流式场景                |
| 支持请求超时控制     | 协议设计和实现上支持设置请求超时时间                          |
| 支持多种序列化方式    | 协议设计和实现上可扩展支持多种序列化方式，比如：Protobuf、JSON、text等 |
| 支持多种解压缩方式    | 协议设计和实现上可扩展支持多种解压缩方式，比如：Gzip、Snappy等        |
| 支持透传调用链信息    | 协议设计和实现上支持调用链信息在服务间透传                       |
| 支持消息染色       | 协议设计和实现上支持请求消息的染色key在服务间透传                  |
| 支持透传用户自定义元数据 | 协议设计和实现上支持用户设置自定义元数据在服务间透传                  |

为了支持这些能力和需求，tRPC协议整体设计设计如下：

传输协议（协议头，自定义设计）+ 编码协议（协议体，默认使用Protobuf，可扩展）。

### 二.协议设计

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption><p>trpc协议</p></figcaption></figure>

tRPC协议支持一应一答和流式两种传输方式。

对于一应一答（unary）的RPC，整个完整协议帧 = 固定帧头（16字节）+ Unary包头（变长）+ Unary包体（变长）

对于流式（stream）RPC，整个完整协议帧 = 固定帧头（16字节）+ 流帧（Init帧、Data帧、Feedback帧、Close帧）。

完整的协议定义，可以查看：[trpc.proto](https://github.com/trpc-group/trpc/blob/main/trpc/trpc.proto)。

### 三.具体定义

见[https://github.com/trpc-group/trpc/blob/main/docs/zh/trpc\_protocol\_design.md](https://github.com/trpc-group/trpc/blob/main/docs/zh/trpc\_protocol\_design.md)

### 四.IDL

tRPC 的IDL基于Protobuf，用户编写Proto文件，通过trpc-cmdline生成相应的桩代码。

#### 1.一应一答

```protobuf
syntax = "proto3";

package proto;

service StreamService {
    rpc List(Request) returns (Response) {};
    rpc Record( Request) returns (Response) {};
    rpc Route( StreamRequest) returns (Response) {};
}


message Point {
  string name = 1;
  int32 value = 2;
}

message Request {
  Point pt = 1;
}

message Response {
  Point pt = 1;
}
```

#### 2.流式

```protobuf
syntax = "proto3";

package proto;

service StreamService {
    rpc List(StreamRequest) returns (stream StreamResponse) {};
    rpc Record(stream StreamRequest) returns (StreamResponse) {};
    rpc Route(stream StreamRequest) returns (stream StreamResponse) {};
}


message StreamPoint {
  string name = 1;
  int32 value = 2;
}

message StreamRequest {
  StreamPoint pt = 1;
}

message StreamResponse {
  StreamPoint pt = 1;
}
```

注意关键字 stream，声明其为一个流方法。这里共涉及三个方法，对应关系为：

* List：服务器端流式RPC。
* Record：客户端流式RPC。
* Route：双向流式RPC。

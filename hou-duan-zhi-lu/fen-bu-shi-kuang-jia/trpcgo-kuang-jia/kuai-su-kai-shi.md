# 快速开始

## 一.运行demo

首先需要准备如下依赖：

* [**Go**](https://go.dev/doc/install)**，**版本应该大于等于 go1.18。
* [**tRPC 命令行工具**](https://github.com/trpc-group/trpc-cmdline)**，**用于从 protobuf 生成 Go 桩代码。

然后可以克隆项目运行示例程序：

```sh
git clone --depth 1 git@github.com:trpc-group/trpc-go.git
cd trpc-go/examples/helloworld

# 运行服务端代码
cd server && go run main.go

# 运行客户端代码
cd client && go run main.go

# 最后你会在客户端日志中发现 Hello world! 字样
```

## 二.demo原理

这个demo演示的是一应一答的RPC调用过程：

client通过service proxy调用服务端实现的service，两者通讯的协议都依赖于pb文件生成的stub代码。

### 1.pb文件

最开始只有一个helloworld.proto的pb文件，通过这个文件可以生成两个文件：

* helloworld.pb.go，这个文件是通过protobuf官方能力生成的。
* helloworld.trpc.go，这个文件是trpc命令行工具额外生成的。

```protobuf
syntax = "proto3";

package trpc.helloworld;
option go_package="trpc.group/trpc-go/trpc-go/examples/helloworld/pb";

service Greeter {
  rpc Hello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string msg = 1;
}

message HelloReply {
  string msg = 1;
}
```

### 2.服务端依赖

以下是服务端依赖的代码，有三个核心点：

* 定义GreeterService接口，由外部实现。
* 定义GreeterServer\_ServiceDesc，这是pb文件信息的核心，抽出了service和method信息。
* 定义RegisterGreeterService工具函数，该函数将server.Service、GreeterService、GreeterServer\_ServiceDesc三者组合起来。server.Service是个接口，可以用trpc.NewServer赋值。这里整体的语义是在server.Service中，GreeterServer\_ServiceDesc这个由pb生成的标识，对应着GreeterService处理接口。

```go
type GreeterService interface {
	Hello(ctx context.Context, req *HelloRequest) (*HelloReply, error)
}

func GreeterService_Hello_Handler(svr interface{}, ctx context.Context, f server.FilterFunc) (interface{}, error) {
	req := &HelloRequest{}
	filters, err := f(req)
	if err != nil {
		return nil, err
	}
	handleFunc := func(ctx context.Context, reqbody interface{}) (interface{}, error) {
		return svr.(GreeterService).Hello(ctx, reqbody.(*HelloRequest))
	}

	var rsp interface{}
	rsp, err = filters.Filter(ctx, req, handleFunc)
	if err != nil {
		return nil, err
	}
	return rsp, nil
}

// GreeterServer_ServiceDesc descriptor for server.RegisterService.
var GreeterServer_ServiceDesc = server.ServiceDesc{
	ServiceName: "trpc.helloworld.Greeter",
	HandlerType: ((*GreeterService)(nil)),
	Methods: []server.Method{
		{
			Name: "/trpc.helloworld.Greeter/Hello",
			Func: GreeterService_Hello_Handler,
		},
	},
}

// RegisterGreeterService registers service.
func RegisterGreeterService(s server.Service, svr GreeterService) {
	if err := s.Register(&GreeterServer_ServiceDesc, svr); err != nil {
		panic(fmt.Sprintf("Greeter register error:%v", err))
	}
}
```

### 3.客户端依赖

以下是客户端依赖的代码，核心点有两个：

* 定义GreeterClientProxy接口。
* 定义GreeterClientProxyImpl结构，实现GreeterClientProxy接口，外部直接使用。

```go
type GreeterClientProxy interface {
	Hello(ctx context.Context, req *HelloRequest, opts ...client.Option) (rsp *HelloReply, err error)
}

type GreeterClientProxyImpl struct {
	client client.Client
	opts   []client.Option
}

var NewGreeterClientProxy = func(opts ...client.Option) GreeterClientProxy {
	return &GreeterClientProxyImpl{client: client.DefaultClient, opts: opts}
}

func (c *GreeterClientProxyImpl) Hello(ctx context.Context, req *HelloRequest, opts ...client.Option) (*HelloReply, error) {
	ctx, msg := codec.WithCloneMessage(ctx)
	defer codec.PutBackMessage(msg)
	msg.WithClientRPCName("/trpc.helloworld.Greeter/Hello")
	msg.WithCalleeServiceName(GreeterServer_ServiceDesc.ServiceName)
	msg.WithCalleeApp("")
	msg.WithCalleeServer("")
	msg.WithCalleeService("Greeter")
	msg.WithCalleeMethod("Hello")
	msg.WithSerializationType(codec.SerializationTypePB)
	callopts := make([]client.Option, 0, len(c.opts)+len(opts))
	callopts = append(callopts, c.opts...)
	callopts = append(callopts, opts...)
	rsp := &HelloReply{}
	if err := c.client.Invoke(ctx, req, rsp, callopts...); err != nil {
		return nil, err
	}
	return rsp, nil
}
```

## 三.消息

trpc-go中，将客户端和服务端的消息抽象出一个Msg接口，并提供了一个消息实现。

Msg伴随了整个rpc的生命周期，可以看到其中携带很多rpc信息。

```go
type msg struct {
	context             context.Context
	frameHead           interface{}
	requestTimeout      time.Duration
	serializationType   int
	compressType        int
	streamID            uint32
	dyeing              bool
	dyeingKey           string
	serverRPCName       string
	clientRPCName       string
	serverMetaData      MetaData
	clientMetaData      MetaData
	callerServiceName   string
	calleeServiceName   string
	calleeContainerName string
	serverRspErr        error
	clientRspErr        error
	serverReqHead       interface{}
	serverRspHead       interface{}
	clientReqHead       interface{}
	clientRspHead       interface{}
	localAddr           net.Addr
	remoteAddr          net.Addr
	logger              interface{}
	callerApp           string
	callerServer        string
	callerService       string
	callerMethod        string
	calleeApp           string
	calleeServer        string
	calleeService       string
	calleeMethod        string
	namespace           string
	setName             string
	envName             string
	envTransfer         string
	requestID           uint32
	calleeSetName       string
	streamFrame         interface{}
	commonMeta          CommonMeta
	callType            RequestType
}
```

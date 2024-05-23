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

生成消息的WithCloneMessage函数如下，作用是生成新的msg，并将该消息绑在ctx下的一个新的valueCtx中：

```go
// WithCloneMessage copy a new message and put into context, each rpc call should
// create a new message, this method will be called by client stub.
func WithCloneMessage(ctx context.Context) (context.Context, Msg) {
	newMsg := msgPool.Get().(*msg)
	val := ctx.Value(ContextKeyMessage)
	m, ok := val.(*msg)
	if !ok {
		ctx = context.WithValue(ctx, ContextKeyMessage, newMsg)
		newMsg.context = ctx
		return ctx, newMsg
	}
	ctx = context.WithValue(ctx, ContextKeyMessage, newMsg)
	newMsg.context = ctx
	copyCommonMessage(m, newMsg)
	copyServerToClientMessage(m, newMsg)
	return ctx, newMsg
}
```

## 四.Invoke

在Client中，Invoke是一切的入口：

```go
// Client is the interface that initiates RPCs and sends request messages to a server.
type Client interface {
	// Invoke performs a unary RPC.
	Invoke(ctx context.Context, reqBody interface{}, rspBody interface{}, opt ...Option) error
}
```

代码如下：

```go
func (c *client) Invoke(ctx context.Context, reqBody interface{}, rspBody interface{}, opt ...Option) (err error) {
	// The generic message structure data of the current request is retrieved from the context,
	// and each backend call uses a new msg generated by the client stub code.
	ctx, msg := codec.EnsureMessage(ctx)

	span, end, ctx := rpcz.NewSpanContext(ctx, "client")

	// Get client options.
	opts, err := c.getOptions(msg, opt...)
	defer func() {
		span.SetAttribute(rpcz.TRPCAttributeRPCName, msg.ClientRPCName())
		if err == nil {
			span.SetAttribute(rpcz.TRPCAttributeError, msg.ClientRspErr())
		} else {
			span.SetAttribute(rpcz.TRPCAttributeError, err)
		}
		end.End()
	}()
	if err != nil {
		return err
	}

	// Update Msg by options.
	c.updateMsg(msg, opts)

	fullLinkDeadline, ok := ctx.Deadline()
	if opts.Timeout > 0 {
		var cancel context.CancelFunc
		ctx, cancel = context.WithTimeout(ctx, opts.Timeout)
		defer cancel()
	}
	if deadline, ok := ctx.Deadline(); ok {
		msg.WithRequestTimeout(deadline.Sub(time.Now()))
	}
	if ok && (opts.Timeout <= 0 || time.Until(fullLinkDeadline) < opts.Timeout) {
		opts.fixTimeout = mayConvert2FullLinkTimeout
	}

	// Start filter chain processing.
	filters := c.fixFilters(opts)
	span.SetAttribute(rpcz.TRPCAttributeFilterNames, opts.FilterNames)
	return filters.Filter(contextWithOptions(ctx, opts), reqBody, rspBody, callFunc)
}
```

大致流程如下：

* 将手动指定的消息option覆盖到默认的option上。
* 设置超时和全链路超时。
* 添加selector拦截器。
* 进入客户端拦截器逻辑。

客户端拦截器的代码逻辑如下：

```go
// Filter invokes every client side filters in the chain.
func (c ClientChain) Filter(ctx context.Context, req, rsp interface{}, next ClientHandleFunc) error {
	nextF := func(ctx context.Context, req, rsp interface{}) error {
		_, end, ctx := rpcz.NewSpanContext(ctx, "CallFunc")
		err := next(ctx, req, rsp)
		end.End()
		return err
	}

	names, ok := names(ctx)
	for i := len(c) - 1; i >= 0; i-- {
		curHandleFunc, curFilter, curI := nextF, c[i], i
		nextF = func(ctx context.Context, req, rsp interface{}) error {
			if ok {
				var ender rpcz.Ender
				_, ender, ctx = rpcz.NewSpanContext(ctx, name(names, curI))
				defer ender.End()
			}
			return curFilter(ctx, req, rsp, curHandleFunc)
		}
	}
	return nextF(ctx, req, rsp)
}
```

这里的逻辑是将一个个的拦截器组合成一串闭包，最后执行nextF的效果是，从第一个拦截器执行到最后一个拦截器，然后进入真正的ClientHandleFunc逻辑。client中真正的clientHandleFunc逻辑是callFunc函数，是真正进行网络请求的地方。

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

callFunc函数如下：

```go
func callFunc(ctx context.Context, reqBody interface{}, rspBody interface{}) (err error) {
	msg := codec.Message(ctx)
	opts := OptionsFromContext(ctx)

	defer func() { err = opts.fixTimeout(err) }()

	// Check if codec is empty, after updating msg.
	if opts.Codec == nil {
		report.ClientCodecEmpty.Incr()
		return errs.NewFrameError(errs.RetClientEncodeFail, "client: codec empty")
	}

	reqBuf, err := prepareRequestBuf(ctx, msg, reqBody, opts)
	if err != nil {
		return err
	}

	// Call backend service.
	if opts.EnableMultiplexed {
		opts.CallOptions = append(opts.CallOptions, transport.WithMsg(msg), transport.WithMultiplexed(true))
	}
	rspBuf, err := opts.Transport.RoundTrip(ctx, reqBuf, opts.CallOptions...)
	if err != nil {
		if err == errs.ErrClientNoResponse { // Sendonly mode, no response, just return nil.
			return nil
		}
		return err
	}

	span := rpcz.SpanFromContext(ctx)
	span.SetAttribute(rpcz.TRPCAttributeResponseSize, len(rspBuf))
	_, end := span.NewChild("DecodeProtocolHead")
	rspBodyBuf, err := opts.Codec.Decode(msg, rspBuf)
	end.End()
	if err != nil {
		return errs.NewFrameError(errs.RetClientDecodeFail, "client codec Decode: "+err.Error())
	}

	return processResponseBuf(ctx, msg, rspBody, rspBodyBuf, opts)
}
```

这个函数主要做如下几件事：

* 序列化 --> 压缩 --> 编码
* 进入Transport的RoundTrip的逻辑，也就是真正发请求的地方
* 解码 --> 解压缩 --> 反序列化

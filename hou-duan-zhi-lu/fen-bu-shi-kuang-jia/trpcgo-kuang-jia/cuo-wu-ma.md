# 错误码

### 一.简介

相关demo见：trpc-go/examples/features/errs。

良好的错误码设计对分布式系统非常重要，错误码主要用来快速定位错误原因是什么，trpc-go有自己的错误码设计。

pb里无需定义错误，通过trpc协议能够知道具体的错误信息。

trpc-go的错误码定义如下：

```go
type Error struct {
	Type int
	Code trpcpb.TrpcRetCode
	Msg  string
	Desc string

	cause error      // internal error, form the error chain.
	stack stackTrace // call stack, if the error chain already has a stack, it will not be set.
}
```

重要的字段有三个，分别是Type、Code、Msg。

* Type：标识这是一个什么样的错误，目前有三种错误，一是框架错误码，二是下游框架错误码，三是业务错误码。
* Code：具体的数字错误码，实际是个int32类型。
* Msg：对错误码的解释。

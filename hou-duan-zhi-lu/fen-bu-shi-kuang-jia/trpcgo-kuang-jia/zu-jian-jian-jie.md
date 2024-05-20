# 组件简介

## 一.日志

trpc-go自带日志支持，日志在分布式系统中的重要性是不言而喻的，如果没有日志，就没办法准确定位到问题。

trpc-go中日志是作为插件出现的，内部使用了zap库，支持控制台打印和文件打印，同样也支持上传到远端的日志存储系统，如腾讯云的cls。

## 二.错误码

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

## 三.配置

trpc-go支持自定义配置，既可以从本地读取，也可以从远端配置中心读取，同时支持配置动态更新。

## 四.admin

管理命令（admin）是服务内部的管理后台，它是框架在普通服务端口之外额外提供的 http 服务，通过这个 http 接口可以给服务发送指令，如查看日志等级，动态设置日志等级，还可以查看pprof，搭配上火焰图代理网站，可以在线浏览任何一台机器的火焰图来排查信息。

## 五.名字服务

### 1.客户端

trpc-go客户端和名字服务相关的组件是selector，selector包含discovery、servicerouter、loadbalance、circuitbreaker4个部分。

selector负责从某个service中筛选出一个后端节点，由如下步骤组成：

* discovery进行服务发现，找到service下所有后端节点。
* servicerouter在discovery和loadbalance之间做过滤。
* loadbalance负责在所有节点中选择一个节点，有轮询、加权轮询、一致性哈希等算法。
* circuitbreaker负责熔断，当调用出现网络问题时，上报错误。

默认 selector 逻辑: Discovery->ServiceRouter->LoadBalance->Node->业务使用->CircuitBreaker.Report。

### 2.服务端

trpc-go服务端和名字服务相关的组件是registry，定义了服务注册的通用接口。

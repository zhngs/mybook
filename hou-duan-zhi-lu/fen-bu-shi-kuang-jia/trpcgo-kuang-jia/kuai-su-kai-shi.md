# 快速开始

### 一.运行demo

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

### 二.demo原理

这个demo演示的是一应一答的RPC调用过程：

client通过service proxy调用服务端实现的service，两者通讯的协议都依赖于pb文件生成的stub代码。

# 术语

### 一.trpc命名规范

通常线上提供的业务服务，会包括多个业务子系统，每个业务子系统内部又包括多个独立服务。这些服务之间的调用一般都是使用专门的路由名字间接寻址到被调用者的ip/port来进行访问的，而不是直接写死ip/port。

因此，服务会先把自身的路由名字会注册到名字服务系统中，然后将其作为这个服务的唯一身份，方便主调服务进行识别，这就要求服务自身的命名能在众多微服务中具有唯一性。

同时在微服务架构下，服务很多，如何通过服务的一些信息能快速识别某个服务，方便对服务日常的运维管理，比如：服务部署、监控、告警等，这也对服务的命名有一定的规范要求。

因此，trpc对服务命名、路由命名、RPC接口命名做了一些规范设计。

### 二.服务命名

tRPC在服务命名上定义了以下3个纬度信息：

* app名（应用名），表示某个业务系统的名称，用于标识某个业务下不同服务模块的一个集合。
* server名（模块名），表示具体服务模块的名称，一般也称为模块的进程名称。
* service名，表示具体服务提供者的名称，一般使用proto文件定义的Service名称； 其中`app.server` 的组合在全局上要具备唯一性。

服务命名这样定义的好处是：

* 对于服务开发，可以方便脚手架工具自动生成tRPC服务代码。
* 对于服务寻址，可以生成全局唯一的服务路由名称，方便服务注册和服务发现。
* 对于服务运营，可以方便服务的部署、各种维度的监控数据采集、告警等。

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>rpc调用示意图</p></figcaption></figure>

服务A作为客户端，服务B作为服务端，在tRPC中我们称客户端也为 `主调(caller)`，服务端也为 `被调(callee)`。主调（caller）A通过服务路由的名字（Route ID）向名字服务拿到ip/port列表后，然后在通信协议中设置调用的RPC接口名（Interface ID），访问被调B（callee）。

### 三.服务路由命名

为了方便对服务的路由寻址，服务的路由命名规范如下：

由`.`号分割的四段字符串 **"trpc.{app}.{server}.{service}"** 来组成。

其中，第1段固定为`trpc`，表示这个服务是一个tRPC服务，第2段到第4段参考服务命名的含义。

另外，主调（caller）与被调（callee）的命名一般也与服务的路由命名一致。

### 四.RPC接口命名

下面是一个比较简单的服务提供的接口proto文件：

```protobuf
syntax = "proto3";

package trpc.test.helloworld;

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
   string msg = 1;
}

message HelloReply {
   string msg = 1;
}
```

基于proto文件的用法，tRPC在tRPC协议上对RPC接口的名字制定了统一的规范，RPC调用方法名由proto文件中的 **"/{package\_name}.{service\_name}/{method\_name}"** 组成。

其中 `{package_name}` 的命名我们建议使用 `trpc.{app}.{server}`，`{service_name}` 为上面proto文件中定义的service名字，`{method_name}` 为service下具体的方法名，即我们要调用的RPC接口。

即RPC的调用方法名建议为**"/{trpc}.{app}.{server}.{service}/{method}"。**

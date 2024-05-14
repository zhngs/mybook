# 名字服务

### 一.客户端

trpc-go客户端和名字服务相关的组件是selector，selector包含discovery、servicerouter、loadbalance、circuitbreaker4个部分。

selector负责从某个service中筛选出一个后端节点，由如下步骤组成：

* discovery进行服务发现，找到service下所有后端节点。
* servicerouter在discovery和loadbalance之间做过滤。
* loadbalance负责在所有节点中选择一个节点，有轮询、加权轮询、一致性哈希等算法。
* circuitbreaker负责熔断，当调用出现网络问题时，上报错误。

默认 selector 逻辑: Discovery->ServiceRouter->LoadBalance->Node->业务使用->CircuitBreaker.Report。

### 二.服务端

trpc-go服务端和名字服务相关的组件是registry，定义了服务注册的通用接口。




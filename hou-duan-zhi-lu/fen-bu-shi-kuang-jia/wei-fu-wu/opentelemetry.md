# OpenTelemetry

### 一.简介

OpenTelemetry是一个可见性框架和工具集，被设计来创建和管理遥测数据，比如trace，metric，log。

trace是分布式追踪，在微服务领域可以将一次请求经过的上下游比如redis、mysql、rpc服务等串起来，方便快速排查问题。

metric是视图指标，用来监控服务的状态。

log是日志，程序运行时打印。

### 二.trace

在trace中的一个重要概念是span，span代表一次操作，比如RPC调用、http调用等。

span包含如下信息([https://opentelemetry.io/docs/concepts/signals/traces/](https://opentelemetry.io/docs/concepts/signals/traces/))：

* Name
* Parent span ID (empty for root spans)
* Start and End Timestamps
* Span Context
* Attributes
* Span Events
* Span Links
* Span Status

如下是一个简单的span示例：

```
{
  "name": "/v1/sys/health",
  "context": {
    "trace_id": "7bba9f33312b3dbb8b2c2c62bb7abe2d",
    "span_id": "086e83747d0e381e"
  },
  "parent_id": "",
  "start_time": "2021-10-22 16:04:01.209458162 +0000 UTC",
  "end_time": "2021-10-22 16:04:01.209514132 +0000 UTC",
  "status_code": "STATUS_CODE_OK",
  "status_message": "",
  "attributes": {
    "net.transport": "IP.TCP",
    "net.peer.ip": "172.17.0.1",
    "net.peer.port": "51820",
    "net.host.ip": "10.177.2.152",
    "net.host.port": "26040",
    "http.method": "GET",
    "http.target": "/v1/sys/health",
    "http.server_name": "mortar-gateway",
    "http.route": "/v1/sys/health",
    "http.user_agent": "Consul Health Check",
    "http.scheme": "http",
    "http.host": "10.177.2.152:26040",
    "http.flavor": "1.1"
  },
  "events": [
    {
      "name": "",
      "message": "OK",
      "timestamp": "2021-10-22 16:04:01.209512872 +0000 UTC"
    }
  ]
}
```

一次trace是由多个span组成，最后可以得到类似如下的瀑布图，可以比较清楚地看到父子关系和时序关系：

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption><p>瀑布图</p></figcaption></figure>

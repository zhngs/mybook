# 日志

### 一.demo演示

执行如下命令：

```sh
cd trpc-go/examples/features/log

# 服务端
go run server/main.go -conf server/trpc_go.yaml

# 客户端
go run client/main.go
```

服务端结果如下所示：

```log
2024-04-06 13:48:37.338 WARN    server/main.go:41       recv msg:msg:"Hello"
2024-04-06 13:48:37.338 ERROR   server/main.go:42       recv msg:msg:"Hello"
```

客户端结果如下所示：

```log
2024/04/06 13:48:37 Received response: trpc-go-server response: Hello
```

### 二.原理

代码里包含了`"trpc.group/trpc-go/trpc-go/log"`依赖，引入了日志模块。

查看trpc\_go.yaml如下：

```yaml
global:                             # global config.
  namespace: development            # environment type, two types: production and development.
  env_name: test                    # environment name, names of multiple environments in informal settings.

server:                                            # server configuration.
  app: examples                                    # business application name.
  server: logExample                               # service process name.
  service:                                         # business service configuration，can have multiple.
    - name: trpc.examples.log.Log                  # the route name of the service.
      ip: 127.0.0.1                                # the service listening ip address, can use the placeholder ${ip}, choose one of ip and nic, priority ip.
      port: 8080                                   # the service listening port, can use the placeholder ${port}.
      network: tcp                                 # the service listening network type,  tcp or udp.
      protocol: trpc                               # application layer protocol, trpc or http.
      timeout: 1000                                # maximum request processing time in milliseconds.
      idletime: 300000                             # connection idle time in milliseconds.

plugins:                                           # plugin configuration.
  log:                                             # logging configuration.
    default:                                       # default logging configuration, supports multiple outputs.
      - writer: console                            # console standard output, default setting.
        level: warn                                # log level of standard output.
      - writer: file                               # local file logging.
        level: debug                               # log level of local file rolling logs.
        formatter: json                            # log format for standard output.
        writer_config:
          filename: ./trpc.log                     # storage path of rolling continuous log files.
          max_size: 10                             # maximum size of local log files, in MB.
          max_backups: 10                          # maximum number of log files.
          max_age: 7                               # maximum number of days to keep log files.
          compress: false                          # determine whether to compress log files.
```

可以看到plugins里有日志相关的配置，trpc-go的日志使用了[uber-go/zap](https://github.com/uber-go/zap)。

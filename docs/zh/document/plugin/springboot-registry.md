# SpringBoot 注册

本文介绍如何使用[SpringBoot注册插件](https://github.com/sermant-io/Sermant/tree/develop/sermant-plugins/sermant-springboot-registry)。

## 功能介绍

该插件为纯SpringBoot应用提供服务注册发现能力，方便用户在不修改代码的前提下快速接入注册中心（目前只支持**ZooKeeper**），同时提供超时重试的能力，实现服务调用的高可用。

插件会根据发起客户端调用Url解析下游服务，并根据负载均衡策略选择优选实例，动态替换Url，完成服务调用。

目前Url支持的格式：http://${domainName}/${serviceName}/${apiPath}，其中`domainName`为实际调用的域名，`serviceName`为下游的服务名，`apiPath`则为下游请求接口路径。

## 参数配置

### 插件配置

SpringBoot注册插件需要按需修改插件配置文件，可在`${path}/sermant-agent-x.x.x/agent/pluginPackage/springboot-registry/config/config.yaml`找到该插件的配置文件，配置文件如下所示

```yaml
sermant.springboot.registry:
  enableRegistry: false                 # 是否开启springboot注册能力
  realmName: www.domain.com             # 匹配域名, 当前版本仅针对url为http://${realmName}/serviceName/api/xx场景生效
  enableRequestCount: false             # 是否开启流量统计, 开启后每次进入插件的流量将都会统计
     
sermant.springboot.registry.lb:     
  lbType: RoundRobin                    # 负载均衡策略, 当前支持轮询(RoundRobin)、随机(Random)、响应时间权重(WeightedResponseTime)、最低并发数(BestAvailable)
  registryAddress: 127.0.0.1:2181       # 注册中心地址
  instanceCacheExpireTime: 0            # 实例过期时间, 单位秒, 若<=0则永不过期
  instanceRefreshInterval: 0            # 实例刷新时间, 单位秒, 必须小于instanceCacheExpireTime
  refreshTimerInterval: 5               # 实例定时检查间隔, 判断实例是否过期, 若其大于instanceRefreshInterval, 则值设置为instanceRefreshInterval
  enableSocketReadTimeoutRetry: true    # 针对{@link java.net.SocketTimeoutException}: read timed out是否需要重试, 默认开启
  enableSocketConnectTimeoutRetry: true # 同上, 主要针对connect timed out, 通常在连接不上下游抛出
  enableTimeoutExRetry: true            # 重试场景, 针对{@link java.util.concurrent.TimeoutException}, 是否需要重试, 默认开启, 该超时多用于异步场景, 例如Future, MinimalHttpAsyncClient
```

配置项说明如下:

| 参数键                                                            | 说明                                                                                      | 默认值            | 是否必须 |
|----------------------------------------------------------------|-----------------------------------------------------------------------------------------|----------------|------|
| sermant.springboot.registry.enableRegistry                     | 是否开启springboot注册能力（true/false）                                                          | false          | 是    |
| sermant.springboot.registry.realmName                          | 匹配域名, 当前版本仅针对url为**http://${realmName}/serviceName/api/xx**场景生效                         | www.domain.com | 是    |
| sermant.springboot.registry.enableRequestCount                 | 是否开启流量统计, 开启后每次进入插件的流量将都会统计（true/false）                                                 | false          | 是    |
| sermant.springboot.registry.lb.lbType                          | 负载均衡类型, 当前支持轮询(RoundRobin)、随机(Random)、响应时间权重(WeightedResponseTime)、最低并发数(BestAvailable) | RoundRobin     | 是    |
| sermant.springboot.registry.lb.registryAddress                 | 注册中心地址                                                                                  | 127.0.0.1:2181 | 是    |
| sermant.springboot.registry.lb.instanceCacheExpireTime         | 实例过期时间, 单位秒, 若<=0则永不过期                                                                  | 0              | 是    |
| sermant.springboot.registry.lb.instanceRefreshInterval         | 实例刷新时间, 单位秒, 必须小于instanceCacheExpireTime                                                | 0              | 是    |
| sermant.springboot.registry.lb.refreshTimerInterval            | 实例定时检查间隔, 判断实例是否过期, 若其大于instanceRefreshInterval, 则值设置为instanceRefreshInterval           | 5              | 是    |
| sermant.springboot.registry.lb.enableSocketReadTimeoutRetry    | 针对**java.net.SocketTimeoutException: read timed out**是否需要重试（true/false）                 | true           | 是    |
| sermant.springboot.registry.lb.enableSocketConnectTimeoutRetry | 针对**java.net.SocketTimeoutException: connect timed out**是否需要重试（true/false）              | true           | 是    |
| sermant.springboot.registry.lb.enableTimeoutExRetry            | 针对**java.util.concurrent.TimeoutException**是否需要重试（true/false）                           | true           | 是    |

## 详细治理规则

SpringBoot注册插件需根据指定服务名判断是否需要为请求进行代理，替换url地址。生效服务需基于动态配置中心进行白名单发布，配置发布可以参考[动态配置中心使用手册](../user-guide/configuration-center.md#sermant动态配置中心模型)。

其中key值为**sermant.plugin.registry**。

group为 **app=${service.meta.application}&environment=${service.meta.environment}&service={spring.application.name}** 即服务配置，其中service.meta.application、service.meta.environment的配置请参考[Sermant-agent使用手册](../user-guide/sermant-agent.md#sermant-agent使用参数配置), spring.application.name为微服务名（即spring应用中配置的服务名）。

> **说明：** 服务配置说明参考[CSE配置中心概述](https://support.huaweicloud.com/devg-cse/cse_devg_0020.html)。

content为白名单的具体配置内容，详细说明如下：

```yaml
strategy: all # 白名单类型，all（全部生效）/none（全不生效）/white（value值中配置的才生效）
value: service-b,service-c # 白名单服务集合，仅当strategy配置为white时生效，多个服务名用英文逗号分隔
```

> **注意：** 新增配置时，请去掉注释，否则会导致新增失败。

## 支持版本和限制

框架支持：

- SpringBoot 1.5.10.Release及以上

注册中心支持：

- ZooKeeper 3.6.x及以上

客户端支持：

- HttpClient: 4.x
  
- HttpAsyncClient: 4.1.4
  
- OkhttpClient: 2.x, 3.x, 4.x
  
- Feign(springcloud-openfeign-core): 2.1.x, 3.0.x
  
- RestTemplate(Spring-web): 5.1.x, 5.3.x

## 操作和结果验证

下面将演示如何使用SpringBoot注册插件，验证纯SpringBoot应用快速接入注册中心（ZooKeeper）场景。

### 1 准备工作

- [下载](https://github.com/sermant-io/Sermant/releases/download/v2.1.0/sermant-2.1.0.tar.gz) Sermant Release包（当前版本推荐2.1.0版本）
- [下载](https://github.com/sermant-io/Sermant-examples/releases/download/v2.1.0/sermant-examples-springboot-registry-demo-2.1.0.tar.gz) Demo二进制产物压缩包
- [下载](https://zookeeper.apache.org/releases.html#download) ZooKeeper（动态配置中心&注册中心），并启动

### 2 获取Demo二进制产物

解压Demo二进制产物压缩包，即可得到`service-a.jar`和`service-b.jar`。

### 3 部署应用

（1）启动service-a

```shell
# windows
java -Dserver.port=8989 -Dsermant.springboot.registry.enableRegistry=true -javaagent:${path}\sermant-agent-x.x.x\agent\sermant-agent.jar=appName=default -jar service-a.jar

# mac, linux
java -Dserver.port=8989 -Dsermant.springboot.registry.enableRegistry=true -javaagent:${path}/sermant-agent-x.x.x/agent/sermant-agent.jar=appName=default -jar service-a.jar
```

（2）启动service-b

```shell
# windows
java -Dserver.port=9999 -Dsermant.springboot.registry.enableRegistry=true -javaagent:${path}\sermant-agent-x.x.x\agent\sermant-agent.jar=appName=default -jar service-b.jar

# mac, linux
java -Dserver.port=9999 -Dsermant.springboot.registry.enableRegistry=true -javaagent:${path}/sermant-agent-x.x.x/agent/sermant-agent.jar=appName=default -jar service-b.jar
```

> **说明：** ${path}为sermant实际安装路径，x.x.x代表sermant某个版本号。

> **注意：** 此时配置的域名(www.domain.com)不是真实域名，配置白名单之后才能正常调用。

### 4 配置白名单

配置白名单，请参考[详细治理规则](#详细治理规则)。

其中key值为**sermant.plugin.registry**，group为**app=default&environment=&service=service-a**，content为**strategy: all**。

利用ZooKeeper提供的命令行工具进行配置发布。

1、在`${path}/bin/`目录执行以下命令创建节点`/app=default&environment=`

```shell
# linux mac
./zkCli.sh -server localhost:2181 create /app=default&environment=&service=service-a

# windows
zkCli.cmd -server localhost:2181 create /app=default&environment=&service=service-a
```

> **说明：** `${path}`为ZooKeeper的安装目录

2、在`${path}/bin/`目录执行以下命令创建节点`/app=default&environment=&service=service-a/sermant.plugin.registry`和数据`strategy: all`。

```shell
# linux mac
./zkCli.sh -server localhost:2181 create /app=default&environment=&service=service-a/sermant.plugin.registry "strategy: all"

# windows
zkCli.cmd -server localhost:2181 create /app=default&environment=&service=service-a/sermant.plugin.registry "strategy: all"
```

### 5 验证

调用接口`localhost:8989/httpClientGet`，判断接口是否成功返回，若成功返回则说明插件已成功生效。

**效果图如下图所示：**

<MyImage src="/docs-img/springboot-registry.png"/>
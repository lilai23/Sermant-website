# 性能基准测试

本文档包含Sermant性能的基准测试结果(持续更新)。

## 标签路由插件

我们使用 [Sermant-examples](https://github.com/sermant-io/Sermant-examples) 仓库中的作为基准应用进行性能测试，以说明Sermant的[service-router标签路由插件](../plugin/router.md)挂载至应用上的性能表现。

### 部署场景

本次测试我们将上述仓库中的 [Spring Cloud应用](https://github.com/sermant-io/Sermant-examples/tree/main/grace-demo/spring-grace-nacos-demo) 部署至容器环境中:

- nacos-rest-consumer，部署1个Pod，挂载Sermant的service-router插件。该服务作为入口服务，根据动态配置中心下发的标签规则来筛选下游实例进行调用。本次测试将监控该服务的性能表现。
- nacos-rest-provider，部署3个Pod，挂载Sermant的service-router插件，并且不同的Pod配置的不同的标签以供服务消费者进行筛选。该服务作为nacos-rest-consumer的服务提供者。
- nacos-rest-data，部署1个Pod，不挂载Sermant，该服务作为作为nacos-rest-provider的服务提供者。

该部署场景中，服务调用关系如下：

<MyImage src="/docs-img/test_router.jpg"/>

我们向动态配置中心下发合法的路由插件规则，使得nacos-rest-consumer的请求按标签路由规则发送到指定的nacos-rest-provider实例。

另外，本次测试的基线对照组，所有应用均不挂载Sermant。

### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为2个，规格如下：

```
规格：通用计算型｜16vCPUs｜32GiB｜s6.4xlarge.2
Docker Version: v18.09.9
Kubernetes Version: v1.23
```

K8s中所有应用Pod的规格一致，均为`4vCPUs|8GiB`。

Sermant版本：[`Release v1.1.0`](https://github.com/sermant-io/Sermant/releases/tag/v1.1.0)

### 测试结果

使用Jmeter对nacos-rest-consumer进行并发调用，分别模拟50用户、目标1000tps，以及100用户、目标2000tps。

| 并发线程数 | CPU(基线/挂载Sermant/差异) | Heap内存                | Metaspace内存         | P90(ms)            | P95(ms)            | Throughout(/s)          |
| ---------- | -------------------------- | ----------------------- | --------------------- | ------------------ | ------------------ | ----------------------- |
| 50         | 13.8% / 16.9% / 3.1%       | 92.7M / 122.5M /  29.8M | 56.8M / 66.6M / 9.8M  | 34ms / 34ms / 0    | 35ms / 36ms / 2.8% | 992.8 / 986.4 / -0.6%   |
| 100        | 26.9% / 32.5% / 5.6%       | 150.4M / 183.5M / 33.1M | 56.6M / 66.8M / 10.2M | 34ms / 35ms / 2.9% | 35ms / 38ms / 7.9% | 1980.4 / 1965.4 / -0.7% |

### 总结

Sermant标签路由插件对于宿主应用的CPU占用率、内存占用、吞吐量以及时延影响较低，额外消耗的资源主要用于请求过程中路由规则的匹配和实例的筛选。

## 离群实例摘除插件

我们使用 [Sermant-examples](https://github.com/sermant-io/Sermant-examples) 仓库中的作为基准应用进行性能测试，以说明Sermant的[service-removal离群实例摘除插件](../plugin/removal.md)挂载至应用上的性能表现。

### 部署场景

本次测试我们将上述仓库中的 [Spring Cloud应用](https://github.com/sermant-io/Sermant-examples/tree/main/grace-demo/spring-grace-nacos-demo) 部署至容器环境中:

- nacos-rest-consumer，部署1个Pod，挂载Sermant的service-removal插件。该服务作为入口服务，离群实例摘除插件挂载于该插件上，会对异常的nacos-rest-provider实例做摘除操作，避免请求调用至状态异常的实例。本次测试将监控该服务的性能表现。
- nacos-rest-provider，部署3个Pod，不挂载Sermant。该服务作为nacos-rest-consumer的服务提供者。
- nacos-rest-data，部署1个Pod，不挂载Sermant，该服务作为作为nacos-rest-provider的服务提供者。

该部署场景中，服务调用关系如下：

<MyImage src="/docs-img/test_removal.jpg"/>

我们在离群实例摘除插件中配置合法的实例摘除规则，使得nacos-rest-consumer将状态异常的实例从服务发现列表中删除。

另外，本次测试的基线对照组，所有应用均不挂载Sermant。

### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为2个，规格如下：

```
规格：通用计算型｜16vCPUs｜32GiB｜s6.4xlarge.2
Docker Version: v18.09.9
Kubernetes Version: v1.23
```

K8s中所有应用Pod的规格一致，均为`4vCPUs|8GiB`。

Sermant版本：[`Release v1.1.0`](https://github.com/sermant-io/Sermant/releases/tag/v1.1.0)

### 测试结果

使用Jmeter对nacos-rest-consumer进行并发调用，分别模拟50用户、目标1000tps，以及100用户、目标2000tps。

| 并发线程数 | CPU(基线/挂载Sermant/差异) | Heap内存                | Metaspace内存        | P90(ms)         | P95(ms)             | Throughout(/s)          |
| ---------- | -------------------------- | ----------------------- | -------------------- | --------------- | ------------------- | ----------------------- |
| 50         | 13.8% / 14.3% / 0.5%       | 92.7M / 105.7M /  13.0M | 56.8M / 65.5M / 8.7M | 34ms / 34ms / 0 | 35ms / 34ms / -2.9% | 992.8 / 993.5 / 0.1%    |
| 100        | 26.9% / 28.2% / 0.3%       | 150.4M /  167M / 16.6M  | 56.6M / 65.5M / 8.9M | 34ms / 34ms / 0 | 34ms / 34ms / 0     | 1980.4 / 1984.5 /  0.1% |

### 总结

Sermant离群实例摘除插件对于宿主应用的CPU占用率、内存占用、吞吐量以及时延影响非常轻微，额外的资源消耗主要用于请求成功率的统计以及离群实例的摘除过程，对请求过程基本无影响。

## 流量标签透传插件

本次测试分为两个部分，分别是Http/RPC的测试和消息队列的测试。其中Http/RPC部分使用包含HttpClient、Okhttp、Dubbo、Grpc等客户端的测试微服务进行压测，消息队列部分使用包含Kafka、RocketMQ的测试微服务进行Benchmark测试，以说明Sermant的[流量标签透传插件](../plugin/tag-transmission.md)挂载至被测微服务上的性能表现。

### 部署场景

本次测试的分为对照组和实验组，对照组所有被测微服务均不挂载Sermant，实验组的被测微服务均挂载Sermant。

Http场景中分别部署Http客户端以及服务端的测试微服务，RPC场景分别部署消费端和服务提供端的测试微服务。使用Jmeter对Http客户端和RPC的消费端进行压测。分别记录对照组和实验组的压测指标数据。

消息队列场景中，生产环节测试由Kafka或RocketMQ的生产者生产相同数量的消息，分别记录对照组和实验组的耗时。消费环节的测试，需先在Kafka和RocketMq中生产足够多的消息以供测试，然后分别记录对照组和实验组消费相同数量消息的耗时。

### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为2个，规格如下：

```
规格：通用计算型｜16vCPUs｜32GiB｜s6.4xlarge.2
Docker Version: v18.09.9
Kubernetes Version: v1.23
```

K8s中所有应用Pod的规格一致，均为`4vCPUs|8GiB`。

Sermant版本：[`Release v1.2.0`](https://github.com/sermant-io/Sermant/releases/tag/v1.2.0)

### Http/RPC测试结果 

| **被测组件**       | 达到最大TPS时长（无Sermant/有Sermant/ 差异） | 最大TPS（无Sermant/有Sermant/ 差异） | 平均TPS（无Sermant/有Sermant/ 差异） | CPU占用率增加 | Heap内存增加 | metaspace增加 |
| ------------------ | -------------------------------------------- | ------------------------------------ | ------------------------------------ | ------------- | ------------ | ------------- |
| HttpClient(3.1)    | 16s / 17s / +1s                              | 540 / 538 / -0.4%                    | 525 / 524 / -0.2%                    | 1.57%         | 15M          | 16M           |
| HttpClient(4.5.13) | 18s / 18s / 0                                | 489 / 486 / -0.6%                    | 474 / 475 / +0.2%                    | 0.58%         | 13M          | 17M           |
| Okhttp(2.7.5)      | 20s / 21s / +1s                              | 616 / 617 / +0.2%                    | 610 / 608 / -0.3%                    | 2.96%         | 15M          | 15M           |
| Jdkhttp(jdk8)      | 19s / 19s / 0                                | 615 / 614 / -0.2%                    | 609 / 605 / -0.7%                    | 3.06%         | 19M          | 16M           |
| Dubbo(2.7.5)       | 12s / 13s / +1s                              | 600 / 599 / -0.2%                    | 583 / 585 / +0.3%                    | 5.10%         | 18M          | 15M           |
| Grpc(1.52.1)       | 18s / 19s / +1s                              | 622 / 620 / -0.3%                    | 608 / 607 / -0.2%                    | 3.21%         | 14M          | 15M           |
| Sofarpc(5.10.0)    | 20s / 22s / +2s                              | 604 / 600 / -0.7%                    | 587 / 586 / -0.2%                    | 2.92%         | 18M          | 15M           |
| Servicecomb(2.8.3) | 25s / 23s / -2s                              | 611 / 614 / +0.5%                    | 599 / 600 / +0.2%                    | 4.07%         | 17M          | 14M           |

### 消息队列测试结果 

| **组件**               | **处理耗时增加**   |
| ---------------------- | ------------------ |
| Kafka(2.7.0) 生产者    | 生产每百万条消息3s |
| Kafka(2.7.0) 消费者    | 消费每百万条消息1s |
| RocketMQ(4.8.0) 生产者 | 生产每十万条消息1s |
| RocketMQ(4.8.0) 消费者 | 消费每十万条消息1s |

### 总结

Http/RPC测试中，对达到最大TPS时长、最大TPS、平均TPS、CPU占用率增加、metaspace增加等数据指标做了记录。可以发现Sermant流量标签透传插件的对TPS的影响小于1%，CPU占用率的损耗基本在5%以下，其中Dubbo稍高是因为插件中的反射过程导致的。另外Sermant在压测时对内存占用的增加量也较少。

消息队列与Http、RPC测试方式不同，本次测试主要通过生产或消费固定量的消息的时间消耗的增加量来衡量性能损耗，挂载Sermant流量标签透传插件时，Kafka生产每百万条消息时间增加3s， Kafka消费每百万条消息时间增加1s；RocketMQ生产每十万条消息时间增加1s，RocketMQ消费每十万条消息时间增加1s。表明Sermant流量标签透传插件对二者的性能损耗较低。

## 消息队列禁止消费插件

消息队列禁止消费插件实现了类似于开关的功能，对微服务的TPS等指标无影响，因此本次测试主要针对内存的影响来进行，以说明Sermant的[消息队列禁止消费插件](plugin/mq-consume-prohibition.md)挂载至被测微服务的性能表现。

### 部署场景

本次测试的分为对照组和实验组，对照组所有被测微服务均不挂载Sermant，实验组的被测微服务均挂载Sermant。

针对Kafka和RocketMQ的场景分别进行测试，被测微服务中的消费者均订阅消息队列的Topic进行轮询消费，实验组中Sermant消息队列禁止消费的开关关闭，使用ZooKeeper作为动态配置中心，测试对照组和实验组的内存占用差异。

### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为2个，规格如下：

```
规格：通用计算型｜16vCPUs｜32GiB｜s6.4xlarge.2
Docker Version: v18.09.9
Kubernetes Version: v1.23
```

K8s中所有应用Pod的规格一致，均为`4vCPUs|8GiB`。

Sermant版本：[`Release v1.3.0`](https://github.com/sermant-io/Sermant/releases/tag/v1.3.0)

### 测试结果 

| 测试场景             | 内存总增加量 | Heap内存增加量 | 堆外内存增加量 | Class增加量 | 线程增加量 |
| -------------------- | ------------ | -------------- | -------------- | ----------- | ---------- |
| Kafka(2.7.0)         | 19.6M        | 1.6M           | 18M            | 2055        | 3          |
| RocketMQ-push(5.0.0) | 22.2M        | 2.2M           | 20M            | 2258        | 3          |
| RocketMQ-pull(5.0.0) | 21.7M        | 2.4M           | 19M            | 2262        | 4          |

### 总结

测试结果表明Sermant消息队列禁止消费插件对内存的占用20M左右，影响较小。另外挂载Sermant后Class增加量为2000～2200左右，线程的增加量3～4个，主要来自于Sermant框架的logback线程以及ZooKeeper动态配置监听的EventThread和SendThread线程，其中RocketMQ-pull场景额外增加1个重平衡线程。

## 数据库禁写插件

本次测试分为对照组和实验组，对照组所有被测微服务均不挂载Sermant，实验组的被测微服务均挂载Sermant。
测试的数据库分别为MySQL、MongoDB、PostgreSQL和OpenGauss，测试场景为测试微服务查询数据库数据和插入数据库数据。
使用Jmeter对测试微服务进行压测，微服务启动时挂载async-profiler，分别记录对照组和实验组的压测指标数据。

### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为3个，规格如下：

```
规格：通用计算型｜16vCPUs｜32GiB｜s6.4xlarge.2
Docker Version: v18.09.9
Kubernetes Version: v1.20.2
```

K8s中所有应用Pod的规格一致，均为`4vCPUs|8GiB`。

Sermant版本：[`Release v1.4.0`](https://github.com/sermant-io/Sermant/releases/tag/v1.4.0)

### 测试结果
| **被测数据库和场景**       | 达到最大TPS时长（无Sermant/有Sermant/ 差异） | 最大TPS（无Sermant/有Sermant/ 差异） | 平均TPS（无Sermant/有Sermant/ 差异） | CPU占用率增加 | 总内存增加 | 
| ------------------ | -------------------------------------------- | ------------------------------------ | ------------------------------------ | ------------- | ------------ |
| MySQL(查询场景)    | 2s / 5s / +3s                              | 972/ 955.2 / -1.72%                    | 917.4 / 899.2 / -1.98%                    | 1.56%         | 24M    | 
| MySQL(插入场景) | 2s / 3s / +1s                                | 119.1 / 939.7 / +689%                    | 108.7 / 886.9 / +716%                    | 3.26%         | 24M    | 
| MongoDB(查询场景)    | 21s / 23s / +2s                              | 1499.2 / 1500.8 / +0.1%                    | 1453.3 / 1452.3 / -0.06%          | 0.06%         | 23M    | 
| MongoDB(插入场景)     | 19s / 22s / +3s                               | 1054.8 / 1468.4 / +39.2%                    | 1018.9 / 1433.7 / +40.7%       | 0.98%         | 23M    | 
| PostgreSQL(查询场景)    | 11s / 11s / 0                              | 1372.5 / 1402.7 / +2.2%                    | 1290.5 / 1285.9 / -0.36%         | 0.44%         | 22M    | 
| PostgreSQL(插入场景)   | 2s / 5s / +3s                              | 1622.9 / 1619.9 / -0.19%                    | 1473.2 / 1514.7 / +2.8%          | 1.32%         | 22M    | 
| OpenGauss(查询场景)   | 15s / 17s / +2s                              | 1306.2 / 1309.2 / +0.22%                    | 1229.8 / 1230.5 / +0.06%        | 1.77%         | 21M    | 
| OpenGauss(插入场景)  | 7s / 8s / +1s                              | 811 / 817.8 / +0.84%                    | 745.6 / 745.7 / +0.13%                 | 3.44%         | 21M    | 

### 总结
1. 挂载数据库禁写插件后各场景内存增加总量在22M左右，最大TPS、平均TPS下降和CPU占用不超过5%，影响较小。
2. MySQL和MongoDB在插入场景下，执行了禁写策略，不会和数据库进行数据交互，MongoDB和MySQL执行写入数据库耗时较久，且测试Demo写操作占接口方法执行时间比重大，因此TPS增加较多；
   PostgreSQL和OpenGauss写入数据库执行时间较短，同时测试Demo写操作占接口方法时间比重较小，因此写场景下挂载SermantTPS增加不明显。

## xDS服务发现

**Sermant基于xDS协议的服务发现能力性能测试分为两组：**

1. 基线应用性能对比：测试Java应用挂载Sermant，静态场景下的cpu和内存损耗。

2. Sidecar性能对比：测试Java应用在恒定TPS下使用Sermant xDS服务发现能力或Sidecar（Envoy）相较于基线应用的性能对比，包括Pod的CPU占用、内存和服务调用时延指标。测试场景为spring-client应用调用spring-server应用，采集spring-client应用所在Pod的性能指标。

   > 说明：使用 [xds-service-discovery-demo](https://github.com/sermant-io/Sermant-examples/releases/download/v2.0.0/sermant-examples-xds-service-discovery-demo-2.0.0.tar.gz) 作为本次的基准应用进行性能测试

### 第一组测试：基线应用内存对比

#### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为2个，规格如下：

```
规格：通用计算型｜8vCPUs｜24GiB
Docker Version: v18.09.0
Kubernetes Version: v1.21.7
Istio Version：v1.14.3
```

K8s中所有应用Container的规格一致，均为`4vCPUs|8GiB`。

Sermant版本：[`Release v2.0.0`](https://github.com/sermant-io/Sermant/releases/tag/v2.0.0)

#### 测试结果

| 测试场景             | 内存增加总量 | 堆内存增加 | 堆外内存增加 | class增加量 | CPU占用率增加 |
| -------------------- | ------------ | ---------- | ------------ | ----------- | ------------- |
| 挂载Sermant不开启xDS | 13.45M       | 0.45M      | 13M          | 1902        | 0%            |
| 挂载Sermant开启xDS   | 35.37M       | 2.37M      | 33M          | 3837        | 0%            |

#### 总结

1. 宿主服务只挂载Sermant，不开启xDS服务时，Sermant框架共增加13.45M内存。其中，堆外内存中增加1902个class对象
2. 宿主服务挂载Sermant开启xDS服务堆外内存共增加33M，相较于仅挂载Sermant堆外内存增加20M，主要为堆外内存新增1935个class对象，和xDS依赖和grpc依赖相关。同时，我们和主流的无代理服务治理框架进行了参考对比，开启xDS服务时增加的堆外内存属于正常范围。

### 第二组测试：Sidecar性能对比

#### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为2个，规格如下：

```
规格：通用计算型｜8vCPUs｜24GiB
Docker Version: v18.09.0
Kubernetes Version: v1.21.7
Istio Version：v1.14.3
```

K8s中所有应用Container的规格一致，均为`2vCPUs|4GiB`，Envoy 的Container规格同样为`2vCPUs|4GiB`。

Sermant版本：[`Release v2.0.0`](https://github.com/sermant-io/Sermant/releases/tag/v2.0.0)

#### 测试结果

| 测试场景：1000TPS | CPU占用增加 | 内存占用增加 | 平均调用时延增加 | p90增加 | p99增加 |
| ----------------- | ----------- | ------------ | ---------------- | ------- | ------- |
| Sermant           | 3.70%       | 0.05G        | +0ms             | +0ms    | +0ms    |
| Envoy             | 326%        | 0.06G        | +3ms             | +5ms    | +3ms    |


| 测试场景：2000TPS | CPU占用增加 | 内存占用增加 | 平均调用时延增加 | p90增加 | p99增加 |
| ----------------- | ----------- | ------------ | ---------------- | ------- | ------- |
| Sermant           | 2.17%       | 0.06G        | +0ms             | +0ms    | +0ms    |
| Envoy             | 339%        | 0.06G        | +5ms             | +9ms    | +10ms   |


#### 总结

1. 1000TPS下，基线应用挂载Sermant进行服务发现和调用CPU增加3.7%，调用时延没有增长；使用Envoy，CPU占用相比基线应用增加超过300%，平均调用时延增加3ms，p90更是增加了5ms，资源损耗较大。
2. 2000TPS下，基线应用挂载Sermant进行服务调用性能损耗基本和1000TPS场景下持平；使用Envoy时，CPU占用相比基线应用增加达到339%，平均调用时延增加扩大至5ms，p99增加了10ms。随着TPS的增大，使用Envoy的性能损耗持续扩大，尤其是服务调用时延。
3. 通过性能测试对比，Sermant基于xDS协议的服务发现能力无论是CPU损耗还是调用时延等方面性能均远超Envoy。

## xDS路由和负载均衡

**Sermant基于xDS协议的路由和负载均衡配置能力性能测试分为两组：**

1. 基线应用性能对比：测试Java应用挂载Sermant，静态场景下的内存损耗。

2. Sidecar性能对比：测试Java应用在2000TPS下使用Sermant xDS路由和负载均衡能力或Sidecar（Envoy）相较于基线应用的性能对比，包括Pod的CPU占用、内存和服务调用时延指标。测试场景为spring-client应用查询数据库并调用spring-server应用，spring-client使用Sermant的路由插件实现基于xDS服务的路由和负载均衡能力，采集spring-client应用使用不同Http框架时所在Pod的性能指标。

   > 说明：使用 [xds-demo](https://github.com/sermant-io/Sermant-examples/releases/download/v2.2.0/sermant-examples-xds-demo-2.2.0.tar.gz) 作为本次的基准应用进行性能测试

### 第一组测试：基线应用内存对比

#### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为2个，规格如下：

```
规格：通用计算型｜8vCPUs｜24GiB
Docker Version: v18.09.0
Kubernetes Version: v1.23.5
Istio Version：v1.17.8
```

K8s中所有应用Container的规格一致，均为`2vCPUs|4GiB`。

Sermant版本：[`Release v2.2.0`](https://github.com/sermant-io/Sermant/releases/tag/v2.2.0)

#### 测试结果

**基线测试结果：**

| 测试场景 | 内存总量 | 堆内存总量 | 堆外内存总量 | class总量 |
| -------- | -------- | ---------- | ------------ | --------- |
| 基线测试 | 229.933M | 200M       | 29.933       | 9987      |

**对比测试结果：**

| 测试场景             | 内存增加总量 | 堆内存增加 | 堆外内存增加 | class增加量 |
| -------------------- | ------------ | ---------- | ------------ | ----------- |
| 挂载Sermant不开启xDS | 14.98M       | -1.02M     | 16M          | 1948        |
| 挂载Sermant开启xDS   | 42.9M        | 1.9M       | 41M          | 3960        |

#### 总结

1. 宿主服务只挂载Sermant，不开启xDS服务时，Sermant框架共增加14.98M内存。其中，堆外内存中增加1948个class对象
2. 宿主服务挂载Sermant开启xDS服务（包括xDS服务发现、路由和负载均衡服务）堆外内存共增加41M，相较于仅挂载Sermant堆外内存增加25M，主要为堆外内存新增2012个class对象，和xDS依赖和grpc依赖相关。我们和主流的无代理服务治理框架进行了参考对比，开启xDS服务时增加的堆外内存属于正常范围。

### 第二组测试：Sidecar性能对比

#### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为1个，规格如下：

```
规格：通用计算型｜72vCPUs｜144GiB
Docker Version: v18.09.0
Kubernetes Version: v1.23.9
Istio Version：v1.17.8
```

K8s中所有应用Container的规格一致，均为`4vCPUs|8GiB`，Envoy 的Container规格为`4vCPUs|8GiB`。

Sermant版本：[`Release v2.2.0`](https://github.com/sermant-io/Sermant/releases/tag/v2.2.0)

#### 测试结果

**基线测试结果：**

| 测试场景：基线            | CPU占用率（总体占用率百分比） | 平均调用时延 | p90  | p99  |
| ------------------------- | --------- |  ------------ | ---- | ---- |
| HttpClient                | 52.6%  |  3.29ms      | 4ms  | 7ms |
| HttpAsyncClient           | 58%    |  3.6ms        | 5ms  | 8ms |
| jdkHttp                   | 53%    |  3.47ms       | 5ms  | 8ms |
| OkHttp                    | 45.3%  |  2.98ms      | 4ms  | 7ms |

**对比测试结果：**

| 测试场景：Sermant         | CPU占用率增加 | 平均调用时延增加 | p90增加 | p99增加 |
| ------------------------- | ----------- |  ---------------- | ------- | ------- |
| HttpClient                | +1.2%   |  +0.1ms           | +1ms   | +1ms  |
| HttpAsyncClient           | +1.3%  |  +0.05ms         | +0ms    | +0ms   |
| jdkHttp                   | +1.5%    |  +0.07ms         | +0ms | +0ms  |
| OkHttp                    | +1.5%    |  +0.07ms         | +1ms   | +0ms   |


| 测试场景：Envoy | CPU占用率增加 | 平均调用时延增加 | p90增加 | p99增加 |
| --------------- | ------------- | ---------------- | ------- | ------- |
| HttpClient      | +17.7%        | +1.24ms          | +2ms    | +2ms    |
| HttpAsyncClient | +17.8%        | +1.48ms          | +2ms    | +2ms    |
| jdkHttp         | +28.8%        | +2ms             | +3ms    | +4ms    |
| OkHttp          | +21.2%        | +1.18ms          | +2ms    | +1ms    |


#### 总结

1. 2000TPS下，基线应用挂载Sermant进行服务路由额外CPU占用率增加不超过1.5%，平均调用时延增长不超过0.1ms；使用Envoy，CPU占用率相比基线应用额外增加超过15%，平均调用时延增加1ms以上，资源损耗较大。
2. 相较于Envoy，Sermant在额外CPU占用率和平均调用时延方面平均下降超90%，性能显著提升。

## xDs流控

**Sermant基于xDS协议的流控能力性能测试分为两组：**

1. 基线应用性能对比：测试Java应用挂载Sermant，静态场景下的内存损耗。

2. Sidecar性能对比：测试Java应用在2000TPS下使用Sermant xDs流控能力或Sidecar（Envoy）相较于基线应用的性能对比，包括Pod的CPU占用、内存和服务调用时延指标。测试场景为spring-client应用查询数据库并调用spring-server应用，spring-client使用Sermant的流控插件实现基于xDS服务的流控能力，采集spring-client应用使用不同Http框架时所在Pod的性能指标。

   > 说明：
   > 1. 由于触发流控规则时可能会直接返回失败，无法精准测试流控功能对服务调用的影响，因此本次测试虽然下发了流控规则，但测试中不会触发流控规则。
   > 2. 使用 [xds-demo](https://github.com/sermant-io/Sermant-examples/releases/download/v2.2.0/sermant-examples-xds-demo-2.2.0.tar.gz) 作为本次的基准应用进行性能测试

### 第一组测试：基线应用内存对比

#### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为2个，规格如下：

```
规格：通用计算增强型｜16vCPUs｜64GiB
Containeerd Version: v1.7.16
Kubernetes Version: v1.30.4
Istio Version：v1.23.3
```

K8s中所有应用Container的规格一致，均为`4vCPUs|8GiB`。

Sermant版本：[`Release v2.2.0`](https://github.com/sermant-io/Sermant/releases/tag/v2.2.0)

#### 测试结果

**基线测试结果：**

| 测试场景 | 内存总量 | 堆内存总量 | 堆外内存总量 | class总量 |
| -------- | -------- | ---------- | ------------ | --------- |
| 基线测试 | 224.14M | 199M       | 25.14       | 8602      |

**对比测试结果：**

| 测试场景             | 内存增加总量 | 堆内存增加 | 堆外内存增加 | class增加量 |
| -------------------- | ------------ | ---------- | ------------ | ----------- |
| 挂载Sermant不开启xDS | 16.7M       | -0.3M     | 17M          | 2243        |
| 挂载Sermant开启xDS流控   | 57.68M        | 1.68M       | 56M          | 4197        |

#### 总结

1. 宿主服务只挂载Sermant，不开启xDS服务时，Sermant框架共增加16.7M内存。其中，堆外内存中增加2243个class对象
2. 宿主服务挂载Sermant开启xDS服务（包括xDS流控）堆外内存共增加56M，相较于仅挂载Sermant堆外内存增加39M，主要为堆外内存新增1954个class对象，和xDS依赖和grpc依赖相关。我们和主流的无代理服务治理框架进行了参考对比，开启xDS服务时增加的堆外内存属于正常范围。

### 第二组测试：Sidecar性能对比

#### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为2个，规格如下：

```
规格：通用计算增强型｜16vCPUs｜64GiB
Containeerd Version: v1.7.16
Kubernetes Version: v1.30.4
Istio Version：v1.23.3
```

K8s中所有应用Container的规格一致，均为`4vCPUs|8GiB`，Envoy 的Container规格为`4vCPUs|8GiB`。

Sermant版本：[`Release v2.2.0`](https://github.com/sermant-io/Sermant/releases/tag/v2.2.0)

#### 测试结果

**基线测试结果：**

| 测试场景：基线            | CPU占用率（总体占用率百分比） | 平均调用时延 | p90  | p99  |
| ------------------------- | --------- |  ------------ | ---- | ---- |
| HttpClient                | 54.49%  |  3.61ms      | 4ms  | 6ms |
| jdkHttp                   | 55.30%  |  3.64ms       | 4ms  | 6ms |
| OkHttp                    | 51.30%  |  3.48ms      | 4ms  | 6ms |

**对比测试结果：**

| 测试场景：Sermant         | CPU占用率增加 | 平均调用时延增加 | p90增加 | p99增加 |
| ------------------------- | ----------- |  ---------------- | ------- | ------- |
| HttpClient                | +1.67%   |  +0.05ms           | +0ms   | +1ms  |
| jdkHttp                   | +2.34%    |  +0.08ms         | +0ms | +1ms  |
| OkHttp                    | +1.82%    |  +0.1ms         | +0ms   | +1ms   |


| 测试场景：Envoy | CPU占用率增加 | 平均调用时延增加 | p90增加 | p99增加 |
| --------------- | ------------- | ---------------- | ------- | ------- |
| HttpClient      | +22.22%        | +1.13ms          | +2ms    | +2ms    |
| jdkHttp         | +22.26%        | +1.1ms             | +1ms    | +2ms    |
| OkHttp          | +21.94%        | +1.08ms          | +1ms    | +1ms    |


#### 总结

1. 2000TPS下，基线应用挂载Sermant进行流控的额外CPU占用率增加不超过2.4%，平均调用时延增长不超过0.1ms；使用Envoy，CPU占用率相比基线应用额外增加超过20%，平均调用时延增加1ms以上，资源损耗较大。
2. 相较于Envoy，Sermant在额外CPU占用率和平均调用时延方面平均下降超85%，性能显著提升。

## 预过滤启动加速机制

在Sermant 2.0.0 版本开始支持了通过首次运行生成预过滤名单的方式来帮助后续启动时降低启动耗时。该方式可减少启动过程中的字节码增强的类和方法的匹配耗时，大幅缩减启动时间。

### 部署场景

本次测试使用[示例仓库中的demo应用](https://github.com/sermant-io/Sermant-examples/releases/download/v1.4.0/sermant-examples-flowcontrol-demo-1.4.0.tar.gz)来对启动时长进行测试，启动命令为`java -Dspring.cloud.zookeeper.enabled=false -Dagent.service.dynamic.config.enable=false -javaagent:sermant-agent-1.0.0/agent/sermant-agent.jar -jar spring-provider.jar`，关闭所有Sermant核心服务进行测试。

测试分为三组，第一组为被测微服务不挂载Sermant启动，第二组为被测微服务不使用预过滤方式启动，第三组为被测微服务使用预过滤方式启动。另外分别限制不同的规格来验证资源大小对启动时长的影响。分别测试整体JVM启动时长、应用启动时长以及Sermant启动时长以作分析。

### 部署环境

本次测试使用华为云容器引擎CCE进行应用部署，K8s集群的ECS节点数量为2个，规格如下：

```
规格：通用计算型｜16vCPUs｜32GiB｜s2.4xlarge.2
Docker Version: v24.0.9
Kubernetes Version: v1.25.3
```

K8s中所有被测应用的Pod的规格一致，按实验组别分为`1vCPUs|2GiB`，`2vCPUs|4GiB`以及`4vCPUs|8GiB`。

Sermant版本：[`Release v2.0.0`](https://github.com/sermant-io/Sermant/releases/tag/v2.0.0)

### 测试结果

| 规格/场景<br>（单位：s） | 应用启动时长<br>(不挂载Sermant) | JVM启动时长<br/>(不挂载Sermant) | 应用启动时长<br/>(挂载Sermant预过滤关) | JVM启动时长<br/>(挂载Sermant预过滤关) | 应用启动时长<br/>(挂载Sermant预过滤开) | JVM启动时长<br/>(挂载Sermant预过滤开) |
| ------------------------ | ------------------------------- | ------------------------------- | -------------------------------------- | ------------------------------------- | -------------------------------------- | ------------------------------------- |
| `1vCPUs|2GiB`            | 6.668                           | 7.571                           | 12.375                                 | 18.528                                | 7.034                                  | 10.276                                |
| `2vCPUs|4GiB`            | 3.233                           | 3.721                           | 5.476                                  | 10.465                                | 3.116                                  | 5.750                                 |
| `4vCPUs|8GiB`            | 2.445                           | 2.894                           | 4.175                                  | 9.015                                 | 2.557                                  | 5.118                                 |

### 总结

应用整体的JVM启动时长在预过滤开启前受到一定影响，主要耗时开销在于Sermant本身启动过程中的处理耗时以及应用在启动过程中字节码增强的类和方法的匹配耗时。其中前者较为固定，后者受应用加载过程中的涉及类的数量的影响较大。

通过开启预过滤来启动应用我们可以看到，由Sermant带来的额外的JVM启动时长在`4vCPUs|8GiB`时降低超过65%，`2vCPUs|4GiB`时降低超过70%，`1vCPUs|2GiB`降低超过75%。应用自身的启动时长降低超过90%。值得注意的是，资源越小，启动时长越长，通过预过滤方式来启动的优化效果更加明显。
# 浅入浅出Prometheus

## Prometheus产生背景

在单体应用时代，网络拓扑相对固定，部署位置也相关固定。与之类似的则是相对固定的技术栈，相对固定的服务依赖关系。在这种固定的拓扑关系中，数据监控相对而言只有监控指标是变化的，而监控的位置则是固定很少变化。而进入云原生时代后，服务基本都拆分为微服务，通过某种中间件或者通讯协议相互调用。与之带来的变化则是部署位置变化，网络拓扑变化，灵活的技术栈和不定长的调用链。

而众多的变量则要求云原生时代的监控工具需要具有广度和深度。 理想的广度是可以有多个层面的监控，如果操作系统监控，不同产品的监控，网络数据面监控等等。理想的深度则是可以有可用性监控、性能监控、日志监控和自定义监控等等。

业界主流的监控工具有Zabbix、Nagios、Tivoli、SystemCenter、Splunk等工具。这些工具往往在广度和深度方面各有千秋，只能二选一。 Prometheus则相对而言做到了平衡和中庸。Prometheus适合以机器为维度的微服务架构，并且适合采集按照时间序列发生的数据。与此同时，Prometheus作为分布式监控系统，会按照时间序列聚合数据。因此其并不保证数据一定完整，有可能会发生丢失数据的情况。 如果需要一个十分精准的监控系统，prometheus可能并不是最优选择。

## Prometheus要解决的问题

### 监控观测的两种方式

+ RED

    - Rate – 服务每秒请求数
    - Errors – 服务每秒失败请求数
    - Duration – 每个请求消耗时长

+ USE

    - Utilization - 资源利用率
    - Saturation - 系统饱和度
    - Errors - 系统每秒失败请求数

RED侧重于从用户视角的服务级监控， 而USE则侧重于从全局视角监控系统平台。

Prometheus可以支持单独使用RED和USE，也可以支持混合使用


### 需要解决的问题


Prometheus着力要解决的问题是：

+ 有用的监控而不是大而全的监控。 所有指标是用户根据自身业务场景制定的，并不是prometheus自身确定的。
+ 合理的告警策略。通过告警合并优化，只有在需要的时候才发告警，非必要不会发。
+ 高可用。监控系统作为最后一道防火墙，业务系统挂掉其也不能挂。因此Prometheus设计的及其简单，几乎没有依赖。因此有很高的可用性。

## Prometheus设计上的优缺点

+ 优点
    - 极简的设计风格
    - 插件体系
    - 不需要使用代理
    - 极高的采集性能
    - 支持分区采样和联邦部署
+ 缺点
    - 不支持长期存储、异常检测、自动水平缩放和用户管理
    - 不支持日志
    - 没有官方的Dashboard(grafana可以替代)
    - 可能会丢失数据(不要计算敏感数据)

## 最佳实践和典型案例

### 设计图

![](https://tva1.sinaimg.cn/large/008i3skNly1gy62w9nzt6j30xy0ji40o.jpg)

### 主要组件

#### Prometheus Server
> Prometheus主服务，收集并存储时间序列数据，对外提供PromQL查询支持

#### 主要功能

+ 负责对外提供管理API
+ 计算并存储时序数据
+ 定时从采集端拉取数据。
+ 计算告警数据

#### 运行流程

![](https://tva1.sinaimg.cn/large/008i3skNly1gy62y43q7mj30xu0fggn3.jpg)

+ Server从配置文件(服务发现)中读取Exporter信息(endpoint数据)。
+ 按照设定好的采集间隔从Exporter拉取监控指标数据。
+ 对拉取到的离散监控指标数据进行聚合运算(减少数据存储量)，并持久化到本地磁盘。
+ 按照设定好的配置数据计算告警
+ 如果有命中的告警数据，则发送到AlertManager进行处理。

#### 监控源

在Server中将每个独立可提供监控数据的称之为Instance， 多个Instance逻辑构成为Job。
例如:
```yaml
- job: api-server
    - instance 1: 1.2.3.4:5670
    - instance 2: 1.2.3.4:5671
    - instance 3: 5.6.7.8:5670
    - instance 4: 5.6.7.8:5671
```

Server在采集数据是，对同一个Job使用相同的策略，例如数据转换、告警规则等等。换言之，Job就等同于逻辑单元，instance等同于单元中的实体。


### Prometheus Exporter
> 提供服务监控指标，当Proemtheus Server拉取时提供数据

![](https://tva1.sinaimg.cn/large/008i3skNly1gy62zg0dbgj30xy0i2wfq.jpg)

#### 监控指标类型
+ Counter 持续累加的计数器，常用于计算HTTP报错数量
+ Gauge 可任意变化的数值，常用于离散值，例如CPU利用率，内存使用率等
+ Histogram 区间分布值，常用于统计采样值在某些区间内的分布情况。
+ Summary 一段时间内的分位图，常用于解决百分位问题。

#### 监控指标结构

格式为:`<metric name>{<label name>=<label value>,...} Value `例如: `http_requests_total{method="POST",endpoint="/api/tracks"} 2`

每个监控指标都有如下三个属性：

+ Metrics Name  监控名称
+ Label  每个标签表示一个时间序列。 http_request_total{method="Get"}是一个时间序列。 http_request_total{method="Post"}是另外一个时间序列。
+ Value Float64的数据类型，用来表示时点数据

#### API
每个Exporter需要提供一个可供Prometheus Server读取数据的API，API返回的数据需要满足监控指标结构

例如请求API，返回如下数据:
```
api_http_requests_total{method="POST", handler="/messages"} 2
api_http_requests_total{method="GET", handler="/messages"} 3
```

#### PushGateway
> 临时性Job推送，用于保存短暂性监控数据。和Prometheus Server无法触达的Exporter

如果某些服务不能被Prometheus Server之间拉取数据，则可以将数据推送到PushGateway。然后Prometheus Server从PushGateway拉取数据，借此完成监控数据计算的工作。
主要适用的场景有:
+ 旧服务无法通过改造提供API。
+ 因为网络问题Server无法获取数据。
+ 服务有自己的计算规则，不希望Server定时拉取。

#### AlertManager
> 接受来自于Prometheus Server产生的告警数据，并对外产生有效告警

![](https://tva1.sinaimg.cn/large/008i3skNly1gy6322yhocj30ug0u0mz6.jpg)

AlertManager主要用来管理告警。当Server计算出告警事件后，会将此事件通知AlertManager，而后AlertManager通过预制规则发送告警信息。

Prometheus Server仅仅负责计算是否有满足告警的数据， 而AlertManager则用来决定是否需要发送告警。

一个报警信息在生命周期内有下面3中状态：
+ inactive: 表示当前报警信息既不是firing状态也不是pending状态
+ pending: 表示在设置的阈值时间范围内被激活了
+ firing: 表示超过设置的阈值时间被激活了

当刚刚出现告警信息时，此时处于Pending，表示有告警发生但还达到发送条件。 当持续多次出现后，则发送告警并进入Firing状态。

当告警被处理并消息后，状态由Firing轮转为Inactive状态。

Server在计算是否满足告警规则时，有两个需要关注的周期：
+ 拉取周期(scrape_interval)
+ 评估周期(evaluation_interval)

Server在每个拉取周期中计算监控指标数据。 但是否满足告警规则，则是在评估周期内完成的。 只有连续两个评估周期都得出满足告警这个结论，才会真正的认为需要发送告警。
假设:
```
    scrape_interval：20s
    evaluation_interval：1m
```
告警规则为：当持续1m满足Load>20时发送告警

![](https://tva1.sinaimg.cn/large/008i3skNly1gy633bl4snj30xu0c20tl.jpg)

在第一个拉取周期(20s)中，server计算出当前Load < 20，因此此时认为不满足告警条件。
连续3个拉取周期内，Load > 20。等到下个评估周期的时候，Load >20则认为可能需要告警，告警状态转变为pending。

再经过连续3个拉取周期，开始第二轮评估周期，此时Load >20成立，同时上一个评估周期也满足 Load >20则告警状态转变为firing。

Server向AlterManager发送告警事件。

 ### 最佳实践

#### 使用Labels
将所有的监控维度都添加Label，通过PromQL进行聚合操作。

#### 不能过度添加Label
每个监控指标需要根据业务场景添加合适的Label，但不能滥用。 每一个Laebl会创建一个时间序列，不要一开始就创建非常多的Label，最好根据业务场景慢慢添加。

#### Counter/Gauge
如果数据可能会上升/下降，则是Gauage。反之则是Counter。

#### 面向丢失数据监控
Prometheus不承诺不丢失数据。 所以在做监控预测时，需要容忍数据丢失的场景。
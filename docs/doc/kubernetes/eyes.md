# 浅入浅出 Kubernetes 流量分析

Kubernetes流量大致分为以下四种:

|方式|发起方|访问方式|
|---|-----|------|
|A|集群外|Ingress|
|B|集群外|3层路由|
|C|集群内|svc|
|D|集群内|3层路由|


本文首先从A(集群外通过Ingress访问集群服务)开始聊Kubernetes网络。

方式A的网络拓扑大致如下:

![k1.png](../../pic/doc/kubernetes/k8s-01.png)

为了简化模型，假设服务存在于Kubernetes其中一台节点上。外部流量经过LB(4层)后，被转发到Ingress节点(假设使用Nginx Ingress)中。

Ingrss节点收到流量请求之后，按照Http协议进行分析，解析出Host和Uri。 而后根据节点中内置的Iptables规则将流量转发到对应的目标节点之上。

目标节点接受到流量请求之后进行业务逻辑处理，而后将响应流量返回给来源节点。

来源节点接受到服务响应流量之后，同步返回给下游LB。 下游LB将流量返回给调用方。


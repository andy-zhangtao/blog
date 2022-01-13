# 通过Operator在集群中实现CI

## 何为Operator？
Kubernetes Operator 是另外一种封装、部署和管理 Kubernetes 应用的方法。我们和kubernetes交互一般有两种方式：kubectl和API。

Operator可以扩展kubernetes API，允许在API中添加我们自己的逻辑。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyc09gk43nj313g0lg403.jpg)


+ 从实现方式上面来说， Operator就是告诉api-server一种资源分发规则：`如果有一个资源(自定义资源CRD)N的请求，那么就转发给特定的控制器C`。
+ 而本质上，`Operatore = CRD + Controller`  在这个等式中，CRD是我们新增加的资源， Controller则是专门处理这个资源的控制器。 每一个controller启动的时候，会通过list-watch机制向api-server订阅特定CRD的事件。

而在Controller中，当没接受到一个事件后，就表示用户发来了一个新的`期望状态`，Controller就会不断的判断集群中特定资源`当前状态`和用户的`期望状态`是否一致。 如果不一致，就会不断的进行调整，直到两个状态达到一致。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyc0cixr2pj312w0fuq44.jpg)

所以Operatore简而言之，就是一个简单的循环，这个循环只做一件事，确保某个资源的期望状态和当前状态保持一致。


## 如何使用Operatore实现CI

回到CI这个案例中来， 如果需要使用Operator来实现，那么需要我们做三件事：

1. 将CI抽象出一个资源
2. 获取当前资源状态和用户期望状态
3. 对两种状态进行调整

### 流程图

![](https://tva1.sinaimg.cn/large/008i3skNly1gyc2gtfqxij30o50bw0sy.jpg)

1. 用户提交修改资源状态的yaml文件。
2. api-server接收到资源修改请求以后，判断是集群内置资源还是用户自定义资源(CRD)。
3. api-server将CRD资源处理请求转发到controller。
4. controller判断是否当前是否有相同的任务在构建，如果没有就在Kubernetes集群中创建构建Job任务。

> 为什么需要创建Job任务？
>
> 我们都知道kubernetes有五种任务类型：
>
> + Deployment (适合long-running类型的作业)
> + Daemonset (适合每个节点都运行的作业)
> + StatefulSet (适合需要状态保持的，long-running作业)
> + Job (适合一次性执行的，具有明确结束状态的作业)
> + CronJob (适合定时执行的，并且具有明确结束状态的作业)
>
> CI构建属于一次性执行的任务，不是long-running类型的任务。 所以使用Job是最合适的。

其中最重要的一步就是状态合并处理：
![](https://tva1.sinaimg.cn/large/008i3skNly1gyc2mwk08mj30oi0diaag.jpg)

controller通过listwatch向api-server订阅，CR(CR和CRD不同)变更事件。  当用户提交CR变更后，api-server会将新状态转发都controller。 controller需要从local store中取出当前状态，然后自行实现状态合并(就是资源逻辑处理)。

然后将最新合并的状态提交到集群中。


**上面是理论分析，如果不想看源码到此为止就可以了。 如果还想继续看源码，请往下继续看**

### 代码实现

下面代码使用`kubebuilder`实现。 使用`kubebuilder`时遵循`KB四范式`：

1. 定义API
2. 定义types
3. 实现controller
4. 调试发布


#### 定义API

本地安装好Kubebuilder以后，先创建一个目录作为以后的工程根目录，同时使用`go mod init`初始化golang project环境。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyc2zr6oxfj30aq05e0sn.jpg)

然后开始定义Operator工程:

![](https://tva1.sinaimg.cn/large/008i3skNly1gyc33vrthoj30kg0523yl.jpg)


最后就可以定义API了

![](https://tva1.sinaimg.cn/large/008i3skNly1gyc34tfebgj30m8058q30.jpg)

每一项资源都需要遵循 api规范: /group/version/kind. 当定义好，在api-server中会保存成下面的格式：

![](https://tva1.sinaimg.cn/large/008i3skNly1gyc38ewnepj310i06ejrm.jpg)

其中ci是定义的group， v1是版本，中间的zt.vapi.cc则是最开始定义的domain。 这三项组合起来就可以确保 CR的唯一性。 当我们调用CR时，apiVersion就需要写成下面的值：

![](https://tva1.sinaimg.cn/large/008i3skNly1gyc3aby2npj30no0aw3z2.jpg)

#### 修改types

在`api/v1/api/v1/build_types.go`中，声明我们CR的Spec和Status两个结构体：
![](https://tva1.sinaimg.cn/large/008i3skNly1gyc3c3u48pj310q0j441y.jpg)

在设计属性的时候，尽量保持精简。 例如name字段，在`metav1.ObjectMeta`中已经有了，在Spec中就没必要再定义一次了。

其次需要根据需求设计属性是否是必填项， 是否为必填项由`json的omitempty`决定。 如果添加了omitempty，表示此项可忽略，api-server校验的时候就会允许其为空，反之则不允许其为空。根据这个原则，我们就可以决定每个属性是否为空。

#### 实现controller

修改controller只需要在`controllers/build_controller.go`中实现就可以。在controller中也有一些范式：

+ 获取用户上传的资源期望状态
+ 获取当前集群中的资源状态
+ 计算最终状态
+ 提交到集群中

1. 来看`获取用户上传的资源期望状态`：
![](https://tva1.sinaimg.cn/large/008i3skNly1gyc3ja2o4zj30u608kgmn.jpg)

如果可以查询到值，那么err就是nil，反之根据情况判断是不是要返回错误。 如果忽略错误，那最终返回的err为nil。

当获取到值后，这个build就是用户提交的资源期望状态。我们就开始执行第二步:

2. 获取当前集群中的资源状态：

![](https://tva1.sinaimg.cn/large/008i3skNly1gyc3mby7jfj317i0eqq4m.jpg)

`job.Name`和`job.Namespace`是根据需求赋值的，在最后完整代码中可以看到实现。

如果`found`有值，就说明集群中有此项资源，反之就说明没有这个资源。如果没有资源，直接创建就可以了。 如果有资源，那么需要计算最新状态。

3. 计算最终状态

针对CI任务来说，最终状态是创建一个运行构建任务的Job。所以如果现在存在一个正在运行构建的任务，那就删除后新创建一个新的Job。

为什么需要删除后在创建呢？

这里计算的是最终状态，用户提交的期望状态是构建最新的代码，如果当前存在一个正在构建旧代码的Job。那么此时的状态就是:

```
当前构建"旧"代码 != 期望构建"新"代码
```

所以我们需要删除旧的Job任务，然后再创建一个新的Job任务。


![](https://tva1.sinaimg.cn/large/008i3skNly1gyc3s8rkjsj315w0f2tan.jpg)

4. 调试发布

本地调试推荐使用`minikube`。  在构建的时候需要修改两个地方：

1. 删除makefile中的docker-build目标中的test依赖
2. 注释Dockerfile中的`RUN go mod download`


### 附录

+ 完整代码

> build_types.go

![](https://tva1.sinaimg.cn/large/008i3skNly1gyc3vsqa6zj30u00y078x.jpg)

> build_controller.go

![](https://tva1.sinaimg.cn/large/008i3skNly1gyc3x4cbqvj30u01nzagx.jpg)


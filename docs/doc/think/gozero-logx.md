# 关于 go-zero logx 的一些思考

1.3.5 版本中的 go-zero logx 使用分为两阶段，初始化阶段和运行阶段。 在初始化阶段主要用于参数初始化，此阶段设置的数据将影响运行时的数据输出方式，如果有自定义需求也应该在此阶段设置。 在运行阶段，会根据实际需求将内容输出到特定目标。

go-zero 默认有两类 logger 和三个 writer。 其中 Logger 属于组合关系，三个 writer 各自场景不同。而 concreteWriter 是适用范围最广的一类 writer。当需要实现自定义 writer 时，一定要注意不要串行输出内容，尽可能利用缓存或者使用 chan 并行输出内容。

在 go-zero(1.3.5)中的 logx 整体流程。

![](https://p.ipic.vip/rwp0wp.png)

logx 在 go-zero 中分为两个阶段，init pharse 和 runtime pharse。 两个阶段各有侧重，init pharse 用于根据配置文件中的参数设置进行 logger 和 writer 的初始化。 runtime pharse 用于根据需求输出相对应格式的内容。

## Init pharse

运行且仅运行一次
和 Log 有关的配置参数有两部分： ServiceConf 和 LogConf 。 其中 ServiceConf 存在于 RestConf，是 LogConf 配置入口。

LogConf 是 Log 的具体配置，可以用来设置服务名称，日志级别，输出模式，时间戳和压缩格式。

如果需要设置自定义日志输出方式，可以通过 middleware 方式进行劫持设置。

## Runtime Pharse

根据设置参数进行日志输出。 此阶段无法修改参数。
在 logx 中有两类结构体，logger 和 writer。 logger 用于控制输出哪些字段， writer 用于控制输出方式(终端、file 等)。

logger 默认有两个实现，durationLogger 和 traceLogger。 DurationLogger 会在其他字段基础上多一个"duration"字段。TraceLogger 会在其他字段基础上增加 duration、trace 和 span 三个字段。

DurationLogger 和 TraceLogger 是组合关系，并非二选一。如何自定义增加/删除字段？

当前存在三个 Writer， concreteWriter、mockWriter 和 nopWriter。 三个 writer 有各自适用场景。

mockWriter 用于单元测试。 nopWriter 用于禁止输出日志时的"black hole"。剩余场景均使用 concreteWriter。

concreteWriter 可以对接多种具体的输出方式(plain text、Json 和 console、file 的组合)。

当以上均不满足需求时，可以通过 customWriter 的方式进行自定义设置。在使用自定义 writer 时，需要满足 writer interface{}所定义的函数。同时需要进行“抢先占位”。

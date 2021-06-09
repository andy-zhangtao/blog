# TCP/IP 可视化工具
## I/O Graph
> I/O graph用来显示流量的整体情况

I/O Graph位置在Statistics – IO Graphs

![](https://yx-prod-resources-shared-1254112465.cos.ap-beijing.myqcloud.com/1/78f7f29f329734ab0af14a8496f38e49-275819)

为了方便了解I/O  Graph的使用，首先来依次了解其主要属性。
1. Graph名称，在I/O  Graph中允许同时创建多个图(每个图就是一种计算规则)。这里标记每个图的名称，当有多个图时，通过名称和颜色来区分。
2.筛选条件。 每个图通过设定筛选条件来定向展示某些数据。 默认是全部packets。  假设需要展示错误的tcp流量，就输入tcp.analysis.flags(使用wireshark筛选语法)
3. 展示图形类型。
4. Y轴计算规则。默认显示所有的packets， 可以选择COUNT、SUM等等函数。 这些函数是对5(Y - Field)的计算。
5. 需要应用到4(计算规则)的指标。例如填入frame.time_delta(当前帧与上一帧之间的时间间隔)。
6. 移动平均值。 类似于滑动窗口的平滑规则。
7. Y轴，注意不同的计算规则有不同的单位
8. X轴，时间。 可以是相对时间，也可以是实际时间。

### 函数使用
+ SUM(*) – 在刻度间隔内为所有实例添加并绘制字段的总和
+ MIN(*) – 在刻度间隔内计算所有实例中的最小值
+ AVG(*) – 在刻度间隔内计算所有实例中的平均值
+ MAX(*) –在刻度间隔内计算所有实例中的最大值
+ COUNT(*) – 在刻度间隔内实例出现的次数
+ LOAD(*) – 未知

### 最佳实践
+ Filter和Y Field组合使用。 相当于 select (Y Field) from * where filter
+ 合理使用函数。相当于select f(Y Field) from * where filter.
+ 使用多个Filter组合展示数据。 相当于 select xxx join select xxx join xxxx
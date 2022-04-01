# 在Grafana中制作环比视图的方式
> 以下文档基于Echarts,如果看不懂下面的内容，可以先看[前面一篇文章](https://blog.devexp.cn/#/doc/prometheus/useecharts-in-grafana)


## 同比视图的基础知识

再描述技术实现之前，我们先简单说一下常用的两个统计比较术语。

+ 环比: 与上个统计周期相比的变化比. 例如，2022年2月份与2022年1月份相比较称其为环比.
+ 同比: 与同时期相比的变化比。 例如，2022年2月份与2021年2月份相比较称其为同比。


实际上同比和环比都是变化率的概念，主要用于快速判断数据变化趋势。 在做环比和同比时，有个最重要的基础: **统计口径必须保持一致** (这个问题在后面会体现出来，这里先挖个坑，后面再填)。

## 实现过程

我们先来看一下测试数据，如下所示:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0t7tuwmojj20ha16gwl3.jpg)

`t`表示区分项，`cn`表示值。 从图中可以看出，我们最终想比较的是2022-03-30和2022-03-31这两天的数据差异。 将这两天的数据放在同一张图中，比较的时候就很形象直观了。

下一步再Grafana中创建`panel`,配置好数据源，此时此刻如果选择默认的`Time series`，就会发现测试数据会平铺展示。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0t82ae2l5j22cg0m6n0t.jpg)

这种是默认展示，适合按照时间序列展示数据，查看趋势。但不利于数据分析和比较。

如果我们选择`Echarts`, 会发现仍然是平铺展示。
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0t84c4jgaj22da0m0adi.jpg)

`Echarts`默认只是优化图形展示，并没有提供其他功能(如果提供了，也就没有这篇文章啥事了)。 下面，我们就需要开动脑筋，通过改造`Echarts`来满足环比需求。在[前面一篇文章](https://blog.devexp.cn/#/doc/prometheus/useecharts-in-grafana)中，我们说过，整合`Echart`时，最重要的两件工作: `确定X轴分类数据`和`确定Y轴数据源`。

> `确定X轴分类数据`用于渲染X轴刻度值，一般显示的就是时间。 
> `确定Y轴数据源`显示X轴分类点位上面的数据

在`Echarts`中，描述X轴的属性是: `xAxis`。 其中有两个重要属性:
- type
  + 有四个值，常用的是`category`，`category`表示的是离散数据。当使用`category`时，数据会从`data`中获取。如果`data`为空，那么就会从`series`中获取。
- data
  + X轴的分类数据，可以为空。如果它为空，那么`series`就不能为空。


在前面一篇案例中，我们演示的是`xAxis`使用data的方案。 在本篇中，我们演示不使用data的方案。 

我们先把`Echarts`的处理模型放上来，后面会频繁的使用到。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0tz0g6a05j20yc0f20v7.jpg)


既然`xAxis`不需要包含`data`，就不需要改造`数据渲染区域`了(上篇中，我们通过引入两个单独的数组保存X轴和Y轴数据，此篇中保留此区域不动)。

但需要对X轴数据进行`聚合操作`。也就是说，需要将X轴数据中的年月日数据剔除掉。 例如：将`2022-03-30 01:00:00` 变成`01:00:00`。 而这么做的原因是做环比分析的时候，需要有统一的基准线。

因为环比是不同统计周期内相同维度的数据比较，所以统计维度必须一致。例如下图所示：
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0tz8r68ptj20h406ewen.jpg)

假设在一个统计周期，有A,B,C,D,E五个统计维度。 那么环比就是比较当前周期的五个维度值和上个周期的五个维度值的差异。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0tzb5w8juj20mc0euwfl.jpg)

如果我们不将年月日剔除掉，那么X轴的维度值就无法统一了。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0tzex55qoj20w60b6abe.jpg)

而这就是刚才我们看到数据平铺展示的原因。 

当我们把年月日剔除掉以后，就只剩下时分秒了。 时分秒无论放在哪天都是唯一的，所以就可以保持维度统一了。 

### 如何剔除年月日呢？

剔除年月日有两种方式:

+ 数据源修正
+ 渲染源修正

数据源修正指的是，使用grafana通过sql语句查询数据的时候，在sql语句层面将日期转换成`HH:MM:SS`的格式。例如使用`clickhouse`时的SQL语句:

```sql
SELECT
    formatDateTime(toStartOfHour(create_time),'%H:%M:%S')  as t,
    .....
from  
    .....
```

渲染源修正指的是，在`Echarts`代码中通过js将数据转换成DataTime类型，然后输出`HH:MM:SS`的格式. 例如:
```js
  data.series.map((s) => {
    const sData = s.fields.find((f) => f.type === 'number').values.buffer;
    const sTime = s.fields.find((f) => f.type === 'time').values.buffer;
		
    sData.map((d, i) => {
        var date = new Date(sTime[i]);
        var hours = date.getHours();
        var minutes = "0" + date.getMinutes();
        var seconds = "0" + date.getSeconds();
        var formattedTime = hours + ':' + minutes.substr(-2) + ':' + seconds.substr(-2);

        // formattedTime 是转换成HH:MM:SS后的数据
        .....
    })
  }
```

统一X轴维度数据后，我们需要分别保存当前周期和环比周期的数据。 获取当前周期和环比周期也有两种方式：

+ 通过一条SQL语句查询出两个周期数据
+ 通过两个SQL语句分别查询出两个周期数据

我倾向于`通过两个SQL语句分别查询出两个周期数据`这种方式，因为这种方式可以任意插拔，任意开启，对代码逻辑破坏性最低。而通过这种方式数据的时候，需要标识哪个是环比，哪个是当前。 例如下面的例子:

```sql
SELECT
    t,
    cn "环比"
FROM default.temp_compare
WHERE xxxx
```

这样标识数据以后，在`data.series.map`中就可以通过`s.name`来区分数据了。 区分逻辑如下:
```js
    if (s.name.includes('环比')) {
      sData.map((d, i) => {
        var date = new Date(sTime[i]);
        var hours = date.getHours();
        var minutes = "0" + date.getMinutes();
        var seconds = "0" + date.getSeconds();
        var formattedTime = hours + ':' + minutes.substr(-2) + ':' + seconds.substr(-2);

        if (d) {
          old.push([formattedTime, d.toFixed(0)])
        } else {
          old.push([formattedTime, 0])
        }
      })
    } else {
      sData.map((d, i) => {
        var date = new Date(sTime[i]);
        var hours = date.getHours();
        var minutes = "0" + date.getMinutes();
        var seconds = "0" + date.getSeconds();
        var formattedTime = hours + ':' + minutes.substr(-2) + ':' + seconds.substr(-2);

        if (d) {
          current.push([formattedTime, d.toFixed(0)])
        } else {
          current.push([formattedTime, 0])
        }
      })
    }
```
将环比数据存放在old数组中，将当前数据存放在current数组中。通过这两个数组分别保存X轴的数据源，同时顺带也保存了Y轴数据。

最重要的数据源处理完以后，就可以进入渲染环节了，也就是处理处理模型的第三部分。 默认的`Echarts`是平铺展示类型，所以只有一组数据源。 现在有两组数据源，所以需要修改`series`，添加两组:
```js
series: [
  {
    name: '当前',
    type: 'line',
    data: current,
  },
  {
    name: '环比',
    type: 'line',
    data: old,
  }
],
```

这样就可以显示当前和同比数据了。 此时两组数据会在同一组X轴和Y轴上面叠加展示:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u0y3bp8bj21di0cymyz.jpg)

下面我们优化一下数据展示，分别为`环比`和`同比`两组数据指定X轴和Y轴坐标。 

```js
xAxis: [
  {
    type: 'category',
    boundaryGap: false,    
  },
  {
    type: 'category',
    boundaryGap: false,
  }],
yAxis: Object.assign(
  [{
        type: 'value',
      },
       {
        type: 'value',
      },
  ],
  axisOption
  ),
```
通过将X轴和Y轴由对象改为数组(弱类型就是好，类型可以随便调整🐶)，然后在数组中添加X轴和Y轴的定义，就可以创建两套坐标系。 最后，在series中为每个数据源指定坐标系:

```js

series: [
  {
    name: '当前',
    type: 'line',
    data: current,
    xAxisIndex: 0,
    yAxisIndex: 0,
  },
  {
    name: '环比',
    type: 'line',
    data: old,
    xAxisIndex: 1,
    yAxisIndex: 1,
  }
],

```

双坐标系效果如下:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u1e1zobaj21qy0egtbq.jpg)


## 总结

通过两个效果来看，使用双坐标系时，会产生X轴偏移的情况(从图中可以看出09:00并不是预期的一条线，而是两条线。这是因为两个点出现了偏移)，所以适合数据差异比较大的场景。当数据差异不大时，两边的Y轴数据也会出现一些刻度不均等的情况，从双坐标图也可以看出当前数据 > 环比数据，但当前数据点却低于环比点。 原因就是环比Y轴刻度值比稀疏，造成渲染点偏高。
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u1jdhwxyj21zq0ek41x.jpg)

最后总结一下环比/同比数据表的要点:

1. 通过SQL语句剔除差异项，确保X轴分类维度保持一致
2. 通过两条SQL抽取不同数据源
3. 修改渲染逻辑，将series变为数组
4. 使用唯一坐标系可以更形象直观显示数据。
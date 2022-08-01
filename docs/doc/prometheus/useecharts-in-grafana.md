# 在 Grafana 中使用 Echarts 的方式

> 以下方式适用于 grafana 8 及以上版本，低于 8.0 的版本未亲自验证

本文写于北京 2022 年 3 月，两会并且疫情反复之际。 小区旁边的儿童游乐场不幸中招，不知道又有多少个小朋友会因此隔离。为了转移注意力，故强迫自己学习一直不愿意钻研的 Echarts 图表。

## Grafana 引入 Echarts 的原理

Grafana 一直以来都是云原生领域中做图表展示的扛把子。 其内置的图表已经可以满足大部分场景了，但说实话，Grafana 的图表使用率最高的莫过于 Line(折线图)了。 其他图适用场景有限，不如折线图使用广泛。

但即便如此，折线图有时也无法适用所有场景。 有时我们想使用第三方图表，却受限于 Grafana 有限的 Plugins 机制造成无法低成本引入。 好在，Grafana 8 官方支持了 Echarts 图表。

Grafana 支持 Echarts 图表原理不难理解，Grafana 通过 Query 从数据源查询数据，然后将数据作为`function (data, theme, echartsInstance, echarts) {}`的参数传入到一段 js 代码中。 这个函数需要返回 echarts 实例，grafana 再使用返回的这个实例进行 UI 渲染。

也就是说，Grafana 引入 Echarts 的原理大致如下:

```javascript
// data 是query查询到的数据
function (data, theme, echartsInstance, echarts) {
    // logic code

    return {
        // echarts instance
    }
}
```

我们如果想使用 echarts，那么只需要完成两件事：

1. 对`data`进行数据整合处理
2. 返回`echarts`实例对象

## 数据模型

为了尽可能简单描述问题，我们以[`基础折线图`](https://echarts.apache.org/examples/zh/editor.html?c=area-basic)为例。
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h02ur24cd7j20fg0c2mx9.jpg)

最终效果如下:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h02uxk592dg22ae0hehdt.gif)

为了实现这样的效果，我们首先来看一下数据结构。 再原始数据库中保存的数据如下所示：

| time       | month      | value |
| ---------- | ---------- | ----- |
| 2021-10-01 | 2021-10-01 | 13000 |
| 2021-11-01 | 2021-11-01 | 14300 |
| 2021-12-01 | 2021-12-01 | 13500 |
| 2022-01-01 | 2022-01-01 | 12500 |
| 2022-02-01 | 2022-02-01 | 15500 |
| 2022-03-01 | 2022-03-01 | 13800 |

`time`是`datetime`类型，用来筛选时间段。 `month`是 x 轴用来进行分类的字段，`value`则是最终需要显示的值。

`query`语句可以这样组织:

```sql
SELECT
    (intDiv(toUInt32(date), 10800) * 10800) * 1000 as t,
    month,
    max(value)
FROM default.grafana2

WHERE date >= toDateTime(1631111854) AND date <= toDateTime(1646750254)

group by t, month
ORDER BY t
```

如此这般操作之后，grafana 就会从数据库中查询到数据。 然后 grafana 需要将数据作为参数放入 echarts 函数中。

那么在 echarts 中，data 应该是什么样的呢？

我们在函数中，输出一下 data：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h02v7y49xpj20uj0u0428.jpg)

可以看到，在 series 中保存的便是我们在数据库中的数据。 其中 name 是我们用来准备做 X 轴分类展示的 month，而 fields 则保存的是每个时间点所对应的具体 value。

通过观察也可以看到，在 value 的数组中，只有相对应的时间戳中才会出现值。也就是说第一组是`[13000, null, null, null, null]`，第二组是`[null, 14300, null, null, null]`

而我们准备实现的 echarts 图表需要的数据是这样的：

```json
{
  "xAxis": {
    "type": "category",
    "boundaryGap": false,
    // data 是X轴数据
    "data": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"]
  },
  "yAxis": {
    "type": "value"
  },
  "series": [
    {
      // data 是Y轴数据
      "data": [820, 932, 901, 934, 1290, 1330, 1320],
      "type": "line",
      "areaStyle": {}
    }
  ]
}
```

所以我们需要将 grafana 传入的 data 数据进行拆分，转化成 echarts 可以识别的数据。

## 逻辑处理

grafana 的数据是耦合在 data 对象里面的，所以需要遍历 data 分别取出 X 轴和 Y 轴的数据。代码如下:

```javascript
let id = [];
let value = [];
const series = data.series.map((s) => {
  const sData = s.fields.find((f) => f.type === "number").values.buffer;
  const sTime = s.fields.find((f) => f.type === "time").values.buffer;

  id.push(s.name);
  sData.map((d, i) => {
    if (d) {
      value.push(d);
    }
  });
});
```

`id`保存的是 X 轴数据， `value`保存的是 Y 轴数据。 根据 echarts 的要求，就可以分别传入 X 轴和 Y 轴了。

```javascript
return {
  xAxis: {
    type: "category",
    boundaryGap: false,
    data: id,
    tooltip: {
      show: true,
    },
    axisPointer: {
      type: "shadow",
    },
  },
  series: [
    {
      data: value,
      type: "line",
      seriesLayoutBy: "row",
      label: {
        show: true,
        position: "top",
      },
      areaStyle: {},
      emphasis: {
        focus: "series",
      },
    },
  ],
};
```

通过这样已经可以实现 echarts 的折线图了。 但我们还可以继续再往前走一走。 参考 echarts 的[参数说明](https://echarts.apache.org/examples/zh/editor.html?c=area-basic)
可以添加 tooltip、toolbox 的 feature、legend 等。 完整配置如下:

```javascript
let id = [];
let value = [];
const series = data.series.map((s) => {
  const sData = s.fields.find((f) => f.type === "number").values.buffer;
  const sTime = s.fields.find((f) => f.type === "time").values.buffer;

  id.push(s.name);
  sData.map((d, i) => {
    if (d) {
      value.push(d);
    }
  });
});

return {
  darkMode: "auto",
  color: "#0f8ad1",
  tooltip: {
    trigger: "axis",
    axisPointer: {
      type: "cross",
      crossStyle: {
        color: "#999",
      },
    },
  },
  grid: {
    left: "3%",
    right: "4%",
    bottom: "3%",
    containLabel: true,
  },
  toolbox: {
    feature: {
      dataView: { show: true, readOnly: false },
      magicType: { show: true, type: ["line", "bar"] },
      restore: { show: true },
      saveAsImage: { show: true },
    },
  },
  xAxis: {
    type: "category",
    boundaryGap: false,
    data: id,
    tooltip: {
      show: true,
    },
    axisPointer: {
      type: "shadow",
    },
  },
  yAxis: {
    type: "value",
  },
  legend: {
    data: id,
  },
  series: [
    {
      data: value,
      type: "line",
      seriesLayoutBy: "row",
      label: {
        show: true,
        position: "top",
      },
      areaStyle: {},
      emphasis: {
        focus: "series",
      },
    },
  ],
};
```

## 事后总结

Grafana 引入 echarts 的关键在于数据转换。只要将 grafana 的数据剥离开，分成 X 轴和 Y 轴数据，然后按照 echarts 的规范赋值就可以完成引入了。

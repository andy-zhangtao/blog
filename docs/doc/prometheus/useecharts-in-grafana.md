# 在Grafana中使用Echarts的方式
> 以下方式适用于grafana 8及以上版本，低于8.0的版本未亲自验证

本文写于北京2022年3月，两会并且疫情反复之际。 小区旁边的儿童游乐场不幸中招，不知道又有多少个小朋友会因此隔离。为了转移注意力，故强迫自己学习一直不愿意钻研的Echarts图表。


## Grafana引入Echarts的原理

Grafana一直以来都是云原生领域中做图表展示的扛把子。 其内置的图表已经可以满足大部分场景了，但说实话，Grafana的图表使用率最高的莫过于Line(折线图)了。 其他图适用场景有限，不如折线图使用广泛。

但即便如此，折线图有时也无法适用所有场景。 有时我们想使用第三方图表，却受限于Grafana有限的Plugins机制造成无法低成本引入。 好在，Grafana 8官方支持了Echarts图表。

Grafana支持Echarts图表原理不难理解，Grafana通过Query从数据源查询数据，然后将数据作为`function (data, theme, echartsInstance, echarts) {}`的参数传入到一段js代码中。 这个函数需要返回echarts实例，grafana再使用返回的这个实例进行UI渲染。

也就是说，Grafana引入Echarts的原理大致如下:
```javascript
// data 是query查询到的数据
function (data, theme, echartsInstance, echarts) {
    // logic code

    return {
        // echarts instance
    }
}
```

我们如果想使用echarts，那么只需要完成两件事：
1. 对`data`进行数据整合处理
2. 返回`echarts`实例对象

## 数据模型

为了尽可能简单描述问题，我们以[`基础折线图`](https://echarts.apache.org/examples/zh/editor.html?c=area-basic)为例。
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h02ur24cd7j20fg0c2mx9.jpg)

最终效果如下:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h02uxk592dg22ae0hehdt.gif)

为了实现这样的效果，我们首先来看一下数据结构。 再原始数据库中保存的数据如下所示：

｜time|month|value|
|-----|-----|-----|
|2021-10-01|2021-10-01|	13000|
|2021-11-01	|2021-11-01	|14300|
|2021-12-01|2021-12-01	|13500|
|2022-01-01|2022-01-01	|12500|
|2022-02-01|2022-02-01	|15500|
|2022-03-01|2022-03-01	|13800|

`time`是`datetime`类型，用来筛选时间段。 `month`是x轴用来进行分类的字段，`value`则是最终需要显示的值。

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

如此这般操作之后，grafana就会从数据库中查询到数据。 然后grafana需要将数据作为参数放入echarts函数中。

那么在echarts中，data应该是什么样的呢？

我们在函数中，输出一下data：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h02v7y49xpj20uj0u0428.jpg)

可以看到，在series中保存的便是我们在数据库中的数据。 其中name是我们用来准备做X轴分类展示的month，而fields则保存的是每个时间点所对应的具体value。 

通过观察也可以看到，在value的数组中，只有相对应的时间戳中才会出现值。也就是说第一组是`[13000, null, null, null, null]`，第二组是`[null, 14300, null, null, null]`

而我们准备实现的echarts图表需要的数据是这样的：

```json
{
  xAxis: {
    type: 'category',
    boundaryGap: false,
    // data 是X轴数据
    data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
  },
  yAxis: {
    type: 'value'
  },
  series: [
    {
      // data 是Y轴数据
      data: [820, 932, 901, 934, 1290, 1330, 1320],
      type: 'line',
      areaStyle: {}
    }
  ]
}
```

所以我们需要将grafana传入的data数据进行拆分，转化成echarts可以识别的数据。 

## 逻辑处理

grafana的数据是耦合在data对象里面的，所以需要遍历data分别取出X轴和Y轴的数据。代码如下:
```javascript
let id=[];
let value=[];
const series = data.series.map((s) => {
  const sData = s.fields.find((f) => f.type === 'number').values.buffer;
  const sTime = s.fields.find((f) => f.type === 'time').values.buffer;
	
  id.push(s.name);
  sData.map((d, i) => {
      if (d){
        value.push(d)
      }
    })
});
```

`id`保存的是X轴数据， `value`保存的是Y轴数据。 根据echarts的要求，就可以分别传入X轴和Y轴了。 

```javascript

return {
	xAxis: {
		type: 'category',
		boundaryGap: false,
		data: id,
		tooltip: {
			show: true,
		},
		axisPointer: {
			type: 'shadow'
		}
	},
	series: [{
		data: value,
		type: 'line',
		seriesLayoutBy: 'row',
		label: {
			show: true,
			position: 'top'
		},
		areaStyle: {},
		emphasis: {
			focus: 'series'
		},
	}]
}
```
通过这样已经可以实现echarts的折线图了。 但我们还可以继续再往前走一走。 参考echarts的[参数说明](https://echarts.apache.org/examples/zh/editor.html?c=area-basic)
可以添加tooltip、toolbox的feature、legend等。 完整配置如下:

```javascript

let id=[];
let value=[];
const series = data.series.map((s) => {
  const sData = s.fields.find((f) => f.type === 'number').values.buffer;
  const sTime = s.fields.find((f) => f.type === 'time').values.buffer;
	
  id.push(s.name);
  sData.map((d, i) => {
      if (d){
        value.push(d)
      }
    })
});

return {
  darkMode: 'auto',
  color:'#0f8ad1',
  tooltip: {
    trigger: 'axis',
    axisPointer: {
      type: 'cross',
      crossStyle: {
        color: '#999'
      }
    }
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '3%',
    containLabel: true
  },
  toolbox: {
    feature: {
      dataView: { show: true, readOnly: false },
      magicType: { show: true, type: ['line', 'bar'] },
      restore: { show: true },
      saveAsImage: { show: true }
    }
  },
  xAxis: {
    type: 'category',
    boundaryGap: false,
    data:id,
    tooltip:{
    	show:true,
    },
    axisPointer: {
        type: 'shadow'
      }
  },
  yAxis: {
    type: 'value'
  },
  legend: {
    data: id,
  },
  series: [
    {
      data: value,
      type: 'line',
      seriesLayoutBy: 'row',
      label: {
        show: true,
        position: 'top'
      },
      areaStyle: {},
      emphasis: {
        focus: 'series'
      },
    }
  ]
};
```

## 时候总结

Grafana引入echarts的关键在于数据转换。只要将grafana的数据剥离开，分成X轴和Y轴数据，然后按照echarts的规范赋值就可以完成引入了。 
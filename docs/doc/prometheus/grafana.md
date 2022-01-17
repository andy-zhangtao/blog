# Grafana不常用图表制作方法

**WIP**

## Dashboard List

这个图表更像是快捷列表。 在指定的Pannel中可以放置多个其他地方的Dashboard，并且支持一键跳转到对应界面。

效果如图所示：
![](https://tva1.sinaimg.cn/large/008i3skNly1gygo379d5bj31300euaam.jpg)

这个图的重点在于如何选择哪些Dashboard需要展现，依靠的是一些筛选项，如图所示：
![](https://tva1.sinaimg.cn/large/008i3skNly1gygo4cxpnwj30my0ts3zj.jpg)

+ starred 表示标记为[喜欢]的所有dashboard都展示出来
+ recently viewed表示最近浏览过的dashboard都展示出来
+ search 表示通过下面的`query`和`tags`筛选出来的dashboard都展示出来
+ show headings `不是`筛选条件，是用来控制是否输出分类标题。 如果选择no，那么就是下面的效果
![](https://tva1.sinaimg.cn/large/008i3skNly1gygo8odz9oj312y090mxh.jpg)


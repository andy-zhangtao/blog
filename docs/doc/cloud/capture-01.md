# 网络抓包实践 - 基础

在这篇文章中，我们会先看一下如何通过wireshark进行简单的tcp数据包的分析和展示。 

首先，先来抓个包。 抓包最常用的就是tcpdump工具。 一般的操作系统都会自带tcpdump，如果当前机器没有tcpdump，那么需要自行安装。 下面我们假设tcpdump已经安装好了，此时此刻我们就抓一个简单的http包。 

![](https://blog-1253151026.cos.ap-shanghai.myqcloud.com/http-capture.gif)

通过截图可以看到使用tcpdump抓了请求`www.baidu.com`的http请求。 下一步我们就使用`wireshark`来分析这个pcap包。

## wireshark的筛选功能

通过wireshark打开刚才捕获的网络包，此时此刻应该能看到如下这样无数杂乱无章的数据包。
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0phjk8xgtj218a0u04go.jpg)

对于http这样7层应用来说，我们可以首先根据域名来筛选数据记录。例如下面这样，通过`http.host contains "baidu.com"`就可以筛选出所有包含`baidu.com`的7层请求包。 
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0phphsrvuj2238068q4i.jpg)

当然使用`==`也能达到相同的目的：
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0phq2kh5sj21x805qjsm.jpg)

`contains`和`==`的区别在于： `contains`是模糊匹配，而`==`则是精准匹配。

同理也可以根据`methods`来进行筛选，例如筛选所有的GET请求:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0phvt5w8aj21es0nyaju.jpg)

`wireshark`的filter具有完备的逻辑判断，支持添加与或非操作，比如我们需要筛选所有访问`baidu.com`的`GET`请求。那么添加筛选项: `http.request.method == "GET" and http.host contains "baidu.com"`

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0phyde9ofj21qm07awg3.jpg)

同理，也可以根据URI来进行筛选。 

## wireshark的flow graph

当我们分析网络请求时，很少会针对单个数据包进行分析。更多的会结合这个请求包的上下文进行分析。 此时从众多数据包中将属于这个请求的数据包区分出来就显得很重要了。 

wireshark不但能区分出这些数据包，还能非常贴心的通过UI的形式将其展现出来。 我们继续使用刚才的抓包文件来展现这个功能。 

首先使用`http.host`将7层数据包先过滤出来。此时我们看到只有一条记录(一条记录对应一个完整的数据包)。 现在选中这条记录，在下方展开`TCP`层数据，看到有一个`Stream Index`的属性。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmb1qnz0j21i00u0dk9.jpg)

`Stream Index`表示什么呢？  我们知道tcp是字节流传输模式，每次传输的数据一定属于某个流。 但问题在于当前同时传输这么多的流，我们如何知道某个包属于哪个流呢？  `wireshark`为了方便我们跟踪每个流，通过计算 src ip+port 、 dst ip+port 换算出了`Stream Index`。 

> 为什么是src ip+port 、 dst ip+port呢？ 这是因为确定唯一链接依靠的是五元组(源IP、源端口、目标IP、目标端口和协议类型)。 所以依靠src ip+port 、 dst ip+port就可以确定唯一链接了。 

通过这个过程也可以知道，`Stream Index`不属于TCP协议中的内容，而是`wireshark`为了方便我们跟踪而计算出来的虚拟属性。

我们在filter中使用`Stream Index`进行筛选，就可以看到这个链接上面所有的数据包:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmalb7e1j226s0ji7fk.jpg)

现在我们就可以逐个分析数据包了。 但这样就完了么？ 不！ 这样做虽然可以满足需求，但并不方便。我们仍然可以继续深挖`wireshark`的潜力。

选中`Flow graph`可以通过图形化的方式展现客户端和服务端之间的交互:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmedq0hmj215c0u07c1.jpg)


WTFK! 你看到的是不是这个样子的？  虽然好看了，但仍然不清晰不明了呀。 
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmfyyvivj227e0u0qca.jpg)

此时你距离最后的成功还有一步之遥，点击下面的 `Limit to display filter` ，瞬间混乱变清晰，醍醐变灌顶。
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmh9p4l5j21pf0u0n3r.jpg)

对不对，此时就可以很清晰的看到一个典型的TCP完整生命周期， 首先是三次握手:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmko2bdtj22a606umyz.jpg)

然后就开始传输数据:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmloii4xj22as08uwgx.jpg)

最后是四次挥手，结束链接:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmvf6p29j22b009sdjd.jpg)

中间标红的是什么意思呢？  这次演示过程中，很偶然的捕获了一次超时重发的场景。也就是服务端在返回数据(标1处)时，客户端没有及时返回ACK。在服务端等待超时后，服务端再次发送数据(标2处)。 客户端一次性返还了两次ACK(标3处)。
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmwxc12mj21ca0eeq6r.jpg)


## 总结

这篇文章通过一个抓包分析的案例，展示了如何使用wireshark的filter功能，以及如何显示TCP的Flow Graph功能。 后面我们再基于wireshark强大的分析功能进行演示。


# 网络抓包实践 - 如何抓取容器内的网络包

在上一篇文章中，我们使用wirshark分析抓取的网络包，并且通过`Flow Graph`图形化分析网络连接。在这篇文章中，我们在深入一点，看如何抓取容器的网络包。

抓取容器网络的难点在于如何找到容器使用的网卡。 为什么这么说呢？ 这是因为Docker为了让容器和外界进行网络交互，会借助Linux 内核提供的`Veth Pair`特性为每个容器生成一个虚拟网卡。 而正因为这是一个虚拟网卡，所以查找起来会麻烦一些。 

> 何为`Veth Pair`？ `Veth Pair`顾名思义，就是虚拟网卡对的意思。 可以将`Veth Pair`理解成一根网线，一头插在容器中，另外一头插在网络设备上(这里的网络设备可以是网桥，也可以是另外一个容器)。 被`Veth Pair`互通的两个网络设备就可以通过`网线`进行网络交互了。 在Docker网络环境中，如果容器和容器之间通过`Veth Pair`进行网络互通，调整起来会非常麻烦，所以容器之间都通过`网桥`进行互通。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qj17wttmj215d0u076u.jpg)


## 如何获取虚拟网卡之抽丝剥茧法

说完`Veth Pair`后，我们来看一下如何获取容器的虚拟网卡。 在宿主机执行`ip addr`可以看到有很多的虚拟网卡，而且这些虚拟网卡大多(几乎是全部)没有命名规则，想从名称中找到规律，这几乎就是不可能的事情。
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qjb2wb86j20u00ud483.jpg)

从这一大坨虚拟网卡中找出某个容器的网卡有两种方法，我们先来看第一种方法：抽丝剥茧法

假设，我们创建了一个`nginx`容器，如图:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qs9ycv00j20zo06k0ts.jpg)

下一步登录到nginx容器中，查看iflink文件:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qsk9vbstj20pc05egm5.jpg)

> iflink文件是什么？ iflink文件是Linux 2.6添加的一个文件，用来映射网卡和网卡名称之间的关联关系。 在当前操作系统中的路径是`/sys/class/net/<iface>/ifindex` 。 标识指定的iface名称和具体哪个网卡存在对应关系。 如果用在容器环境中，那么就表示容器的某个网卡和宿主机哪个网卡是`配对(Veth Pair)`关系。

> 详细文件描述可以参看 ABI 文档 https://www.kernel.org/doc/Documentation/ABI/testing/sysfs-class-net

下一步就去找`24388`所对应的网卡，如图:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qt2k6afpj21g604k0tl.jpg)

`vethfd59b10@if24387`就是nginx容器中的eth0网卡所对应的宿主机的网卡。 

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qth5n4f2j20po0oymz6.jpg)

此时拿到了宿主机对应的网卡，那么执行tcpdump抓包就易如反掌了。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qtirttcyj21vm0tg4d6.jpg)

## 如何获取虚拟网卡之釜底抽薪法

通过抽丝剥茧法可以拿到对应的虚拟网卡，但过程有些复杂。 我们再来一种快捷的方法，我称之为釜底抽薪法。使用这种方法时只需要获取容器对应的Pid就可以了。 怎么做，往下看。

首先通过`inspect`获取nginx容器的Pid。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qtliphinj20uk06cwf1.jpg)

然后借助内核提供的切换namespace的工具`nsenter`。 这个工具可以切换到指定进程的namespace中。所以使用刚才获取到的Pid，我们就可以低成本切换到对应的网络命名空间。

先来看切换前，我们能查看到的网卡:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qtpfvaruj218t0u0gvu.jpg)

然后执行切换:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qtq70w8qj215y0cqgoq.jpg)

此时此刻，当前用户已经切换到了nginx容器所处于的网络空间。所以只能看到属于它的网络栈，因此现在看到的eth0对应的就是实际在用的虚拟网卡。 剩下的方式就是针对eth0进行抓包了。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qts9tcv6j21nk0u0qgy.jpg)


# 如何为浏览器选择代理

> 以下内容适用于 Chrome 浏览器。 Safari 浏览器尚未开放网络代理权限，因此不适用

浏览器本质也是一个应用程序，只不过这个应用程序是用来渲染 web 页面的。 浏览器默认情况下没有网络代理，所有从浏览器发出的网络请求都会经过当前电脑的网络设置发出。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4p0w4xkklj21780cw41e.jpg)

可以看出从浏览器发出的请求直线到达对方，能否看到期望的内容完全取决于本地电脑的网络设置了。

而使用代理则会增加一种更为灵活的使用方式。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4p9uof87jj217i0ikdkh.jpg)

从图中可知，代理是一种特殊的网络软件。 它和本地电脑网络设置不同的地方在于，可以由我们自己指定将请求转发到哪个服务器。

了解完代理的使用用途之后，我们来看如何为 Chrome 浏览器配置代理。 Chrome 浏览器开放了大部分 API，因此开发者可以基于这些 API 做很多插件，其中有名的网络代理切换插件名叫`switchyomega`（注意 switchyomega 是代理切换插件，而并非代理插件）。

简单来说，`switchyomega`可以为用户根据使用场景来切换不同的网络代理。 比如访问 wwww.baidu.com时，这个请求应该走哪个网络设置。 访问 wwww.google.com 时应该走哪个网络设置。

> switchyomega 安装文件下载地址 http://file.devexp.cn/f/18065158-629271222-98d803?p=6993 (访问密码: 6993)

> 安装方法参考附录 A

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pa2mxhjcj210i0ga0w7.jpg)

知道这些后，你是否会有疑问：switchyomega 是怎么知道哪个网址应该选择哪个代理的呢？ switchyomega 毕竟不是神，也不具备人工智能。的确是这样的，switchyomega 并不具备未卜先知的能力。 它可以完成代理切换依靠的是规则表。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pam5medyj21ca0l87a5.jpg)

其实说白了，switchyomega 本质就是一个简单的`if - else - then`的逻辑判断器。 但某个网址符合某个规则时，`switchyomega`就将这个请求转发到某个代理去。

这么一说，你是不是就有醍醐灌顶，融会贯通的感觉了？

下面我们具体来看一下如何设置`switchyomega`。 首先完成安装(参考附录 A)，而后就会在 Chrome 浏览器出现`switchyomega`的图标:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4papjb8kyj21n6040aab.jpg)

点击这个图标后，会出现`switchyomega`的主界面：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4paqcldaej20bq0hajs5.jpg ":size=150")

- 直接链接
  顾名思义，`switchyomega`忽略所有的网络代理设置，将请求通过本地电脑网络设置去链接，能不能访问到对端就看命吧。

- 系统代理
  也很容易理解。 你的电脑可能有多个代理。 而当选择系统代理模式时，`switchyomega`不会按照规则表匹配合适的代理，而是一股脑的将请求都转给当前的系统代理。

- 情景模式
  这个使用频率最高的一种模式，也是使用`switchyomega`规则表的地方。比如图中所示，假设我们要访问`devexp.cn`(这个是我的个人博客地址)。针对所有访问`devexp.cn`的请求我有三个选择，如图：

  ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pausrnt8j20c40mq0tu.jpg ":size=150")

  - 如果选择`自动切换场景`，那么`switchyomega`会按照规则表的模式进行处理。
  - 如果选择`直接链接`，那么`switchyomega`将所有访问`devexp.cn`的请求转发到电脑默认网络设置
  - 如果选择`H2S`(这是我自己设置的一个代理)，那么`switchyomega`会将所有访问`devexp.cn`的请求转发到我设置的代理。

回到最开始的图, 我们丰富一下这张图:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pb2ndppfj21ja0ksjxz.jpg)

往下来看如何在 switchyomega 中设置代理参数。点击[选项]，就能进入设置页：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pb4l28dgj20c40i2gmf.jpg ":size=150")

选择[新建情景模式]：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pb5g3m1kj20eo0nwwfl.jpg ":size=150")

一般选择第一个（代理服务器）的比较多,

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pb6fgcbjj20u00v2n0b.jpg ":size=150")

在代理服务器设置页中，就可以输入我们准备好的代理参数了：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pb7lyfcuj21dq0ikgnr.jpg ":size=150")

而最后一步就是设置规则，让符合规则的请求转发到我们的代理服务上。

一种方式是自己逐个编辑网址，如下:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pbaifncuj218g0u00y6.jpg ":size=150")

另外一种方式是当访问网址时，通过 switchyomega 图标进行添加:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pbc13zxsj20c40i23zc.jpg ":size=150")

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pbcah06vj20nw0mawfx.jpg ":size=150")

还有一种是直接导入其他人已经整理好的规则，如下:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4pbdwczo2j21rm0ssq89.jpg ":size=150")

可以直接下载我使用的备份文件: OmegaProfile_H2S.pac: http://file.devexp.cn/f/18065158-629273137-9ac90d?p=6993 (访问密码: 6993)

这些都设置好以后，当下次再访问`devexp.cn`时，chrome 就可以按照我们所预期的规则进行转发了。

## 附录 A 如何按照 switchyomega

- 方法一：

打开 Chrome 浏览器，地址栏输入 chrome://extensions/ 将.crx 的文件拖拽到浏览器中间，会出现拖拽以安装的提示，放入即可。

如果浏览器能直接安装即成功，如果不能安装，或者提示只能通过 Chrome 网上应用商店安装该程序，请参照方法二

- 方法二：

将.crx 的文件的扩展名改为.zip，并解压到指定的文件夹（这个文件夹不能删除, 例如解压到了 test 文件夹）
打开 Chrome 浏览器，地址栏输入 chrome://extensions/, 勾择开发者模式，点击'加载已解压的扩展程序'
选择你刚刚.zip`文件解压所在的 test 文件夹，点击确定。扩展程序列表出现你导入的扩展程序即为成功。

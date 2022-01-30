# Electron 入门教程 五
> 这篇文章主要介绍如何实现弹性布局和获取响应

书还接上文，[前面一篇](/doc/front/electron/use-electron-04.md)中，我们成功的实现了再渲染进程中通过IPC与主进程进行了交互。

当实现了这一步以后，我们在渲染进程中就可以进行前端UI开发，不用再担心跨域限制的问题了。 在本篇文章中，我们进入前端开发过程。

## 动态加载

前端开发最大的好处是所见即所得，每次写完代码后，通过Nodejs的热加载功能可以实现实时渲染的效果。 开发效率会大大提高。

但很遗憾，目前无法实现这一点。 为什么？ 因为我们在`main.js`中硬编码了加载方式:

```typescript
mainWindow.loadFile(path.join(__dirname, './build/index.html'))
```

所以每次修改代码后，都需要重新执行build，然后才能看到效果。 前端的伙伴不要火大，因为后端伙伴一直都是这样编写代码的。

这事还有解，我们通过引入一个变量，通过这个变量来标识当前的环境。 如果是开发环境，那么就执行:
```
mainWindow.loadURL("http://localhost:3000/")
```
如果不是开发环境，再执行
```typescript
mainWindow.loadFile(path.join(__dirname, './build/index.html'))
```

通过变量来控制渲染方式。 这个变量我们就放到`package.json`中，在`package.json`中添加`"dev": false`。 当处于开发环境时，改成`true`。 执行构建的时候，改成`false`。

同样在`main.js`中也需要适配一下，添加判断`dev`的逻辑：
```ts
  if (pkg.dev) {
    mainWindow.loadURL("http://localhost:3000/")
  } else {
    mainWindow.loadFile(path.join(__dirname, './build/index.html'))
  }
```

`pkg`就是我们所引入的`package.json`(`const pkg = require('./package.json')`)。

这样我们每次修改完前端代码后，在electron客户端中都可以实时看到效果了。

## 弹性布局

在第三篇中，我们先看到了布局效果。 在这里，我描述一下布局的细节。 首先我们使用的弹性布局。 弹性布局说的直白一些，就是`声明式API`。

在云原生领域中，`声明式API`绝对是个新概念，但在前端领域中，至少2009年的时候就已经提出来了。

在09年之前的前端如果要实现布局，几乎都是清一色的硬编码，每个div高多少，宽多少。 全部都要描述好。 如果遇到不同的尺寸，CSS样式可能都会乱。 所以那会的web页面在开发的时候，会设定一个基准页面。

包括目前，在某些政府网站还能看到下面有推荐的屏幕尺寸和分辨率。 有这一行提示的，不用问，绝对是10年前的历史产物。

到了09年的时候，W3C联盟感觉这事不能再这么玩了，所以提出了弹性盒子的概念，称之为Flex 布局。 这个方案可以简便、完整、响应式地实现各种页面布局。

当容器声明为Flex布局后， 浏览器渲染的时候，会为这个容器生成两个轴。

水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；交叉轴的开始位置叫做cross start，结束位置叫做cross end。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyw2gztoxsj30w20j875p.jpg)

我们不再需要声明每个容器的高和宽了，只需要告诉浏览器每个容器在这两个轴上是怎么排列的。 浏览器会自动计算容器的高和宽。

基于此原理，就冒出了很多的CSS框架，最有名的就是`bootstarp`了。在`bootstarp`中，我们甚至都不需要描述排列方式了，只需要描述最终状态就可以，此时就是`声明式API`了。

`bootstarp`中将屏幕宽化成`12个单位`，1个单位是多少呢？  鬼才知道! 只有真正渲染的时候，才会知道具体1个单位是多少。  就好比，大家谈薪资的时候都说年底14薪，具体多少钱，需要看每个月多少。14薪仅仅表示一个计算单位而已。

比如下面的布局
![](https://tva1.sinaimg.cn/large/008i3skNly1gyw2oiv7euj318u07kdga.jpg)

我只需要描述`A占屏幕宽度的2/3(8个单位)， B占屏幕宽度的1/3就可以(4个单位)`。 bootstarp会自动计算A的宽度是多少，B的宽度是多少。

如果单单是替我们计算，这事还不算神奇。  声明式API神奇的地方在于，每当页面发生变化，需要重新渲染的时候，bootstarp总会计算最新的尺寸。 也就是当屏幕尺寸变化了，容器的尺寸也会随之变化。 但无论如何变化，总会满足我们的需求。

而这种用户只管提需求，不需要关系背后如何实现的编程理念，就称之为`声明式API`。

说到这里，我们再来看翻译页面的布局应该就容易理解了。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyw2utmlwbj30ic0d4ab9.jpg)

将代码和实际布局一一对照，大家应该容易理解弹性布局了吧。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyw2ws64wkj315y0ccdhx.jpg)

## 获取响应

前端的请求绝大部分都是通过各种事件发起的， 比如按钮事件，焦点事件等等。  回到翻译页面，每次输入带翻译内容后，手动点击后面的按钮就可以触发一个对应的事件。代码如下:

```html
<Button onClick={() => _handlerTranslateButton('zh')}>
    <Typography variant="caption" display="block" gutterBottom>
        汉语 {'-->>'} En
    </Typography>
</Button>
<Button onClick={() => _handlerTranslateButton('auto')}>
    <Typography variant="caption" display="block" gutterBottom>
        其他 {'-->>'} 汉
    </Typography>
</Button>
```
我为两个按钮分别绑定了对应的事件，当点击任何一个按钮后，都会触发相对应的函数。

```ts
const _handlerTranslateButton = (direction: string) => {
    let data = {
        src: source,
        srcLanguage: '',
        destLanguage: ''
    }

    switch (direction) {
        case 'zh':
            data.srcLanguage = 'zh'
            data.destLanguage = 'en'
            break
        default:
            data.srcLanguage = 'auto'
            data.destLanguage = 'zh'
            break
    }


    _handlerClean()
    ipcRenderer.send('translate', data)
    window.translate.on('translate-response', (d: string) => _handlerTranslateSuc(d))
    window.translate.on('translate-error', (d: string) => _handlerTranslateErr(d))
}
```

在这个函数中，就会通过IPC调用主进程的能力去请求后端API。 后端API只需要负责将传人的内容翻译成对应的语言，然后返回数据就可以了。

这个后端API建议手动实现，如果实在没有合适的，也可以先使用我提供的一个Serverless服务。

**所有代码请参考 https://github.com/andy-zhangtao/electron-tutorials tutorial-05**
# Electron 入门教程 一
> 这篇文章主要介绍如何构建Electron、React和Typescript技术栈

## 什么是Electron。

Electron是一个实时框架，允许我们通过编写像开发Web前端一样编写本地应用。
> 本地端指的是本地客户端，例如当前正在写这篇文章的VScode，就是本地安装的一个客户端。

> Web端指的是网页端，例如我们经常访问的知乎、Sina等等。这些使用前端工具构建的网站

> 移动端指的是手机App端。

Electron最早是一个名叫Cheng Zhao的github程序员发起的个人项目，后来大家发现貌似这个东西可以一统大前端，所以很多人前赴后继很踊跃的投入了electron的迭代浪潮中。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyitcou6lbj30nw0jgmy7.jpg)


Electron从原理上来说是结合Chromium(网页浏览组件)与Node(底层系统访问)的一个套娃。

Chromium和Node之所以可以放在一起运行，是因为底层都是基于V8引擎运行的。 所以两者理论上存在共同的运行基础。

但两个组件最大不兼容的地方在于两者的运行时不同。 Chromium负责web页面渲染，只加载了浏览器的运行时。 而node则用于访问文件系统、创建服务器和从外部模块加载代码的接口，加载都是本地运行时。两者运行时不同，则说明有些API是缺失的。 因此electron花了很大的力气再解决这种兼容性问题。

这篇教程不在于讲解electron的原理(因为我也不太懂😳)，重点在如何使用现有的技术构建一个本地APP。  话不多说，先看效果。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyitmx8liog31hc0u0u0z.gif)

这个app，起名叫做"我的奇妙工具箱"。 目前这个工具箱里面只有一个在线翻译功能。 希望当你看完这些教程以后，可以自行往里面添加其他的工具。

## React和Electron的融合

### 初始化React

在Electron上面做开发，就和我们做web开发几乎是一样的。 我熟悉的是react技术栈，所以我选择基于react做electron开发。 第一步就是整合React和electron。

本着简单高效的原则，我选择使用`create-react-app` 来创建react应用。 开发react时可以选择js，也可以选择ts。 ts可以定义数据结构，并且写起来有些强类型语言的感觉。 所以创建react工程时，我选择使用typescript模版。因此执行`create-react-app new-react --template typescript`来创建react模版工程。

经过一个XX分钟的等待，目录创建完毕。 我喜欢使用`yarn`来构建工程，所以删除`package-lock.json`后，执行`yarn install`。 (前端说其实太乱了，各种工具层出不穷，就不能统一精简一下嘛 🥱).


安装完毕后，执行`yarn start`验证一下React工程是否正常。看到下面的小动画，就说明React工程初始化成功。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyiu55tq2uj30pm0nejsz.jpg)

### 初始化Electron

安装electron就容易多了， `yarn add electron --dev`先安装一下electron。 `--dev`表示只需要在开发测试阶段依赖，正常构建发布版本时，不需要把这个依赖打包进去。

安装完毕后，我们修改一下`package.json`，添加一个command。

```json
"scripts": {
    "start-electron": "electron ."
}
```

不要迫不及待的执行`start-electron`，还没有完成整合。 执行一定会报错。

前面说了electron有两部分组成，Chromium和Node。 Chromium用于渲染，Node用于控制。 现在只是安装了electron框架，我们还没有补充Chromium和node的运行逻辑，所以空空如也当然会运行失败了。

### Main.js

按照约定俗成，electron中运行node部分的文件称之为`main.js`，这算是electron的入口文件了。 `main.js`中负责创建electron的窗体和设置窗体的一些属性，什么是窗体？ 窗体就是Chromium运行的地方。

`main.js`的代码可以参考[https://github.com/andy-zhangtao/electron-tutorials/tutorial-01/new-react/main.js](https://github.com/andy-zhangtao/electron-tutorials/tutorial-01/new-react/main.js) 。

在`main.js`中会加载两个文件：`index.html`和`preload.js`。

+ index.html

这是node需要Chromium渲染的html文件。 也就是以后我们开发的主战场。 我直接把react工程中的index.html拿来使用了。 当react和electron完成整合，以后我们就像开发react一样修改业务布局，然后交给electron进行渲染。

+ preload.js

node虽然和Chromium都是运行在V8引擎上面的， 但两者还是没有办法直接互通。这样就对node和Chromium的交互造成一些干扰。 为了让node和Chromium可以正常交互，在Chromium启动时，我们会提前加载运行一些js脚本。

这些脚本虽然运行在Chromium中，但却可以访问Node的API。 相当于在Node和Chromium直接创建了一个桥接通道。

简而言之， 什么是preload呢？ `preload`是渲染进程，在页面加载之前加载的js脚本，这个脚本能调用所有Node API、能调用window工具。 在这一节中，我们可以先不用管`preload.js`里面的内容，等到下一节再来考虑。

### Package.json

当我们把`main.js`里面的内容都组装好以后， 还需要告诉electron，`main.js`就是我们的入口文件。  这一步是修改`package.json`文件，添加:`"main": "main.js"`。

这些都做完以后，再执行一次`yarn start-electron`。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyiyncptvsj31a60hodgp.jpg)

可以看到这个截图，那就表示一阶段成果了。 也就是electron配置成功。

### 整合

首先执行`yarn build`构建react工程,稍等片刻后，就会看到`build`目录了。 这个是react构建好的结果目录。 渲染react所有的文件都保存在build目录中。 在下一篇文档中，我们再尝试将react直接打包到程序中。

我们现在先通过加载url的方式整合react，所以需要将`mainWindow.loadFile(path.join(__dirname, 'index.html'))` 改成 `mainWindow.loadURL("http://localhost:3000/")`。

先执行`yarn start`启动react工程， 然后再新开一个终端，再执行`yarn start-electron` 就可以看到下面的截图了。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyizix6t5wj313g0u0jt2.jpg)

## 总结

1. electron是一个node和Chromium的组合。node负责本地端API的执行，Chromium负责页面的渲染。
2. react和electron进行整合的关键在于，让Chromium负责渲染react。 而渲染所需要的文件资源由main.js来指定。
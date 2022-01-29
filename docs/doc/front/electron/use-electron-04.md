# Electron 入门教程 四
> 这篇文章主要介绍如何在electron中通过IPC发起API请求

书再接上文，[前面一篇](/doc/front/electron/use-electron-03.md)中，我们成功的修改了首页内容。使用material-ui进行UI布局。 在这一篇文章中，我们开始实现一个具体的功能逻辑。

## Electrom运行模型

在实现之前，我们需要先搞明白关于electron的运行机制。 electron中分两类进程，`渲染进程`和`主进程`。 `渲染进程`是chromium，按照浏览器的方式运行。 `主进程`是nodejs，按照"后端"的方式运行。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyux6limg7j309e077weg.jpg)


所以虽然electron属于前端，但前端里面也分为`前端的前端`和`前端的后端`两部分。 chromium属于前端的前端，用于页面内容渲染。 而nodejs属于前端的后端，用于获取数据和控制渲染进程。

了解到这里对于此篇文档内容来说就够了。

因为chromium是浏览器引擎，所以受到浏览器规范的限制。 对于这篇文档来说，就是chromium会收到CORS限制。而主进程是Nodejs，所以不受CORS限制。 所以在electron中如果想请求后端API，那么就有两种方式。

方式一，使用chromium发起请求，但需要我们解决跨越问题。
![](https://tva1.sinaimg.cn/large/008i3skNly1gyuxdhjbs2j30i9077wei.jpg)

方式二，在nodejs主进程中发起请求，此时不需要解决跨域问题。
![](https://tva1.sinaimg.cn/large/008i3skNly1gyuxe86ddij30i9077jrf.jpg)

貌似方式二好像更好一些，但方式二还缺了一笔，渲染进程如何和控制进程进行交互呢？ 虽然这两个进程都运行在electron里面，但同族不同宗啊。

chromium和nodejs唯一的结合点是解析js的V8引擎。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyuxhj0jdhj30ds07rmx2.jpg)

两个进程之间无法之间通讯，如果想要实现方式二中的方案，那么就需要渲染进程和主进程之间进行通讯。 而这就是electron的`IPC`解决方案

## IPC方案

![](https://tva1.sinaimg.cn/large/008i3skNly1gyuxmcw1h3j30i90770st.jpg)

IPC方案简而言之，就是在渲染进程和主进程之间创建一组消息队列。 然后渲染进程和主进程按照下面两种场景依次生产和消费：

* 场景一： 渲染进程`委托`主进程调用外部接口

    + 此时主进程监听消息队列，属于消费者。 而渲染进程属于生产者，主动给主进程发送数据。主进程只有一个，所以不能阻塞处理。 接收到以后，就异步处理。什么时候处理完什么时候再回复。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyuxremfiuj30i9077dfw.jpg)

* 场景二： 主进程主动向渲染进程返回数据

    + 此时主进程就转换角色，变成了生产者。向队列中塞入数据，而渲染进程则变成了消费者，从队列中取响应数据进行渲染处理。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyuxs6rhemj30i907774c.jpg)

在具体实现过程中，这个IPC是由electron启动时，在`preload`过程中创建的。 还记得我们在`main.js`中执行的

```js
        webPreferences: {
            preload: path.join(__dirname, 'preload.js')
        }
```
么？ 这个就用于chromium进程启动前的一些初始化。上面所说的IPC消息队列就需要在这个阶段创建。

为什么要在这个阶段创建呢？ 这涉及到chromium使用的是浏览器API，而IPC则属于服务端API。 chromium默认是没有的， 所以在chromium启动前，通过preload先创建一个全局对象，然后作为参数传递给chromium，算是一种变通方案。

原理就说到这里，具体实现我们通过代码再来说。

## 实现

通过上面两个过程也可以知道，渲染进程和主进程各自都需要监听一个队列，我们先来处理main.js主进程的监听事件。

首先从electron中引入ipcMain。

```
const { app, BrowserWindow,ipcMain } = require('electron')

```

然后通过`ipcMain.on`就可以监听指定的队列了。

```js
  ipcMain.on("translate", function (event, arg) {
    axios.post('api url', arg)
      .then(function (response) {
        mainWindow.webContents.send("translate-response", response.data);
      })
      .catch(function (error) {
        console.log(error.message)
        mainWindow.webContents.send("translate-error", '1' + error.response.status);
      })

  });
```
`translate`是队列名称，当`translate`由消息后，会调用后面的处理函数进行处理。我在后面的处理函数中通过`axios`去调用远程API接口。 如果API返回成功，那么就通过`mainWindow.webContents.send`向`translate-response`返回正常数据。 如果API返回失败，那么就通过`mainWindow.webContents.send`向`translate-error`返回错误描述。


而`translate-response`和`translate-error`则是渲染进程所监听的队列。 为什么是两个呢？ 监听一个也可以，我为了将正常情况和错误情况分来，所以监听两个队列。

在渲染进程中通过下面的代码引入IpcRenderer, IpcRenderer是服务端API，浏览器没有。这也正是为什么需要执行preload的原因:

```
declare const window: any;

const { ipcRenderer } = window.electron;

```

然后执行监听：
```
        window.translate.on('translate-response', (d: string) => {
            setLoading(false)
            _handlerTranslateSuc(d)
        })
        window.translate.on('translate-error', (d: string) => {
            setLoading(false)
            _handlerTranslateErr(d)
        })
```

当然在监听这两个队列执行，需要先发送数据。否则就变成资源死锁了。发送数据也是通过send：
```
ipcRenderer.send('translate', data)
```

有小伙伴应该会好奇`window.translate`是哪里定义的？ 这是在`preload`中所定义的；
```
contextBridge.exposeInMainWorld('translate', {
    on: (topic, fn) => {
        // Deliberately strip event as it includes `sender`
        ipcRenderer.on(topic, (event, ...args) => fn(...args));
    },
});
```

我们在preload中定义一个名叫`translate`的对象，然后通过`contextBridge`传递给渲染进程使用。 `contextBridge`可以理解成渲染进程中的`window`。

这样操作完以后，当我们在`translate`页面发起调用请求时，渲染进程就通过IPC委托主进程发起请求了。

总结一下：
1. 渲染进程和主进程通过IPC进行相互通讯
2. 渲染进程使用的是浏览器API，我们需要通过preload的方式注入一些服务端API

**所有代码请参考 https://github.com/andy-zhangtao/electron-tutorials tutorial-04**
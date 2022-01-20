# Electron 入门教程 二
> 这篇文章主要介绍如何打包React和Electron

书接上文，[前面一篇](/doc/front/electron/use-electron-01.md)我们说到如何在electron中集成react。 如果可以通过执行`yarn start-electron`可以看到旋转的小宇宙，就说明`软`集成成功。 `软`集成指的是我们现在通过加载`http://localhost:3000`的方式展现了工程内容。 但这并不是长久之计，当最终打包成可执行程序以后，我们不能要求用户每次打开工具的时候，首先运行react工程。

所以这篇文章中，我们尝试直接将React工程文件打包到electron中。

## electron-builder

打包electron工程，目前使用较多的是`electron package`和`electron builder`。 对比了一下这两个工具，`electron builder`使用的人比较多，同时使用上比`electron package`更为灵活，所以我选择使用`electron builder`构建工具。

先执行`yarn add electron-builder --dev`安装`electron-builder`， 后面的`--dev`表示`electron-builder`仅仅是在开发阶段才需要，运行的时候是不需要的(当我们通过electron-builder构建好工具以后，就不需要electron-builder。 如果没有--dev参数，那么构建运行包的时候，还会把electron-builder打包到最终的包内，包的体积会变大)。

安装好`electron-builder`以后，就可以配置构建参数了。

### 参数配置

`electron-builder`可以单独使用配置文件，也可以直接在`package.json`里面添加构建参数。 为了保持逻辑简单，我就直接在`package.json`中维护构建参数了。 相关的参数可以参考[这里](https://www.electron.build/configuration/configuration#configuration)

针对这次demo，我们首先构建`React`工程，通过`yarn build`构建好相关的工程。 预期构建好的文件都在`build`目录
![](https://tva1.sinaimg.cn/large/008i3skNly1gyjxurw2jhj30po0lsq4v.jpg)

下一步，我们需要将`main.js`中的`mainWindow.loadURL("http://localhost:3000/")`改成读取文件模式`mainWindow.loadFile(path.join(__dirname, './build/index.html'))`, 为什么是`/build`呢？ 这是因为electron执行渲染的时候，默认是从应用根目录开始读取的，而不是`build`目录。所以需要修改一下目录路径。

到底能不能正常展现react界面呢？ 待会儿我们构建的时候看看效果如何。

`electron-builder`构建打包需要知道哪些应该构建，哪些不需要构建。 这些参数就放在`package.json`里面了。  安装官方文档中的描述，我们在`package.json`中创建`build`配置。

```json
  "build": {
    "appId": "zt.first.electron",
    "extends": null
  }
```

`appId`是应用的唯一描述符， `extends`是一个使用`create-react-app(cra)`的坑，官网描述的信息是说当`builder`发现这是一个`cra`工程时，会默认加载`"build/electron.js"`作为程序入口文件。 如果没有这个文件，那么生成asar时就会报错。 所以需要将`extends`置为`null`来禁用这个配置。

配置好以后，执行`./node_modules/.bin/electron-builder build -m`来执行构建。 `-m`表示仅mac系统。


> 如果使用mac构建工程，可能会遇到频繁需要输入口令索取权限的问题。 此时可以通过在`证书链(keychain access) -> 系统`给特定的私有证书授予可信任权限解决。


默认构建文件放在`dist`目录。 在`dist`目录中，会看到`dmg`文件和`mac`目录。 选择直接运行`mac`目录里面的文件可以看到效果，或者执行`dmg`执行安装后再运行也可以看到同样的效果。

什么效果？ 下面的大白屏效果！

![](https://tva1.sinaimg.cn/large/008i3skNly1gyjyjrj0lyj307605g0si.jpg)

打开调试模式后可以看到错误信息

![](https://tva1.sinaimg.cn/large/008i3skNly1gyjyksp2v1j30bd08nmxa.jpg)

这表示虽然我们打包成功了，但打包的内容是错误的。 我们究竟打包了哪些内容呢？

### 打包展开

我们将`mac`目录中的可执行程序里面的app.asar展开.

> 使用`asar`解压`app.asar`内容。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyjyplx3qij310606u0t3.jpg)

我们请求的index.html是在`build/`目录下面。
![](https://tva1.sinaimg.cn/large/008i3skNly1gyjyq7pm7ej31ac06omya.jpg)

而我们真正需要打包的`build`目录不存在，反而打包了一个没有用的`pubilc`目录！原因在于我们使用的默认打包参数，没有针对CRA做调整

### 参数调整

[参考官网](https://www.electron.build/configuration/configuration#configuration) 回到`package.json`的`build`参数中，我们添加`files`参数。

将electron运行所需要的文件都添加进去,同时需要修改`homepage`属性，这个属性用来告诉`react`应该去哪里加载静态文件。 打包构建后的执行文件都放在了`build`目录，所以我们需要告诉`react`去`build`目录加载文件。 修改后的`package.json`配置如下:
```
   "homepage": "./",
  "build": {
    "appId": "zt.first.electron",
    "extends": null,
    "files": [
      "build",
      "main.js",
      "preload.js",
      "package.json",
      "!node_modules/asar/**/*"
    ]
  }
```

重新执行`yarn build`和`./node_modules/.bin/electron-builder build -m `进行打包构建。

构建成功后，执行构建后的程序，就可以看到在`electron`里面欢快旋转的小宇宙了。

![](https://tva1.sinaimg.cn/large/008i3skNly1gyjzjvpg9og31420u0q4b.gif)




# Electron 入门教程 三
> 这篇文章主要介绍如何集成material-ui

书继续接上文，[前面一篇](/doc/front/electron/use-electron-02.md)我们将react工程打包到了electron当中，如果正常的话，你应该也可以看到生成的dmg文件。 这个文件分发出去后，其他的小伙伴就可以直接安装了。

从流程上来说，整个架子搭建完毕了。 剩下就是按照React的开放范式，逐步完善前端逻辑。 现在的前端也存在前后端分离，对，你没看错。 前端也有前后端分离。 这个结论先放在这里，稍后再来论证。

前端的前端就是UI，这个软件的门面。 为了好看一些，我选择使用material-ui(aka mui)。

## 引入Material-ui

material-ui 是一套适配了React的前端UI框架，安装后开箱即用。 我们通过`yarn add @fontsource/roboto @material-ui/core @mui/material @emotion/react @emotion/styled @mui/icons-material`直接安装。

安装完毕后，按照最终效果图所示，我选择[抽屉模式](https://mui.com/zh/components/drawers/#mini-variant-drawer)作为工具的入口页面。 为了尽快看到效果，我们选择直接使用官网中的实例代码。

新建`src/componts/layout`目录，创建`index.tsx`作为入口UI。

将官网中的代码复制到`index.tsx`当中(记着选择ts版本的代码)。

### 修改React渲染入口

我们将`index.tsx`处理好以后，还需要告诉`react`首先从`MiniDrawer(刚才拷贝过来的模块名称)`开始渲染。 哪里修改入口呢？ 在`src/index.tsx`文件中，找到
```
ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```

`<App />`是React运行时首先进行渲染的地方。  我们将`<App />`修改成刚才的`MiniDrawer`：
```
ReactDOM.render(
  <React.StrictMode>
    <MiniDrawer />
  </React.StrictMode>,
  document.getElementById('root')
);
```

执行`yarn start`后，就可以看到入口页面已经不是小宇宙了
![](https://tva1.sinaimg.cn/large/008i3skNly1gym9dq4mszj30ls0r6wh6.jpg)

模样出来了，但你点击左边的菜单，会发现什么都没有变化。 很正常，这只是一个Drawer的demo而已。 我们下一步就来改造一下这个demo。

### 改造首页

为了不啰嗦，我先描述一下改造思路：
1. 移除多余的图标，只保留一个图标
2. 点击图标，右侧可以展现相对应的页面

#### 删除图标

我们通过查看`src/componts/layout/index.tsx`的源码可以发现，图标的逻辑在:

```
                <List>
                    {['Inbox', 'Starred', 'Send email', 'Drafts'].map((text, index) => (
                        <ListItem button key={text}>
                            <ListItemIcon>
                                {index % 2 === 0 ? <InboxIcon /> : <MailIcon />}
                            </ListItemIcon>
                            <ListItemText primary={text} />
                        </ListItem>
                    ))}
                </List>
```

所以直接大刀阔斧的删除掉没用的菜单，只保留一个就可以了。
```
                <List>
                    <ListItem button key={"index"}>
                        <ListItemIcon>
                            <InboxIcon />
                        </ListItemIcon>
                        <ListItemText primary={"翻译"} />
                    </ListItem>
                </List>
```

这样再看浏览器中的页面，就只剩下一个图标了。
![](https://tva1.sinaimg.cn/large/008i3skNly1gym9l79i2lj30g20eqwej.jpg)

#### 添加新页面

虽然我们是单页面应用，但页面的跳转也需要`router`来实现。 现在先不引入`router`进行跳转，先使用最简单的`障眼法`来实现。

我们创建翻译的页面`src/componts/translate/translate.tsx`。在里面放入两个文本框，一个输入，一个输出。然后在对error信息框、操作按钮和翻译输出进行布局调整。 布局图如下：

![](https://tva1.sinaimg.cn/large/008i3skNly1gym9z6x5esj318o0hk0tq.jpg)

现在的前端绝大部分都使用栅格布局方案，所以我也选择使用`Grid`进行布局。 详细的布局，大家可以直接查看最后的源码。

完成布局后，就开始添加逻辑处理了

#### 页面跳转

为什么说是使用`障眼法`呢？ 这是因为在首页布局中，我们通过对`hidden`属性进行`true/false`转化来显示和隐藏。 等到后面再说如何通过`router`进行页面跳转。

思路已定，那我们就首先在`index.tsx`合适的位置引入`TranslateGrid`。

```
                    <Grid item xs={12}>
                        <div hidden={!openTranslate}>
                            <TranslateGrid />
                        </div>
                    </Grid>
```

然后初始化一个`openTranslate`属性，通过修改它的值来控制显示还是隐藏。 既然是障眼法，那么原先的内容也需要进行逆向操作。

```
<div hidden={!openIndex}>
xxxx
</div>
```
`openTranslate`和`openIndex`互为反操作。

什么时候触发呢？ 就是我们在点击图标按钮的时候进行赋值操作。 所以为图标绑定`onClick`事件，

```
                <List >
                    <ListItem button key="index" onClick={() => handleIndexShow(IndexType.Translate)} >
                        <ListItemIcon>
                            <IconButton >
                                <GTranslateOutlinedIcon sx={{ color: blue[700] }} />
                            </IconButton>
                        </ListItemIcon>
                        <ListItemText primary="翻译" />
                    </ListItem>
                </List>
```

```
    const handleIndexShow = (it: Number) => {
        switch (it) {
            case IndexType.Translate:
                setOpenTranslate(true)
                setOpenIndex(!openIndex)
                break
        }
    }
```

回到浏览器，看一下效果。点击按钮后，应该会看到右侧主页面可以在翻译页面和主页面之间切换了。

![](https://tva1.sinaimg.cn/large/008i3skNly1gymafhc1eoj317q0fagmo.jpg)

## 总结

1. material-ui 是一套基于React的开箱即用的UI框架，非常好用
2. 当页面多时，可以选择使用`router`进行跳转。 页面少时，使用`hidden`控制成本低


**所有代码请参考 https://github.com/andy-zhangtao/electron-tutorials tutorial-03**
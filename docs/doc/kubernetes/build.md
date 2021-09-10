# 如何构建Kubernetes源码

`kubernetes`仓库地址是`https://github.com/kubernetes/kubernetes`。在仓库文档中指明了两种构建方案：

1. 基于Golang环境进行构建。
2. 基于docker环境进行构建。

## 基于Golang环境进行构建

按照文档中的描述，将仓库代码`clone`到 `$GOPATH/src/k8s.io` 目录中。然后执行`make`即可。

这个方案需要:

+ golang环境(很容易满足，安装最新版即可)
+ bash >= 4.2(Linux基本都满足，但Mac OS默认的bash是3.X，需要将所有的env bash替换成env <高版本的bash>)

## 基于docker环境进行构建

通过执行`make quick-release`可以执行这个构建方案，通过`quick-release`也可以看出这是一个非常快速的构建方案，但如果你深处祖国大陆，那么就放弃吧。

执行`quick-release`后会尝试先构建后续操作的build image。 但构建这个镜像需要访问`gcr.io`仓库。所以本地如果不能访问这个地址的话，就放弃吧。

这是官方提供的两个方案，第一个方案比第二个方案稍好一些。

## 基于docker构建环境进行构建

我基于第一个方案做了一些封装，生成了kubernetes的构建镜像：`registry.cn-hongkong.aliyuncs.com/vikings/kubernetes:build-env`

这个环境包含`golang 1.17`和构建所需的依赖库。 使用时只需要执行下面的命令即可启动构建:

```shell
docker run -it --name build -v <kubernetes source code path>:/go/src/k8s.io --entrypoint bash --workdir /go/src/k8s.io registry.cn-hongkong.aliyuncs.com/vikings/kubernetes:build-env make
```

这样就可以完成所需构建，唯一不足在于构建环境是Linux，因此不会执行跨平台构建。
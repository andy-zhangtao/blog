# Mock Server实现方法总结
> WIP版本!

以下是经过验证可以使用的`Mock Server`方法，考虑到现实中存在不同的使用场景，因此只提供重要步骤，如果与使用场景不匹配则需要按需裁剪。

## 7层请求Mock Server
> 支持Http/Https(1.1/2.0) 的7层Mock请求

7层流量分为两类：

+ 字节传输
+ 二进制传输


如果是字节传输，只需要维护好响应数据即可。如果是二进制传输，则稍微复杂一些(相对于字节传输而言)，需要根据实际场景来定义如何`组织`二进制数据(例如传输文件/图片/视频等等)。

下面优先考虑如何实现字节传输场景Mock。

### PostMan

使用`PostMan`作为Mock时，有两种方式: 1. 从零搭建和2. 基于已有请求创建。

+ 从零创建一个Mock Server

在Postman中选择`Mock Server`，选择NEW 然后选择`Mock Server`。
在页面中选择`Method`和`Request URL`，如果有`Response Body`，则一并输入。点击Next，在下一页中输入`Mock Server`的名称和环境标识。

**注意：如果选择了`Make mock server private`。 那么必须使用`x-api-key`才能访问到Mock。**

![](https://yx-prod-resources-shared-1254112465.cos.ap-beijing.myqcloud.com/1/7cf6163fb20d868c09eab83f792c7e59-58861)


![](https://yx-prod-resources-shared-1254112465.cos.ap-beijing.myqcloud.com/1/e44d2c9dc931a3a3630569d384b293bf-96461)

![](https://yx-prod-resources-shared-1254112465.cos.ap-beijing.myqcloud.com/1/4963d8dc893f56db7e0fb213fb73fe86-119993)

+ 基于当前请求创建Mock Server

在Postman中选择`Mock Server`，选择NEW 然后选择`Mock Server`。
选择`Select an existing collection` 。 然后从当前集合中选择需要mock的API。  注意如果使用这个功能，需要提前创建example。
而创建`example`则需要在正常调用API后，将`response`保存为example即可。

![](https://yx-prod-resources-shared-1254112465.cos.ap-beijing.myqcloud.com/1/5308cac62ea7bfcb02b14d15eca93dbf-166310)

![](https://yx-prod-resources-shared-1254112465.cos.ap-beijing.myqcloud.com/1/badb7b7ea945c0d302037a47404d739f-131109)

## 4层Mock Server

待续 ...
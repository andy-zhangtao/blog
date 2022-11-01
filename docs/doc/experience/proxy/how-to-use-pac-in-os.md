# 如何在 Mac 中使用 pac 文件

开始之前，我们先说一下为什么会出现 Pac 文件。 Pac 文件全称叫做"Proxy AutoConfig"，顾名思义就是代理自动切换文件。通过这个文件，mac 中的一些工具就可以选择通过何种方式访问外部网络。

因此 pac 文件是为了方便切换代理而存在的。如果打开`网络偏好设置`，可以看到还有其他代理设置方式:
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h51elxcl7bj20fe0cmt9l.jpg ":size=350")

- 自动发现代理

  > mac 自动在内网中通过 WPAD 协议搜索有没有代理服务器。而代理服务器做了点啥呢？ 没做啥别的事情，代理服务器会自动向客户端下发`pac`文件。因此代理服务器就是一个集中管理`pac`文件的服务器而已。

- 自动代理配置

  > 如果内网中没有代理服务器，那么就可以自己编写一个`pac`文件保存起来。通过`自动代理配置`功能加载这个文件。当然也可以将`pac`文件放到网络中，通过`自动代理配置`功能自动加载。 比如`https://gist.githubusercontent.com/andy-zhangtao/f314e15bd3d45bc4353a3aee3fbb2666/raw/bfdcad89d018ac893567d67327b0b33ef3f7a219/omega_auto.pac`就是我写的`pac`文件。

- 网页代理、安全网页代理、FTP、Socks 代理
  > 这几个代理大同小异，输入指定的代理服务器地址即可。

mac 通过这些设置就可以实现绕过防火墙访问外部网络的作用，这里的防火墙泛指所有防火墙，不仅仅是 GFW。

上面这几种代理方式中，使用最方便的就是`pac`文件了。 我们来看一下如何编写`pac`文件。

首先`pac`文件是一个`js`代码片段，我们写`pac`文件其实就是再写`js`函数。 这个`js`函数原型是:

```js
function FindProxyForURL(url, host)
```

- url
  要访问的 URL. 注意不需要保留协议类型。比如需要为`https://www.google.com`编写一条规则，那么可以写成:

  ```js
  if (dnsDomainIs(host, ".google.com")) {
    return "PROXY proxy.intranet1.com:8080";
  }
  ```

  这个配置的意思是，当有访问外部请求时，系统会截取请求的`host`部分。然后将`host`与`.google.com`进行匹配。如果匹配成功，那么返回`PROXY proxy.intranet1.com:8080`。否则不做任何事情。

  是否匹配成功的规则如下:

```js
dnsDomainIs("www.google.com", ".google.com"); // true
dnsDomainIs("www", ".google.com"); // false
```

而`PROXY proxy.intranet1.com:8080`是什么意思，留在后面再说

- host
  从 URL 中提取得到的主机名。不包括协议类型和端口号。也就是说，https://www.google.com 、 http://www.google.com和https://www.google.com:1234 截取的 host 都是www.google.com。

了解到这两个参数含义后，剩下的事情就是我们如何编写自己的逻辑了。浏览器内置了一些函数可以简化代码量:

- 基于主机名的判断函数

```js
isPlainHostName();
dnsDomainIs();
localHostOrDomainIs();
isResolvable();
isInNet();
```

- 和代理相关的功能函数

```js
dnsResolve();
convert_addr();
myIpAddress();
dnsDomainLevels();
```

- 基于 URL 或主机名的判断函数

```js
shExpMatch();
```

- 基于时间的判断函数

```js
weekdayRange();
dateRange();
timeRange();
```

- 日志记录功能函数

```js
alert();
```

当我们编写完逻辑以后，就需要告诉系统如何分发流量了。 一般有 7 种分法规则:

- DIRECT

  > 直连，不经过任何代理

  例如

  ```js
  // 所有baidu.com域名的请求直连
  if (dnsDomainIs(host, ".baidu.com")) {
    return "DIRECT";
  }
  ```

- PROXY host:port

  > HTTP 代理

  例如上面的例子

  ```js
  // 所有google.com域名的请求全部转发到proxy.intranet1.com:8080
  if (dnsDomainIs(host, ".google.com")) {
    return "PROXY proxy.intranet1.com:8080";
  }
  ```

- SOCKS host:port

  > Socks 代理

  使用方式不赘述

- HTTP host:port 、 HTTPS host:port

  > HTTP 代理 和 HTTPS 代理

- SOCKS4 host:port 、 SOCKS5 host:port
  > 一般会使用 Socks 代理的方式替代

在设置代理地址的时候，左边的代理地址是主代理。如果主代理不通，则会切换到备用代理。 同时经过 N 分钟之后(默认 30 分钟)会尝试连接主代理。

如果所有代理都不通时，并且没有设置默认的`DIRECT`选项时，系统会暂时会忽略代理。

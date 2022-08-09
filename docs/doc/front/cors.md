# Cors 跨域基本原理

## 为什么有了跨域这个东西

世上本没有路，走的人多了也就有了路。 跨域这算是这么一回事。 在 Web 的世界上本没有跨域这个东西，但架不住鸡鸣狗盗之徒越来越多，所以后来就有了跨域。

何出此言？

互联网刚出现的时候，是默认全开放的。也就是你说，你写的一个网站默认是允许所有人访问的。这一点非常符合互联网"互联互通"的精神本质。但后来人们却发现，事情好像有点变质变味。

比如说，Bob 写了一个网站让 Alice 访问，当 Alice 访问这个网站的时候，谁知 Eve 中途拦截了请求，在里面植入了 Eve 自己写的 Js 脚本。 这个脚本会源源不断的请求 Bob 网站的数据，并且通过浏览器发给 Eve。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h4zp71shi1j20o60gmq4r.jpg ":size=400")

Eve 的脚本此时此刻充当了小偷的角色，盗取数据并返回给 Eve。

既然第三方的 Js 有这样的危险，那么我们对这些 Js 脚本禁止执行怎么样？ 必须不能！ 你想想互联网中除了有坏人，也有好人呀。如果我们禁止所有 Js 脚本运行的话，那我们就无法使用网银了，电商平台了，甚至连绚丽多彩的 Web UI 都无法看到了。

既然不能禁止，也不能放任自流，那又该何去何从？所以引出了`跨域`，跨域简而言之，就是浏览器在发请求的时候，先询问一下服务器，你让我能做点啥？

## 为什么是浏览器

这是一个好主意，为什么跨域只出现在浏览器呢？ 这也算是当前一个无解的事情，因为 Http 请求天生是无状态的，服务器无法区分这个请求和上个请求是否是同一个客户端发起。当无法判断是否为同一个客户端时，服务端也就无法判断这个请求是应该通过还是应该禁止。

所以跨域这件事只能下沉到浏览器来执行了。让浏览器承担第一道防火墙的责任。 毕竟能做浏览器的都是大厂，这点基本职业操守，浏览器厂商还是有的。

那话说回来，服务端就一点也不做么？ 并不是的， 服务端限制来自服务端请求有另外方案，比如 Token、API Check 等方式。

因为 RestFul API 之间无状态，所以只能通过校验请求合法性，来变相控制对方是否有权限获取/变更数据。

## 浏览器如何实现跨域

浏览器通过`协议`+`域名`+`端口`这三者来判断是否属于跨域。 整理如下:

| 浏览器地址         | 准备访问地址            | 是否跨域 | 原因             |
| ------------------ | ----------------------- | -------- | ---------------- |
| http://www.bob.com | http://www.aclice.com   | 跨域     | 域名不一致       |
| http://www.bob.com | https://www.aclice.com  | 跨域     | 协议、域名不一致 |
| http://www.bob.com | https://www.bob.com     | 跨域     | 协议不一致       |
| http://www.bob.com | http://www.bob.com:8080 | 跨域     | 端口不一致       |
| http://www.bob.com | http://www.bob.com      | 不跨域   |                  |

浏览器每次在发起请求的时候，都会经过这个逻辑判断。 如果判断需要跨域，那么就会先向服务端发起`OPTIONS`请求。`OPTIONS`请求是向服务器询问，你允许哪些外部域名访问你？如果服务端允许当前浏览器访问，浏览器再继续访问。如果不允许，那就此作罢。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h508xugpvij20nk0bmt9w.jpg ":size=400")

如图所示， 浏览器通过 OPTIONS 请求询问服务端。 服务端回复只接受来自https://www.bob.com的请求。

如果当前浏览器是https://www.bob.com那么愉快的发起正式请求，如果浏览器不是https://www.bob.com。那么就此别过以后再见。

是不是感觉跨域好像很简单，也没有多么复杂。 事实也是这样的，跨域本身不复杂，复杂的里面的参数配置。

### 常见的参数配置

前面说过了，跨域的控制权限在服务端，不在客户端。 因此服务端如果需要支持跨域，那么需要将以下参数通过 OPTIONS 请求返回给客户端。

- Access-Control-Allow-Origin: <origin> | \*

  > 用来标识谁可以访问我。 这个 header 只支持返回一个值，要么是特定外部 URI(https://www.devexp.cn)，要么是"*"。 不接受其他值
  > 如果服务端指定了具体的域名而非“\*”，那么响应首部中的 Vary 字段的值必须包含 Origin。这将告诉客户端：服务器对不同的源站返回不同的内容。

- Access-Control-Expose-Headers: 允许浏览器可以访问哪些自定义 Header

  > 默认情况下，浏览器只能读取最基本的响应头，Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。
  > 如果服务端想让浏览器获取其他 Header，就需要通过这个 Header 来指定。 这个 Header 支持逗号分隔的多值

- Access-Control-Max-Age: 预检请求的结果在多少秒内有效

  > 如果每次请求都需要预检，那么浏览器工作压力大，而且也浪费带宽。所以服务端可以通过这个 Header 告诉浏览器，多长时间内免检。

- Access-Control-Allow-Credentials: 是否允许客户端携带验证信息

  > 默认客户端只能携带特定的 header 发给服务端，如果服务端允许验证身份信息的 header(Cookie)。那么通过将此 Header 置为 true 来解决此问题。
  > ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h509ihlzcgj20sf0dmwf4.jpg ":size=400")

  当 Access-Control-Allow-Credentials: true 时，其他控制 Header 就不能是"\*"了。比如:

  - 服务器不能将 Access-Control-Allow-Origin 的值设为通配符“\_”，而应将其设置为特定的域，如：Access-Control-Allow-Origin: https://example.com。
  - 服务器不能将 Access-Control-Allow-Headers 的值设为通配符“\_”，而应将其设置为首部名称的列表，如：Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
  - 服务器不能将 Access-Control-Allow-Methods 的值设为通配符“\*”，而应将其设置为特定请求方法名称的列表，如：Access-Control-Allow-Methods: POST, GET

- Access-Control-Allow-Methods: 表示实际请求可以是哪些请求

  > 这个 Header 接受逗号分隔的多值

- Access-Control-Allow-Headers: 表示实际请求可以携带的 Header
  > 这个 Header 接受逗号分隔的多值

### "豁免"的情况

不知为何，浏览器会豁免一些场景的跨域判断。 被豁免的场景称之为"简单请求"，不被豁免的则称为"复杂请求"。

符合下面条件的就是简单请求:

1. 是 GET、HEAD、POST 之一
2. 只有 Accept、Accept-Language、Content-Language、Content-Type 这几个 Header
3. Content-Type 的值仅限于下列三者之一：text/plain、multipart/form-data、application/x-www-form-urlencoded

除此之外的请求都需要经过预检判断。

## 常见问题

1. 配置了跨域，浏览器仍然显示跨域错误
   首先判断是否满足跨域判断，然后检查服务端返回的 Access-Control-Allow-Origin、Access-Control-Allow-Methods 和 Access-Control-Allow-Headers 是实际发起的值是否匹配。
2. OPTIONS 请求返回 204 还是 200？
   这个问题仁者见仁，智者见智。 但目前绝大多数的配置指南都建议返回 204. 因为从语义上说 204 表示请求成功，但没有任何返回。所以比较适合 OPTIONS 请求场景。

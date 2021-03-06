# TCP/IP(链接)
## 流程

应用层向TCP层发送未知字节数量的数据，TCP层将接收到的数据切分成不同大小(根据MTU计算而来)的数据报。然后TCP层将切分后的数据报发送给IP层。IP层再添加包头和包尾发送出去。

TCP为了跟踪每个包是否都被正确的接受，会为每个包添加一个SEQ。对方会为每个包返回一个对应的响应信息作为接受确认(ACK)。当在指定时间内(RTT)没有收到ACK，则认为发送失败，并由此触发超时重传。

![](https://yx-prod-resources-shared-1254112465.cos.ap-beijing.myqcloud.com/1/9ccbf15f8fec01af4bab24d9fae211fe-30675)

## SEQ和ACK的规则

SEQ和ACK都是随机值，但为了尽可能在一个发送过程中携带更多的“信息”，SEQ和ACK会遵守一定的规则来产生。

1. 在同一个链接的同一侧，seq和ack不能重复。  tcp使用四元组来区分链接，在同一个链接中，发送端不能发出重复的seq。 但不同的发送端可以发出相同的seq。

2. 发送端第一次使用SEQ=0作为初始化。

3. 接收端用ACK来表示当前已经确认的数据长度。假设接收端回复的包中ACK=38，则表示发送端<=37字节之前的数据都完整接受了。

4. 在PUSH状态中，发送端从ACK处发数据(ACK之前的已经完成接受)，并将此ACK作为SEQ发送出去。

5. 在其他状态中，发送端将ACK+1作为SEQ(ACK不变)。 接收端将SEQ+1作为ACK(SEQ不变)。此时的SEQ和ACK仅仅作为数据包标识。

![](https://yx-prod-resources-shared-1254112465.cos.ap-beijing.myqcloud.com/1/82e97b686202f8971294ae08bd9f7f0d-188323)

## 创建链接的细节

### 两个队列

服务端(接收端)当执行Listen函数后，会创建两个队列(应该是在内核中)，syn队列和accept队列。

+ syn队列属于"二次握手队列"，保存的是客户端发来syn包的链接，这样的链接只有服务端做好了tcp初始化，客户端不一定做好了tcp初始化。因此不能用于接受数据。

+ accept队列属于"三次握手队列”，保存客户端发来syn+ack的链接。这样的链接表示客户端和服务端都做了tcp初始化，随时可以接受和发送数据。

### 过程

+ 客户端向服务端发送syn包，其中目标IP，目标端口和源IP此时都已经确定，源端口随机挑选。syn包的seq初始化为0。
+ 服务端接受到syn包后，回复ack包。 根据规则5，ack包中的seq = 0，ack=1(syn包中的seq+1作为ack返回，并因为是服务端第一个包所以seq=0)。同时将源IP、源端口、目标IP和目标端口存入syn队列。(源IP、源端口、目标IP和目标端口可以mapping成strem index，作为链接唯一标识)
+ 客户端接受到ack包，根据规则5，回复ack包给服务端，其中seq=1,ack=1(客户端将ack+1作为seq，同时保持ack不变)。
+ 服务端接受到ack后，将syn队列中向对应的tcp stream index取出放入accept队列中。与此同时，在内核中创建socket文件，开始与客户端交互数据。

### 关注点

1. 队列长度设置。 Linux使用backlog来设置队列长度， backlog分为系统参数和Listen参数，取min{系统，Listen}最小值。 通过 ss -lntp 来查看。Recv-Q表示Syn队列当前保持的链接。Send-Q表示Syn队列最大值。backlog经验值是服务QPS 1.5倍。

2. tcp_synack_retries。 当服务端没有收到ACK时，重试次数。 在内网环境中，这个值建议设置为0。网络环境不好可以适当增大。

3. tcp_syn_retries。当客户端没有收到ACK时的重试次数。内网环境中，设置为0。

4. tcp_max_syn_backlog。系统层面backlog大小。
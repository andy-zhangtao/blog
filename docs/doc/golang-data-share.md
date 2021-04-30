# Golang 内存变量如何保证读写一致性


这个问题是由读[The Go Memory Model](https://golang.org/ref/mem)这篇文章而有感而发。

数据一致性不止在Golang中存在，在其他语言和领域也存在。不论问题出现在哪里，解决方案几乎都相同。

先来看看最先出现数据一致性问题的CPU。

## CPU如何保持cache一致性

在CPU刚出世的时候，其实是不存在这个问题的。 因为那个时候，只有单核CPU，并且是单进程运行。 后来伴随着集成电路技术的发展，硬件爆发式发展。 CPU为了能更快的运行，诞生了`多级流水线`,`乱序执行`,`分支预测`等技术。

同时单核CPU也变成了多核CPU，而为了解决CPU和内存之间的速度差异，又添加了`L1`，`L2`和`L3`三级cache。 其中`L1`和`L2`在CPU内部，而`L3`则是所有CPU共享。

![](https://app.yinxiang.com/FileSharing.action?hash=1/d5f88ed6ae790a9d958f721cdcea5e0f-48970)


`L1`，`L2`和`L3`的出现，使得CPU不需要读取内存这样低速设备(相对于CPU而言，内存IO速度挺慢)。和内存交互的事情，由`LX`来完成，CPU每次只需要从`LX`中读取数据就可以了。


无论从上面出现的每项技术来看，好像都没什么问题， 但这些技术组合到一起，就要命了。 例如同一个内存变量`a`。如果大家谁也不修改它的值，那么在`L1`,`L2`和`L3`中会是相同的。 但如果此时此刻，有一个CPU修改了这个值，那么在其他cache中的`a`势必就变成了脏数据， 再读出来就错了。

CPU界为了解决三级cache数据不一致的问题，推出了`MESI`(及其子协议)。 `MESI`大致描述了数据的四种状态: `修改`,`独占`，`共享`和`无效`。

当一个CPU想要修改某个数据时，会通过数据总线发送`独占`申请。当申请成功以后，就进入`修改`状态[MESI的讲解](https://www.jianshu.com/p/0e036fa7af2a)

![](https://app.yinxiang.com/FileSharing.action?hash=1/6575f0878c5803677c0b4c7980ac0992-13830)


| 初始状态 | 操作 |响应  |
| --- | --- | --- |
| Invalid(I) |PrRd  |1. 给总线发BusRd信号 2. 其他处理器看到BusRd，检查自己是否有有效的数据副本，通知发出请求的缓存 3. 如果其他缓存有有效的副本，状态转换为(S)Shared 4. 如果其他缓存都没有有效的副本，状态转换为(E)Exclusive 5.如果其他缓存有有效的副本, 其中一个缓存发出数据；否则从主存获得数据 |
|  | PrWr | 1. 给总线发BusRdX信号2. 状态转换为(M)Modified 3. 如果其他缓存有有效的副本, 其中一个缓存发出数据；否则从主存获得数据 4. 如果其他缓存有有效的副本, 见到BusRdX信号后无效其副本 5. 向缓存块中写入修改后的值  |
| Exclusive(E) | PrRd  | 1. 无总线事务生成 2. 状态保持不变 3.读操作为缓存命中 |
|  |PrWr  | 1. 无总线事务生成 2. 状态转换为(M)Modified 3.向缓存块中写入修改后的值 |
| Shared(S)  | PrRd | 1. 无总线事务生成 2. 状态保持不变 3. 读操作为缓存命中 |
|  | PrWr | 1. 发出总线事务BusUpgr信号 2. 状态转换为(M)Modified  3. 其他缓存看到BusUpgr总线信号，标记其副本为(I)Invalid. |
|Modified(M)   | PrRd | 1. 无总线事务生成 2. 状态保持不变 3. 读操作为缓存命中 |
| |PrWr |  1. 无总线事务生成 2. 状态保持不变 3. 写操作为缓存命中   |

通过MESI协议可知，CPU Cache保持数据一致性所依靠的也是`消息-通知`机制，只不过此时的机制是通过数据总线发布，而不是通过常规的网络协议发布，所以效率非常高。

通过CPU Cache数据一致性可以看出，天下武功唯快不破。只要消息传递的足够快，就足矣保证数据的一致性。 在很大程度上，分布式数据一致性都依靠的是`消息-通知`机制。

现在回到Golang语言上面，Golang又是如何保证内存数据一致性的呢？

## Golang的解决方案
众所都知，并发是golang的一大特性，协程创建成本极低，因此golang可以轻松同时创建上万个协程。这些协程可以共享所有的全局变量。golang可以支持这个特性，依靠的是两种方案:

+ chan通讯
+ lock机制

先说`chan通讯`。

### Chan方案

chan从本质上面来说是一个`环形数组+一个写锁+一个读锁`，遵从CSP(Communicating Sequential Processes)原则。

![](https://app.yinxiang.com/FileSharing.action?hash=1/ba31a8301bafcfb82a136eae321679ff-53870)

Chan分为带缓冲和不带缓冲两类，区别在于发送时是否阻塞。带缓冲的chan当缓冲满了以后再向其写入时会阻塞，而不带缓冲的chan如果没有读取操作，则会一直阻塞。

简单而言，带缓冲的chan会通过一个环形数组来保存数据，如果头尾相遇，则表示数组满了。而不带缓冲的chan就没有这个数组了。从实际使用情况来看，不带缓冲的chan容易出现死锁情况，需要先创建读Goroutine再创建写Goroutine。而带缓冲的chan就没有这种情况了。

发送消息时:

1. channel已经关闭，那就不能发。panic掉。
2. 看一下有没有阻塞在读操作上的goroutine，有的话，赶紧把数据复制给它。把它安排在下一次调度切换上.
3. 看没有被阻塞的goroutine。如果带buffer，buffer还有空位，就放在buffer里。否则就阻塞挂起当前发送消息的goroutine。

读取消息时：

1. channel已经关闭，也可以读，只是读出来的数据为空。
2. 看一下有没有阻塞的写操作的goroutine，有的话唤醒它。读取它发送的数据(A)。
3. 判断数据(A)放哪。无buffer的话，就直接把写数据(A)给消费者。带buffer的话，就先看buffer里是否有数据(B)，有就把数据(B)给读取者，再把数据(A)放到原来数据(B)空出来的位置上。

而整个过程都需要加锁(Lock)，Lock的实现放在下面描述。

所以通过chan传递数据时，都是按照
```
生产者 --> Buffer --> 消费者
```

这样的顺序流转的。而且都是采用拷贝的方式(typedmemmove)进行，所以使用channel时，避免使用过大的数据。而且要改变从channel里读取出来的值时，channel的类型要使用指针类型。

Go放弃通过共享内存的方式而采用共享通信的方案来实现数据一致性，经过实际证明，此方案效率很高。

### Lock方案


刚才说到使用chan在多个Goroutine之间同步数据时，需要通过Lock来实现写锁。当Goroutine之间需要共享一块大数据时，就不便采用chan方案了(受制于值拷贝，因此大数据拷贝时会拉低性能)。 可以通过Lock来对这块数据加锁实现共享的目的。

Lock分两类: `sync.Mutex`和`sync.RWMutex`。 互斥锁是独占锁，只能上锁一次，然后解锁一次。 读写互斥锁是所有的 reader 共享一把锁或是一个 writer 独占一个锁， 如果一个 reader 拿到锁了， 其他的 reader 还可以 lock 但是 writer 不能 lock。

![](https://app.yinxiang.com/FileSharing.action?hash=1/127db79407cefb787c9b39db08d2b259-17251)

Lock从语义上来说就是一个原子操作。 也就是加锁的逻辑是原子操作，只有操作完成后(解锁)才可以做其它操作。

在Golang中通过`atomic.CompareAndSwapInt32`实现原子操作。

```
func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		if race.Enabled {
			race.Acquire(unsafe.Pointer(m))
		}
		return
	}
```
`CompareAndSwapInt32`是汇编实现的， 下面抄自其它人的汇编注释。
```
TEXT ·CompareAndSwapInt32(SB),NOSPLIT,$0-17
  JMP ·CompareAndSwapUint32(SB)

TEXT ·CompareAndSwapUint32(SB),NOSPLIT,$0-17
  MOVQ  addr+0(FP), BP      // 第一个参数命名为addr，放入BP(MOVQ，完成8个字节的复制)
  MOVL  old+8(FP), AX       // 第二个参数命名为old，放入AX
  MOVL  new+12(FP), CX      // 第三个参数命名为new，放入CX
  LOCK                      // 锁内存总线操作，防止其它CPU干扰（这里没有UNLOCK，我想是CPU执行完操作会自动UNLOCK吧，上面的硬件层面原子操作有提及）
  CMPXCHGL  CX, 0(BP)       // CMPXCHGL，该指令会把AX中的内容和第二个操作数中的内容比较，如果相等，那么把第一个操作数内容赋值给第二个操作数
                            // 即：if(*addr == old); then *addr = new
  SETEQ swapped+16(FP)
  RET
```

延伸一下以下几类操作都是原子操作：

+ CAS操作。比互斥锁乐观，Compare-And-Swap，可能有不成功的时候。
+ Swap操作。和上面不同，直接交换。
+ 增减操作。原子地对一个数值进行加减
+ 读写操作。防止读取一个正在写入的变量，我们的读写操作也需要做到原子。


通过Lock，原子操作将state置为1，以后其它Goroutine看到state==0，则表示其已经锁上了，等待下次轮询判断。

通过UnLock,原子操作将state置为0，其它Goroutine就可以继续竞争了。


这就是goalng实现内存数据一致性的两个方案。
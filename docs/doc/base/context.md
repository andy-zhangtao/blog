# 控制Goroutine超时退出的几种方法

Go最强大的地方莫过于goroutine了。写golang如果不使用goroutine，那就是暴殄天物。 但goroutine虽好，如果不好好控制，就会适得其反。下面介绍几种控制goroutine的方法。

**下文的一个重要基础是goroutine是可以正常退出的，如果goroutine里面存在bug导致无法正常退出，那么还是先fix bug吧**

## 典型的watch大法

watch大法简单来说，就是传入一个context with timeout。 然后goroutine内部watch contenxt。如果context超时退出，那么goroutine也就同步退出。

典型的代码结构如下:

```golang
func watchContext(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			return nil
		default:
			// do some thing
			return nil
		}
	}
}
```

利用`select`满足即触发的特性，当`watchContext`执行的时候，首先会执行default。 当default执行完以后，通过for循环会再次进入select的逻辑。如果context到达timeout，那么ctx.Done()就满足条件，此时watchContext就会退出。

调用这个`watchContext`的代码如下:

```golang
func main(){
    ctx, cancle := context.WithTimeout(2*time.Second)
	defer cancle()

	go watchContext(ctx)

    // do other thing
}
```

这种模型的弊端在于：如果`watchContext`里面`default`的执行逻辑 << context的超时时间，那么就存在重复执行的风险。因此需要自行设计防重复机制。 

如果做好防重复机制之后，这种模式可以应对70%的场景。 但我们不会止步于此，再看下面一种进阶用法。

## 多个goroutine的超时管理

上面`watchContext`只处理一个事件，但如果`watchContext`内部也需要重建goroutine时，又该如何处理呢？

也就是下面的模型：


```golang
func longJob(){
	// do some long thing
}

func watchContext(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			return nil
		default:
			go longJob()
			return nil
		}
	}
}
```

在这种模型中，我们可以通过context管理`watchContext`的退出，但无法管理内部logJob的退出。 

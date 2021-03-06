![image](./dou.png)
豆豆Pool
## GOLANG简单的工作池
本repo是一个简单的golang工作池，仅供交流和学习使用。golang工作池的作用是可以限制goroutine的启动数量。
# use
````golang
    //NewTask是放到工作池当中运行的函数。使用的时候需要先实例化他
	w := pool.NewTask(func() error { fmt.Println(time.Now());return nil })
	//实例化工作池
	p := pool.NewPool(3)
	//这里启用另外一个goroutine向worker当中写入，不然会出现all goroutines are asleep，需要从管道中获得一个数据，而这个数据必须是其他goroutine线放入管道的
	go func() {
		for {
			p.Worker <- w //把需要运行的函数依次放入工作池。
		}
	}()
	p.Run()
````

# 讨论

## 为什么需要工作池？
一般情况下，goroutine在操作系统上只要你的硬件资源够它是可以无限启动的。但是如果出现大规模的启动goroutine的情况会造成大量占用系统资源，我们知道普通的部署一个golang应用的时候操作系统不仅仅会运行golang程序还有其他辅助的程序运行，所以理论上讲工作池的目的就是为了限制golang的启动数量，保证不会出现硬件计算资源溢出的情况。

## 实际我们真的需要工作池吗？
理论上来讲，我们其实不需要在golang层面设置工作池的。如果是网络请求大部分时候我们会使用nginx或者其他网关，中间件作为golang程序的代理，我们可以在请求访问流量进入到golang程序之前使用nginx或者其他中间件限制流量或者使用熔断机制来保证我们的golang程序不会开满goroutine造成硬件计算资源溢出的情况。
如果流量小完全没必要限制goroutine
如果流量大没有熔断机制对整个服务都是很危险的。

## 工作池性能的讨论

我在编写自己的工作池之前使用了很多其他的工作池，我发现工作池本身并不会提升执行效率，反而会拖慢效率，使用工作池程序执行时间大概是原来的2/3,但是有一些pool会在一定程度上节省内存，比如ants，但内存的节省我觉得更多的是在于数据结构的复用，没有大量的重复创建数据对象导致的内存节省。

## goroutine是否需要像其他编程语言一样使用I/O多路复用？
根据我在网上可以搜索到的资料，以及我自己的理解，golang本身是有一个pool用来复用goroutine的，所以我们并不需要自己再去实现一个多路复用的功能，反而会拖慢程序。

## 其他
以上论调并没有严谨的验证过，只是个人遇到的情况分享，希望和大家共同讨论学习，共同进步。
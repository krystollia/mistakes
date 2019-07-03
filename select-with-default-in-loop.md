## 来看一段写的不怎么好的代码：
```go
func doSomething(interval float64) {
	regularTicker := time.NewTicker(time.Minute * time.Duration(60.0*interval))
	defer regularTicker.Stop()
	begin := true
	for {
		select {
		case <-regularTicker.C:
			// 以后周期性调用
			doSomeTimeConsumingthing()
		default:
			if begin {
				// 初始时做一次
				begin = false
				doSomeTimeConsumingthing()
			}
		}
}
```

注意到了吗，在循环里使用了 select，且带有 default，关于 for，select，default的描述，有一篇言简意赅的文章来自 [studygolang](https://studygolang.com/articles/6546):

> ### for 
> 如果for循环体一直处于繁忙状态，则cpu被一直抢占，而cpu居高不下，如果循环体有io等待则会出让cpu，不会一直抢占
> ### select
> 监听各case的io事件，各case必须都是 chan，一旦有一个case触发io则执行case块
> 如果没有default，则select会被一直阻塞，如果有default，则在没有io事件时，直接执行default块，退出select
> ### for  select
> 在有default的情况下，很容易造成类似for死循环 抢占cpu的情况

以上代码，在 select 时如果 regularTicker 的周期还未到，`case <-regularTicker.C` 不成立，会走default，一直在对 begin 进行判断，在线上环境占用了 100% 的 CPU。

## 改良后的代码：

```go
func doSomething2(interval float64) {
	regularTicker := time.NewTicker(time.Minute * time.Duration(60.0*interval))
	defer regularTicker.Stop()
	doSomeTimeConsumingthing()
	for range regularTicker.C {
		doSomeTimeConsumingthing()
	}
}
```

## 结论
for 和 select 结合有时无法避免，谨慎选择 default 所做的操作。

能用 for range 尽量用 for range。

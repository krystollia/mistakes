```
  var m map[string]string
  go func() {
  	m["k"] = "v"
  }()
  for k,v := range m {
  	...
  }
```

以上代码，在golang1.8之前两个goroutine同时访问map，是不会导致crash的。而golang1.8的引入了更严格的检测，会在map出现竞争访问时将程序crash。详见 [golang1.8 release notes](https://golang.org/doc/go1.8#mapiter)

避免map竞争访问，应引入mutex：

```
  var m map[string]string
  var mu sync.Mutex
  go func() {
  	mu.Lock()
  	m["k"] = "v"
  	mu.UnLock()
  }()
  mu.Lock()
  for k,v := range m {
  	...
  }
  mu.UnLock()
```
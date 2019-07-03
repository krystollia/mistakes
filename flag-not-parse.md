## 定义了 flag 但没有 parse

```
  fs := flag.NewFlagSet("name", flag.ExitOnError)

	mongoAddr := fs.String("mongo-addr", "some_adress:some_port", "Specify the raw data mongo database URL")

	session, err := mgo.DialWithTimeout(*mongoAddr, time.Duration(600)*time.Second)
	if err != nil {
		log.Fatal("Can not connect MongoDB, err:", err)
	}
	session.SetMode(mgo.Monotonic, true)
	defer session.Close()
  
  // 往 session 所在的 mongo 写一堆数据
```

发现问题了吗？只定义了 flag 但没有进行 parse，所以使用的是默认的地址，而默认的地址是线上地址。于是每次运行发现本地数据库怎么什么都没有！！然后重复往线上数据库写了一堆数据。


需要在 flag 定义完后加一行：
```
if err := fs.Parse(os.Args[1:]); err != nil {
	os.Exit(1)
}
```

这个愚蠢的bug，我们项目的人每个人都遇到过，包括我自己。

好的习惯：
默认地址一定要写成本地地址，以保证在没有设置 flag 的情况下对线上数据库进行随意读取，而线上地址也是不应该暴露在代码中的。

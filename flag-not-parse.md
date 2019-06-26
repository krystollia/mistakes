## 定义了 flag 但没有 parse

```
  fs := flag.NewFlagSet("name", flag.ExitOnError)

	mongoAddr := fs.String("mongo-addr", "47.94.181.104:31935", "Specify the raw data mongo database URL")
	dir := fs.String("dir", "", "dir to temporary save data")

	session, err := mgo.DialWithTimeout(*mongoAddr, time.Duration(600)*time.Second)
	if err != nil {
		log.Fatal("Can not connect MongoDB, err:", err)
	}
	session.SetMode(mgo.Monotonic, true)
	defer session.Close()
  
  // 往 session 所在的 mongo 写一堆数据
```

发现问题了吗？在运行

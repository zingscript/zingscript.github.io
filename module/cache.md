---
name: cache 模块
sort: 2
---

# 缓存模块

beego 的 cache 模块是用来做数据缓存的，设计思路来自于 `database/sql`，目前支持 file、memcache、memory 和 redis 四种引擎，安装方式如下：

	go get github.com/astaxie/beego/cache
	
## 使用入门

首先引入包：

	import (
		"github.com/astaxie/beego/cache"
	)

然后初始化一个全局变量对象：

	bm, err := NewCache("memory", `{"interval":60}`)

然后我们就可以使用bm增删改缓存：

	bm.Put("astaxie", 1, 10)
	bm.Get("astaxie")
	bm.IsExist("astaxie")
	bm.Delete("astaxie")

## 引擎设置

目前支持四种不同的引擎，接下来分别介绍这四种引擎如何设置：

- memory

	配置信息如下所示，配置的信息表示 GC 的时间，表示每个 60s 会进行一次过期清理：
	
		{"interval":60}													
- file

	配置信息如下所示，配置 `CachePath` 表示缓存的文件目录，`FileSuffix` 表示文件后缀，`DirectoryLevel` 表示目录层级，`EmbedExpiry` 表示过期设置
	
		{"CachePath":"./cache","FileSuffix":".cache","DirectoryLevel":2,"EmbedExpiry":120}
		
- redis

	配置信息如下所示，redis 采用了库 [redigo](http://github.com/garyburd/redigo/redis)，表示 redis 的连接地址：
	
		{"conn":":6039"}
		
- memcache

	配置信息如下所示，memcache 采用了 [vitess的库](http://code.google.com/p/vitess/go/memcache)，表示 memcache 的连接地址：	
	
		{"conn":"127.0.0.1:11211"}	
		
## 开发自己的引擎

cache 模块采用了接口的方式实现，因此用户可以很方便的实现接口，然后注册就可以实现自己的 Cache 引擎：

	type Cache interface {
		Get(key string) interface{}
		Put(key string, val interface{}, timeout int64) error
		Delete(key string) error
		Incr(key string) error
		Decr(key string) error
		IsExist(key string) bool
		ClearAll() error
		StartAndGC(config string) error
	}		

用户开发完毕在最后写类似这样的：

	func init() {
		Register("myowncache", NewOwnCache())
	}
		
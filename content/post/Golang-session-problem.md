+++
title = "Golang session problem"
menu = "main"
Categories = ["Development","GoLang"]
Tags = ["Development","golang"]
Description = ""
date = "2017-02-19T21:03:06+08:00"

+++

# Golang Session 使用遇到的问题

以前从事嵌入式工作，没有做过多用户的处理，比如web服务器上后台管理人员与普通用户的区别。

最近自己在写一个认证服务器的简单管理功能，想优化下就增加了admin管理权限。

## 初步设计
通过通过mux来管理路由。然后在访问/admin/路径下的API时需要检测是否登陆。

## 实现中遇到的问题

1. mux的具体实现不知道，需要进一步了解，今后的文章要分析透彻。
2. google了很多的管理登陆的文章，发现都是需要在处理的API内部做判断登陆，这点我感觉繁琐了，增加一个中间件不就好了吗，但是怎么增加是一个问题。
3. 另外一个就是session的使用中遇到的了，我明明增加了session，但是就是没有收到数据。


## 中间件验证登陆解决方案
通过mux建立路由，所有访问/admin/目录下的请求需要先验证是否登陆，即判断是否存在session。

只需要

```
    router := mux.NewRouter()
	adminRoutes := mux.NewRouter()

	router.HandleFunc("/", index)

	adminRoutes.HandleFunc("/admin/", adminIndex)
	...
	
	router.PathPrefix("/admin").Handler(negroni.New(
		NewCheckLogin(),
		negroni.Wrap(adminRoutes),
	))
	// /account/login
	router.HandleFunc("/account/login", login)
	http.ListenAndServe(":8080", router)
```

negroni.New(New)eckLogin(),negroni.Wrap(adminRoutes)
这句是进行验证的，具体原理还是需要去研究下，目前不是非常的明白。


## Session的使用比较奇怪，也是需要后续去学习领悟的地方。

除go基本类型外，复杂对象结构存储必须先注册。

```
import(
		"encoding/gob"
		"github.com/gorilla/sessions"
	)

	type Person struct {
		FirstName	string
		LastName 	string
		Email		string
		Age			int
	}

	type M map[string]interface{}

	func init() {

		gob.Register(&Person{})
		gob.Register(&M{})
	}
```
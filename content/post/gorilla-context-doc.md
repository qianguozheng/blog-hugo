+++
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2017-02-23T17:26:24+08:00"
menu = "main"
title = "gorilla context doc"

+++

包context存储的值在一个请求的声明周期中是共享的。

>注意：gorilla/context, 比`context.Context`诞生的早，不能很好的支持[`http.Request.WithContext`](https://golang.org/pkg/net/http/#Request.WithContext) (引入于Go1.7). 你应该只使用gorilla/context或者换到新的 `http.Request.Context()`.

例如， 一个路由可以设置从URL中提取出来的变量， 并且程序的处理者可以访问这些值， 或者这个可以用来存储会话值在请求的最后。还有集中其他的常用方法。

这个想法是Brad Fitzpatrick在go-nuts的邮件列表中提出的：
    
        http://groups.google.com/group/golang-nuts/msg/e2d679d303aa5d53
        
这里是基础用法： 首先定义你需要的键。 键的类型是interface{} 所以你可以存储任何类型。

这里我们定义一个键使用常用的int类型来必买名称冲突：

    package foo
    
    import (
        "github.com/gorilla/context"
    )
    
    type key int
    
    const MyKey key = 0
    
然后设置一个变量。 变量是绑定到http.Request对象的， 所以你需要请求的实例来设置值：

    context.Set(r, MyKey, "bar")
    
    
程序可以后续使用你提供的键来访问变量：

    func MyHandler(w http.ResponseWriter, r *http.Request){
        // val is "bar".
        val := context.Get(r, foo.MyKey)
        
        // returns ("bar", true)
        val, ok := context.GetOk(r, foo.MyKey)
        //...
    }

这是最基础的使用方式。 我们在下面讨论一些别的用法。

所有类型都可以存储在context中。强制使用一种类型， 把键变得私用，并且包裹Get() 和Set()来接受和返回特定类型的值：

    type key int
    
    const mykey key = 0
    
    // GetMyKey returns a value for this package from the request values.
    func GetMyKey(r *http.Request) SomeType{
        if rv := context.Get(r, mykey); rv != nil {
            return rv.(SomeType)
        }
        return nil
    }
    
    // SetMyKey sets a value for this package in the request values.
    func SetMyKey(r *http.Request, val SomeType){
        context.Set(r, mykey, val)
    }
    
变量必须在请求的结尾清除，以删除所有存储的值。这个可以在http.Handler中做， 在请求被处理的时候。 只需要调用Clear()

    context.Clear(r)
    
...或者使用 ClearHandler(), 这个方便的包裹来一个http.Handler在请求声明周期的末尾来请求变量

包gorilla/mux 和gorilla/pat中的路由调用Clear()所以如果你使用他们中的一个，你不需要手动清除上下文。



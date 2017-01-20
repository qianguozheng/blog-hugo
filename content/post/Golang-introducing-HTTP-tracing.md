+++
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2017-01-20T10:09:56+08:00"
menu = "main"
title = "Golang introducing HTTP tracing"

+++


# 介绍

在Go 1.7我们引入了HTTP跟踪， 用来收集HTTP客户端请求生命周期中详细信息的工作。包在net/http/httptrace. 手机的信息可以用来调试延时问题，服务监控，写适配系统等。

# HTTP 事件

httptrace包提供了许多钩子在HTTP生命周期中收集信息。这些事件包括：

* Connection creation
* Connection reuse
* DNS lookups
* Writing the request to the wire
* Reading the response

# 跟踪事件

你可以通过放置一个*httptrace.ClientTrace 包含钩子函数在请求的context.Context中来开启http追踪。各种http.RoundTripper实现通过查找上下文的 *httptrace.ClientTrace和调用相关的钩子函数来汇报内部事件

最终作用于请求的上下文，用户应该将一个 *httptrace.ClientTrace防到请求的上下文在开始请求之前。

```golang
    req, _ := http.NewRequest("GET", "http://example.com", nil)
    trace := &httptrace.ClientTrace{
        DNSDone: func(dnsInfo httptrace.DNSDoneInfo) {
            fmt.Printf("DNS Info: %+v\n", dnsInfo)
        },
        GotConn: func(connInfo httptrace.GotConnInfo) {
            fmt.Printf("Got Conn: %+v\n", connInfo)
        },
    }
    req = req.WithContext(httptrace.WithClientTrace(req.Context(), trace))
    if _, err := http.DefaultTransport.RoundTrip(req); err != nil {
        log.Fatal(err)
    }
```

在请求流程中，http.DefaultTransport会在事件发生时出发钩子。上面的程序会打印DNS信息当DNS查询完成时。同样的会打印连接信息当链接与请求主机建立连接时

# 使用http.Client跟踪

跟踪机制设计用来跟踪单个的http.Transport.RoundTrip的声明周期事件。然而，一个客户端可以发起多个round trips来完成HTTP请求。例如，URL重定向，注册的钩子被调用多次当客户端跟踪HTTP重定向，发起多个请求。用户负责识别类似的事件在http.CLient中。下面的程序识别通过httpRoundTripper包裹识别当前的请求。

```
package main

import (
    "fmt"
    "log"
    "net/http"
    "net/http/httptrace"
)

// transport is an http.RoundTripper that keeps track of the in-flight
// request and implements hooks to report HTTP tracing events.
type transport struct {
    current *http.Request
}

// RoundTrip wraps http.DefaultTransport.RoundTrip to keep track
// of the current request.
func (t *transport) RoundTrip(req *http.Request) (*http.Response, error) {
    t.current = req
    return http.DefaultTransport.RoundTrip(req)
}

// GotConn prints whether the connection has been used previously
// for the current request.
func (t *transport) GotConn(info httptrace.GotConnInfo) {
    fmt.Printf("Connection reused for %v? %v\n", t.current.URL, info.Reused)
}

func main() {
    t := &transport{}

    req, _ := http.NewRequest("GET", "https://google.com", nil)
    trace := &httptrace.ClientTrace{
        GotConn: t.GotConn,
    }
    req = req.WithContext(httptrace.WithClientTrace(req.Context(), trace))

    client := &http.Client{Transport: t}
    if _, err := client.Do(req); err != nil {
        log.Fatal(err)
    }
}
```

程序将会跟踪重定向google.com 到www.google.com并打印如下：

```
Connection reused for https://google.com? false
Connection reused for https://www.google.com/? false
```

传输在net/http包支持跟踪http/1和http/2

如果你是一个定制的http.RoundTripper实现作者，你可以通过检查请求上下文并在相应的事件发生时触发钩子函数

# 结论

GO的HTTP跟踪对于那些感兴趣调试HTTP请求延时和些网络外部负载调试工具时。 通过开启这个功能，我们希望可以看到HTTP调试，性能。可视化工具来自社区例如httpstat.

By Jaana Burcu Dogan

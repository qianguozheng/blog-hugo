+++
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2017-02-20T11:03:48+08:00"
menu = "main"
title = "Golang gorilla mux"

+++

翻译 github.com/gorilla/mux/doc.go


包mux实现了路由请求与分发。

mux代表“HTTP请求多路复用”。 像标准的http.ServeMux, mux.Router匹配进来的请求和一些注册的路径， 并且调用处理函数来处理匹配URL或其他条件的。 主要特色如下：

* 请求可以基于URL主机， 路径，路径前缀， schemes, 头，请求值， HTTP方法或者特定的匹配方式。
* URL主机可路径可以通过可选的正则表达式来匹配。
* Registered URLs can be built, or "reversed", which helps maintaining references to resources.
* 路由可以作为子路由： 子路由仅仅当父路由匹配的时候才会比对。 这种方式可以定义遗嘱路径拥有共同的条件特征，如主机，路径牵住，或者其他的重复属性。 额外的，这个优化请求匹配。
* 实现了http.Handler接口所以这个和标准的http.ServeMux兼容


我们来注册一些URL路径和处理函数：

    func main(){
        r := mux.NewRouter()
        r.HandleFunc("/", HomeHandler)
        r.HandleFunc("/products", ProductsHandler)
        r.HandleFunc("/articles", ArticlesHandler)
        http.Handle("/", r)
    }

我们注册了3个路径映射URL给处理函数。这个与http.HandleFunc()类似: 如果一个请求的URL符合路径，相应的处理函数被调用， 以(http.ResponseWriter, *http.Request)作为参数。

路径可以有变量。 他们定义使用这种格式 {name} 或者 {name:pattern}. 如果一个正常的表达模式未被定义， 匹配的变量将会是任何值，直到下一个斜线。例如：

    r := mux.NewRouter()
    r.HandleFunc("/products/{key}", ProductHandler)
    r.HandleFunc("/articles/{category}/", ArticlesCategoryHandler)
    r.HandleFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler)
    
组可以在模式内使用，只要没有捕捉(?:re). 例如：

    r.HandleFunc("/articles/{category}/{sort:(?:asc|desc|new)}", ArticleCategoryHandler)
    
名字可以用来创建一组路由变量的映射，可以通过调用mux.Vars()获得：

    vars := mux.Vars(request)
    category := vars["category"]
    
如果任何的capturing组存在， mux会panic()在解析的过程中。阻止这种情况发生，转换任何的capturing组为非capturing，例如 转换"/{sort:(asc|desc)}"为"/{sort:{?:asc|dest}}". 这是从之前的不可预测的行为转换来的当capturing组存在的时候。

这就是所有你需要知道的基础。更高级的选项在下面解释。

路由同样可以限制一个域名或者子域名。 定义一个主机模式来匹配， 他们同样可以有变量：

    r := mux.NewRouter()
    //Only matches if domain is "www.example.com".
    r.Host("www.example.com")
    //Matches a dynamic subdomain.
    r.Host("{subdomain:[a-z]+}.domain.com")
    
还有其他几种匹配可以添加，来匹配这种路径前缀：

    r.PathPrefix("/products/")

...或者HTTP方法

    r.Methods("GET", "POST")
    
...或者 URL schemes:
    
    r.Schemes("https")
    
...或者 头部值

    r.Headers("X-Requested-With", "XMLHttpRequest")
    
...或者 请求值

    r.Queries("key", "value")

...或者 使用定制匹配函数

    r.MatcherFunc(func(r *http.Request, rm *RouteMatch) bool{
        return r.ProtoMajor == 0
    })

...最终， 也可以组合几种匹配到一个路由中：

    r.HandleFunc("/products", ProductsHandler).
        Host("www.example.com").
        Methods("GET").
        Schemes("http")
        
一遍又一遍的设置同样的路径很烦人， 所以我们可以使用组来共享满足相同条件的路由：
    
    r := mux.NewRouter()
    s := r.Host("www.example.com").Subrouter()
    
然在在路由中注册子路由：

    s.HandleFunc("/products/", ProductsHandler)
    s.HandleFunc("/products/{key}", ProductHandler)
    s.HandleFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler)
    
以上三个URL路径只有当域名为 "www.example.com"时才会被测试, 因为子路有会首先被测试。这个不仅方便而且优化请求路径。你可以创建子路由结合任意树形的匹配器。

子路经可以用来创建域名或者路径“namespaces”: 你定义子路由在中间位置，然后其余部分可以基于子路由注册自己的路径

还有其他的子路由，当一个子路由有路径前缀时， 内部的路由使用它作为他们的基础：

    r := mux.NewRouter()
    s := r.PathPrefix("/products").Subrouter()
    //"/products/"
    s.HandleFunc("/", ProductHandler)
    // "/products/{key}/"
    s.HandleFunc("/{key}/", ProductHandler)
    // "/products/{key}/details"
    s.HandleFunc("/{key}/details", ProductDetailsHandler)
    
注意，路径提供给PathPrefix()代表一个“掩码”： 调用PathPrefix("/static/").Handler()意味着处理函数会传递任何请求符合"/static/*".这种方式很容易实现静态文件的路由。

```
func main()
    var dir string
    flag.StringVar(&dir, "dir", ".", "the directory to serve files from. Defaults to the current dir")
    flag.Parse()
    r := mux.NewRouter()
    
    // This will serve files under http://localhost:8000/static/<filename>
    r.PathPrefix("/static/").Handler(http.StripPrefix("/static/", http.FileServer(http.Dir(dir))))

	srv := &http.Server{
		Handler:      r,
		Addr:         "127.0.0.1:8000",
		// Good practice: enforce timeouts for servers you create!
		WriteTimeout: 15 * time.Second,
		ReadTimeout:  15 * time.Second,
	}

	log.Fatal(srv.ListenAndServe())
}
```
    
现在我们来看看如何建立注册URL。

路由可以是名称， 所有定义名称的路由可以有自己的URLs， 或者反转。我们定义一个名称通过调用Name()在路由上。例如：

    r := mux.NewRouter()
    r.HandleFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler).
    Name("article")
    
建立URL,获取路由，调用URL()方法，传递一系列的键值对给路由变量。对于之前的路径，我们可以这样做：
    
    url, err := r.Get("article").URL("category", "technology", "id", "42")

...并且结果会是url.URL带有下面的路径

    "/articles/technology/42"
    
这个同样对于主机变量：

    r := mux.NewRouter()
    r.Host("{subdomain}.domain.com").
        Path("/articles/{category}/{id:[0-9]+}").
        HandlerFunc(ArticleHandler).
        Name("article")
        
    // url.String() will be "http://news.domain.com/articles/technology/42"
	url, err := r.Get("article").URL("subdomain", "news",
	                                 "category", "technology",
	                                 "id", "42")

所有路由中的变量都是必须的，并且他们的值必须服务响应模式/这些需求保证生成的URL会一直符合注册路径 -- 唯一例外的是那些显示定义 “build-only”路由永远都不会匹配。

正则支持也存在于匹配路由中的头。例如，我们可以作：

    r.HeaderRegexp("Content-Type", "application/(text|json)")
    
...路由会匹配请求带有Content-Type为application/json以及application/text的。

同样可以为路由建立只有URL主机或者路径的路由：
使用方法URLHost()或者URLPath()替代/ 对于之前的路由，我们可以这样做：

    // "http://news.domain.com/"
	host, err := r.Get("article").URLHost("subdomain", "news")

	// "/articles/technology/42"
	path, err := r.Get("article").URLPath("category", "technology", "id", "42")
	
如果你使用子路由， 主机和路径单独定义可以建立如下：

    r := mux.NewRouter()
	s := r.Host("{subdomain}.domain.com").Subrouter()
	s.Path("/articles/{category}/{id:[0-9]+}").
	  HandlerFunc(ArticleHandler).
	  Name("article")

	// "http://news.domain.com/articles/technology/42"
	url, err := r.Get("article").URL("subdomain", "news",
	                                 "category", "technology",
	                                 "id", "42")
    
    

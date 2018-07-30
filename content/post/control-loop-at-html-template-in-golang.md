+++
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2018-07-30T10:55:21+08:00"
menu = "main"
title = "control loop at html template in golang"

+++

最近用go实现了一个管理后台，主要用来做一些路由器的配置，远程管理这些设备。

其中有一个功能是设计到循环的，所以采用了俄slice传递参数到html模板里面，但是这些参数是固定的，而且模板不好控制循环遍历slice, 所以写了个全局变量来控制

样例如下，想要输出想要的，不容易控制
```
{{range .Slice}}
<a>print anything you want</a>
{{end}}
```

实现
---

```
type FuncMap map[string]interface{}
定义几个模板函数
template.FuncMap {
    "var": newVar,
    "set": setVar,
    "cmp": cmpVar,
}
//声明模板的全局变量
func newVar(v interface{})(*interface{}, error) {
    x := interface{}(v)
    return &x, nil
}
//赋值模板的全局变量
func setVar(x *interface{}, v interface{}) (string, error) {
    *x = v
    return "", nil
}
//对比全局变量，这里和int类型对比
func cmpVar(x *interface{}, e int) bool {
		return (*x).(int) == e
}

//使用

{{$global_var := var 0}}

{{range $k,$v := .Slice}}
    {{if eq $v.Port 0}}
        <a>print something</a>
        {{set $global_var 1}}
    {{end}}
    {{if cmp $global_var 1}}
        <a>do another logic</a>
    {{end}}
{{end}}


```




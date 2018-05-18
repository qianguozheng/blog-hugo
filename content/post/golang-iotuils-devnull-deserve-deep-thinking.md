+++
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2018-05-18T14:41:06+08:00"
menu = "main"
title = "golang iotuils devnull deserve deep thinking"

+++


[!ioutils源码地址](https://golang.org/src/io/ioutil/ioutil.go)

```
type devNull int


// devNull implements ReaderFrom as an optimization so io.Copy to

// ioutil.Discard can avoid doing unnecessary work.

var _ io.ReaderFrom = devNull(0)


func (devNull) Write(p []byte) (int, error) {

	return len(p), nil

}


func (devNull) WriteString(s string) (int, error) {

	return len(s), nil

}


var blackHolePool = sync.Pool{

	New: func() interface{} {

		b := make([]byte, 8192)

		return &b

	},

}


func (devNull) ReadFrom(r io.Reader) (n int64, err error) {

	bufp := blackHolePool.Get().(*[]byte)

	readSize := 0

	for {

		readSize, err = r.Read(*bufp)

		n += int64(readSize)

		if err != nil {

			blackHolePool.Put(bufp)

			if err == io.EOF {

				return n, nil

			}

			return

		}

	}

}


// Discard is an io.Writer on which all Write calls succeed

// without doing anything.

var Discard io.Writer = devNull(0)
```

使用样例，说到底这个devNull到底有什么用呢？ 模拟一个黑洞，用来吃数据，而且黑洞是接口类型，
这个就是我能想到的仅有的解释了。


说实话，我自认为对于Go的语法已经懂得的不少了，但是看到了这些package的实现，还是觉得自己
太自已为是了。

`var Discard io.Writer = devNull(0)`  //这个定义以及初始化我就没这个想象力！

`var _ io.ReaderFrom = devNull(0) `

//这个定义又如何理解呢？ 没有定义变量，想说明什么？ 让devNull必须实现ReadFrom方法？
//这是我能想到的唯一目的了。


```
    package main  
      
    import (  
        "fmt"  
        "io"  
        "io/ioutil"  
        "strings"  
    )  
      
    func main() {  
        a := strings.NewReader("hello")  
        p := make([]byte, 20)  
        io.Copy(ioutil.Discard, a)  
        ioutil.Discard.Write(p)  
        fmt.Println(p)　　　　　　//[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]  
    }  
```

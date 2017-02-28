---
title: golang共享库
date: 2016-01-20
---
包的形式去管理代码可以使代码结构更加清晰。java，python等等语言采用了了这种方式，golang是用来关键字package和import，也是包级别的代码管理。然而，在程序上也有种管理方式，大名鼎鼎的有so和dll，可以从源码上编译成程序来被其余程序调用。一般情况下我们称为library，也叫共享库。golang从1.5的版本以后即支持共享库了。

编译选项：--buildmode=value

value值有：
1. **shared** 这是golang提供给自身调用的共享库。
2. **c-shared** 用来生成c的共享库，这是极具诱惑力的，意味着golang可以写给其余语言调用的库了，例如python里面的[module](https://blog.filippo.io/building-python-modules-with-go-1-5/),甚至php的扩展也可以由golang来实现了。

### talk is cheap

我当前目录下有：
```
|_test
|      |_test.go
|_main.go
```
``` 
$cat test/test.go
package test
func Test(){
    println("hello world")
}
$cat main.go
package main
import "test"
func main(){
    test.Test()
}
$go install --buildmode=shared -linkshared test    #安装test包做为golang的共享库
$go build -linkshared main.go                      #用了共享的地方，记得打上-linkshared的标记
$./main
hello world
```
当你改变了test里面的代码重新安装下这个共享库，不需要去重新编译main程序，只需要重启main程序即可生效。遗憾的是，目前golang官方不支持动态加载共享库，不然，热更新可以很容易实现了。

另外，有意思的一点是，即使你在更改完共享库之后，不重启main程序，而是另外启动main程序，俩个程序运行的结果是不同的。

在安装共享库的时候，有一个[bug](https://github.com/golang/go/issues/12236)要注意下,目前golang编译时，需要明确指定库文件。


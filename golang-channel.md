---
title:  channel!channel!
date:  2016-01-20
---
进程之间的通信是很复杂的，线程之间共享全局数据则妙的多。而goroutine之间的通信依靠的channel就是妙不可言了。

#### channel的一些特性
###### 1. 有缓存和无缓存的channel
```
    c:=make(chan int,10)  //这是有缓存的
    c0:=make(chan int)    //这是无缓存的
```
这样一点在初学时，是关注的比较多的，无缓存的channel要求goroutine之间的读写是同时的。

###### 2. 带读写方向的channel
```
    var c <-chan int   //此处的channel是只能读的
    var c0 chan<- int  //此处的channel是只写的
```
其实这里是很有意思的，也推荐多使用带读写方向的channel，goroutine之间的分工就更加明显了。
##### 3.channel的关闭
```
    c:=make(chan int,10)    //实例化一个有缓存的channel
    close(c)                //关闭channel
    a,ok:=<-c               //在关闭之后，读取channel，这里是有问题的，已关闭的channel读不到数据，这里用了一个多返回值来判断是否关闭了channel
    b:=<-c                  //直接读数据是会引发panic的
```
这里有一点注意，就是channel的关闭问题，是否需要关闭呢！因为channel是goroutine之间的通信，各个goroutine共享了这个内存，不关闭是不是会引起内存泄露的问题呢！作为全局变量声明的，是不会被gc回收的。而像普通变量时，在所有引用channel的goroutine不在引用时，便会进行gc回收，所以不关闭，内存最终还是会被回收的。另外要注意的一点就是，只读channel不能关闭，仔细思考起来，这样一点也体现了golang对channel的设计用心。
##### 4. 遍历channel
```
   for v:=range c{
        println(v)
   } 
```
channel更像是一个管道，所以这里的遍历显得有点不可思议。其实这里如同循环的读取。
```
   for{
       if v,ok:=c;ok{
           println(v)
       }else{
           return
      }
   }
```
这样代码就臃肿了些。
##### 5.消息队列案例
在消息队列中，基本的也比较简单的模型有生产者和消费者模型，golang能以很美妙的姿态实现它
```
    package main
    func main(){
        c:=make(chan int,10)
        go Consume(c)
        go Consume(c)
        go Create(c)    
        select{}        //永久休眠，保证其余goroutine的时间片
    }
    func Consume(c <-chan int){
        for v:=range c{
            println(v)
        }
    }
    func Create(c chan<- int){
        for i:=0;i<1000;i++{
            c<-i
        }
        close(c)       //在写入处关闭channel
    }
```
这部分代码就像实现了三个线程（实际上是三个协程），线程之间的数据以队列（channel）的方式来处理。

另外值得一提的是，golang在channel设计中，在只读的channel处提供多返回值来检测channel是否关闭，而只写入的channel则没有办法判断channel是否关闭，如果写入已经关闭的channel会引发panic。与之相反的是，在只读channel处是不允许关闭的，会产生语法错误。可写入的channel自然是可以关闭的。


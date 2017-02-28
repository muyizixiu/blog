---
title:golang routine调度中的核心设置
date:2015-10-10
---
观察一下在golang中多个goroutine是怎么调度的，上代码：

    package main
    import(
        "fmt"
    )
    func main(){
     go func(){//goroutine 持续打印1
                i:=1
                fmt.Println(i)
            }()
            for{//main 程持续打印0
            i:=0
            fmt.Println(i)
        }
   
    }
实际上在打印一段1之后会一直打印0，也就是没有切回main之外的goroutine。和平时理解的多线程似乎是不一样的，似乎跟一个普通函数的区别就是没等结束就跳出运行。来一段另外的代码：

    package main
    import(
         "fmt"
         "time"
    )
    func main(){
    go func(){//goroutine 持续打印1
            i:=1
            fmt.Println(i)
        }()
        for{//main 程持续打印0
                  time.Sleep(time.Second*1)//每次阻塞一秒再打印
            i:=0
            fmt.Println(i)
        }
    
    }
这里代码唯一不同的就是在for循环中增加了休眠，这里就可以发现别的goroutine在执行了。很奇怪是不是？又有点像多线程了。

然而在实际工作中main程序会一直执行，要不特意进行阻塞（推荐个小技巧，select{}就能进行永久的休眠）。碰到这种问题其实只是一行简单的小代码就可以解决的，这时候就要祭出golang的runtime了。

    package main
    import(
        "fmt"
        "runtime"
    )
    func main(){
        runtime.GOMAXPROCS(2)//运行机器是双核
        go func(){//goroutine 持续打印1
            i:=1
            fmt.Println(i)
        }()
        for{//main 程持续打印0
            i:=0
            fmt.Println(i)
        }
   
    }
这段代码取消了原有的主程休眠，增加了运行时的cpu数量，这里的运行结果就可以看到预计的各个goroutine不停切换的效果了。是不是跟理想的多线程一样的呢！这个有待进一步探讨！

上结论：`在golang1.4及一下，默认单核上运行多个goroutine时，main主程是要长时间占据cpu时间片的，其余goroutine在首次执行之后只有等待main主动让出cpu才能执行，main程序如果不让出cpu的话，程序是无法切回别的goroutine。`

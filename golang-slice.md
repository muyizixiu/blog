---
title:golang将slice转array
date:2015-10-29
---
最近写代码，遇到个问题，怎么将一个slice转化为数组。

golang中将数组切片就变成了slice：

    slice=array[m:n]
但是slice转换成array就是麻烦事了。数组初始化之后下标是可以直接访问的。

    package main
    import()
    func main(){
        var a [4]int
        a[3]=1
    }
这样提供了一个可以遍历的方式去用slice给数组赋值。

    for i:=0;i<4;i++{
        a[i]=slice[m+i]
    }
三行，多了个变量i，如果向array转slice一样，一行就行，多好。
一行：
    
    copy(a[:],slice[m:m+4])
这就是答案。有点诡异是吧，其实跟这个函数无关，真正有用的是a后面的[:]。其实这里只是用到了slice是指向array的指针这个特点。将a生成一个切片，然后引用赋值。这样相当于给array赋值了。这就是如何给用slice给array赋值的故事。

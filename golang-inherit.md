---
title: golang的组合之道(1)
date: 2015-12-03
categories:
- 技术
---
作为程序员，是很喜欢有对象的。虽然编程思想一般是面向需求，但也要面向对象才行。golang是21世纪的编程语言，函数式编程的思想有，面向对象方式也有。golang可以简单地实现面向对象的几个特征：继承，封装，和接口抽象。

### 1.  来自组合的继承
组合是可以实现继承的。


## A
```golang
    type family struct{
        FamilyName string
    }
    type member struct{
        family
        LastName string
    }
```
在这里，member结构体继承family的属性。有意思的是，golang中的继承只是一个语法糖，A的代码可以等效于：

## B

    type family struct{
        FamilyName string
    }
    type member struct{
        family family
        LastName string
    }

但两者并非一致，在A处访问FamilyName属性可以obj.family.FamilyName或者obj.FamilyName两种方式，然而B处的代码只能使用obj.family.FamilyName来访问属性。

当使用obj.FamilyName来访问变量时，就像面向对象中的继承一样了。不过在golang中，这称之为组合。组合的使用更加的清晰，比起c++中复杂的继承，无疑golang是做到了异常的简洁。

在应用组合的过程中也不要担心有什么副作用，如A和B的代码一样，他们在使用上虽有相似之处，但不会引起模棱两可的副作用的。如下所示：

 
    package main

    import "encoding/json"

    type A struct {
	    Name string
    }

    type B struct {
	    NickName string
	    A
    }
    type C struct {
	    NickName string
	    *A
    }

    func main() {
	    a := A{"Jacob"}
	    b := B{"lisa", A{"lisonana"}}
	    b.A.Name = "lisam"//此处用(.A)访问，实际上直接访问也是可以的，即（b.Name="lisam"）
	    aStr, _ := json.Marshal(a)
	    bStr, _ := json.Marshal(b)
	    println("a: ", string(aStr))// a:  {"Name":"Jacob"}
	    println("b: ", string(bStr))// b:  {"NickName":"lisa","Name":"lisam"}
	    d := C{"lisa", &A{"lisonana"}}
	    d.A.Name = "lisam"
	    //d.A = &A{"lisam"}
	    dStr, _ := json.Marshal(b)
	    println("d: ", string(dStr))// d:  {"NickName":"lisa","Name":"lisam"}
    }


注意到三处的打印的结果。

    package main

    import "encoding/json"

    type A struct {
	    Name string
    }

    type B struct {
	    NickName string
	    A A
    }
    type C struct {
	    NickName string
	    A *A 
    }

    func main() {
	    a := A{"Jacob"}
	    b := B{"lisa", A{"lisonana"}}
	    b.A.Name = "lisam"//此处不能用(.A)访问，即（b.Name="lisam"）是错误的
	    aStr, _ := json.Marshal(a)
	    bStr, _ := json.Marshal(b)
	    println("a: ", string(aStr))//a:  {"Name":"Jacob"}
	    println("b: ", string(bStr))// b:  {"NickName":"lisa","A":{"Name":"lisam"}}
	    d := C{"lisa", &A{"lisonana"}}
	    d.A.Name = "lisam"
	    //d.A = &A{"lisam"}
	    dStr, _ := json.Marshal(b)
	    println("d: ", string(dStr))// d:  {"NickName":"lisa","A":{"Name":"lisam"}}
    }

完全可以放心地把组合当成继承来使用。

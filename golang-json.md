---
title:  golang json的大小写技巧
date:  2015-10-16
---
golang中json格式有大小写要求，json中小写键名会无法导出，结构体小写也无法导出json对应键名。
就像这样（撸代码）：

       package main
        import(
                "fmt"
               "encoding/json"
        )
        type js struct{
                name string
        }
        func main(){
                j_:=js{name:"li"}
                k,_:=json.Marshal(j_)//已经把j转化为json流了
                fmt.Println(k)//nil
        }
看似造成了僵硬的json数据处理，以及和外部格式不一致！
   当然golang有考虑到这一点!

在结构体字段后加标签可以处理大小写问题：

        package main
        import(
                "fmt"
               "encoding/json"
        )
        type js struct{
                Name string    `json:"name"`
        }
        func main(){
                j_:=js{name:"li"}
                k,_:=json.Marshal(j_)//已经把j转化为json流了
                fmt.Println(k)//[123 34 110 97 109 101 34 58 34 108 105 34 125]
        }
反之，json流转化为结构体也不用担心大小写了。具体原理应该是golang通过反射拿到了tag然后转换，作为一种golang自身设计与json标准的一种衔接吧。但这里依旧还是要有对应的键名作为tag。其实，golang还提供另一种更加宽泛的转换，那就是map了，结构体的json转换处理可以有更好的安全应用，使代码更加严谨，map的json转换则十分宽松，对于不明确的json解析就很方便了。

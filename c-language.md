---
title: c语言学习速记
date: 2015-10-28
categories:
- 技术
---
以前闹着玩似地学习过c语言，现在出于工作安排，要应用c，做些应用层协议的实现，重新捡起c来，发现以前学的真真是用来输出下hello  world 而已。前事不忘后事之师也，就写一篇速记，来记录下c语言学到的重要节点。

当然撸一把代码

    #include <stdio.h>
    #define HELLO_WORLD "hello world"
    int main(void){               //是的，我是花括号不换行党
        printf(HELLO_WORLD);
        return 1;
    }

其实这代码没什么意义，致敬一下而已，下面是正文╮(╯▽╰)╭。

###  c语言速记

* c运算符
 1. ++a和a++，这玩意儿以前学的时候很苦恼人，其实，不用太过在意，这就是一个子表达式，相当于（++a)和(a++),不同点在于前者先做自加运算，后者先赋值给外部，然后再做自加运算。至于像这种++a + ++a这种看编译器心情的东西，绝对不要管，在c语言中的一个表达式中有很多子表达式时，不推荐多个位置出现同一变量。因为c语言中子表达式的执行顺序是看编译器心情的，其副作用不能确定。
 
 2. 表达式的副作用，c的编译器实现有很多，彼此不一定做了相同的实现，所以没有被明确写入c99，c89规范的东西也有很多，表达式有时候就会很奇怪。

            #include <stdio.h>
            #define false 0
            int main(void){
                 int a;
                 a=1;
                 if (false && ++a >0){
                       print("a equals to %d",a);//a equqls to 1
                 }
            return 1;
            }
    这里表达式++a没有进行运算。

            #include <stdio.h>
            #define true
            int main(void){
                 int a;
                 a=1;
                 if (true && ++a >0){
                      print("a equals to %d",a);//a equqls to 2
                 }
            return 1;
            }

       这里表达式++a运算了，这种子表达式的执行与运算符有关（这里是&&），&&运算符在左表达式为false的情形下，不执行右表达式，这种细节其实很累人的，有些还跟相关的编译器实现有关，推荐更加直接明了的写法。

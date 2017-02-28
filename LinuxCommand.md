---
title: Linux niubility 命令集
date: 2015-12-15
---
对于绝大多数应用程序，系统调用（syscall）是被封装隐藏好了的，但大多数函数操作都有涉及的。文件读写，io操作，网络请求等等属于操作系统调度的操作都跟操作系统密切相关，所以对于程序（尤其是一定规模的程序），其性能，状态的分析，不仅仅是从代码层面上进行分析，也要注重在操作系统层面上进行分析。linux系统提供的很多命令，在这一方面，是极其厉害的。


###  一. lsof （list open files）
```
Useage：

    无选项时，lsof列出所有打开文件
    -a : 与逻辑
    -c : 指定进程
    -l : 显示用户ID
    -h : 帮助
    -t : 仅获取进程ID
    -U : UNIX套接口地址
    -i : 进行正则匹配
```
 其余的简单就可以探索明白，其中a和i参数解释一下

1. **-a** : 进行与逻辑操作，lsof命令可以有多个参数，在多个参数指定的条件下，默认所有条件是或逻辑操作的，在参数-a存在的情形下就变成了与逻辑
2. **-i** : niubility就在这里出现了。lsof -i[46] [protocol][@hostname|hostaddr][:service|port]使用格式虽然有点长，对于网络连接的把控也体现出来了。比如说：
 + 查看某一端口：lsof -i :22
 + 查看某一ip建立的连接：lsof -i @localhost
 
-i参数指定正则匹配，比起用管道|加grep匹配，这一参数还有更加明确的匹配格式，冒号加端口（:port），或者@符号加网络地址,都能进行更加明确的匹配。


linux中一切皆文件（宛如js中一切皆对象），lsof 命令在linux中就不仅仅局限于文件了，可以用于分析网络端口（忘掉netstat命令吧），ip连接，文件读写等操作


### 二. tcpdump

抓包素来很酷，极具Geek范，在linux命令行下也有niubility的抓包命令：tcpdump

tcpdump在网络上抓取数据，数据量大也可以保存下来，通过分析工具，例如wireshark，来分析数据。

tcpdump的命令本身其实很简单，网路上有很多资料，这篇[博客](http://bbs.chinaunix.net/thread-2222434-1-1.html)内容较为齐全。

tcpdump 重要的几个关键字和参数：
```

指定网络参数：net（网络域），port（指定端口），host（指定主机，可以是主机名，也可以是ip地址）

指定网络流方向:dst(去向),src（来源）

指定网卡：-i
```

具体用法如下：
```

\# tcpdump host localhost  
这是指定为本机回环地址的情形，在没有指定网卡（无参数-i）的情形下，自动指定为第一块网卡，在这里，如果回环地址不是第一块网卡，是不会有数据的。
\# tcpdump dst localhost -i lo
抓取所有流向本机回环地址的数据，且指定网卡为回环地址的网卡，（lo是网卡名称，可用ifconfig命令查看）
\# tcpdump host localhost or dst 192.168.1.1
and 和or 的与或逻辑是支持的
```

### 三. expect

还在为命令行交互而烦恼吗，还在讨厌密码太长不想输入吗？

expect是可以处理命令行交互的命令，命令行下的一些输入，比如输入密码，是需要交互的输入，无法直接通过输入输出重定向来实现，这个时候使用expect命令就是切合了。

expect本身就像一个伪shell一样，所以能够提供交互功能。

简单的脚本示范
```
# !/usr/bin/expect
spawn echo "hello world"
interact
```
输出为：
```
spawn echo "hello world"
hello world
```
spawn 一般是启动命令，后面跟着要执行的命令，spawn会启动这个命令进程，然后与其交互。第一行的显示表示启动的命令，是对应的脚本行的原样显示。

interact关键字是指：将交互还给shell。
还有另外一个关键字：exit，退出脚本执行。

expect 重要的交互命令
```
# !/usr/bin/expect
spawn passwd root # 执行更改密码的命令
expect "password" # expect 接收进程中的输出，匹配到'password'存在时，执行下一条语句
send "hello_world"# 向进程中输入数据。这里就是你的新密码了
exit
```

### 四. ps (process status)
此ps非adobe 公司出品的ps，而是linux下查看进程相关状态的利器。同时，ps命令涉及了太多的参数，也为其使用带来了疑惑。在编写一些涉及处理程序的脚本时，ps几乎是必不可少的了。

ps参数基本格式有三种：

```
1：unix风格，参数前带一个'-'，i.e：ps -a
2: BSD风格，参数不带'-'，i.e: ps a
3: GNU长参数风格，参数带俩个'-',i.e: ps --deselect
```
前两者风格的参数是可以无空格地混合在一块的，比如说常见的ps -aux ,是混合了两种风格的参数。混合参数之间默认是取与的关系

介绍几个比较重要的参数
```
-A unix风格，选择全部的进程
-a 除去sid与pid相等的和与终端无关的全部进程
-U 或者是 U 可以指定与该用户相关的进程
 x 与终端有关的进程
-o 指定输出格式
```

参数有很多，配合输出格式就会大有用途。

输出格式有pid,state,tname,time,command等等，只需在-O 或者 o参数下指定就行了；
```
ps -aux o command,state,time   ## 可以输出启动命令，状态，时间片信息
```

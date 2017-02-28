---
title: strace 和基友 ltrace
date: 2016-04-26
---
正如其字面意思一样，trace即追踪。在操作系统里面，进程是一个相对独立的单位，在不知其源代码的情形下，要想知道一个进程在做些什么，这就是strace和ltrace所干的活了。

因为其重要性和其能发挥的力量，我在这里就直接翻译linux中的man文档，以促使其完整和准确。

##### strace： 追踪系统调用和信号量

1.解释
在一般的情形下，strace运行指定的命令，直至其结束。一个进程所接收的信号和进行的系统调用将会被strace拦截并记录。每个系统调用的名字，其参数，连同其返回值都将在标准错误中打印出来，如果使用了-o参数的时候，就会写入到指定的文件。

strace是一个可以用于诊断，学习的调试工具。借由其不需重新编译去追踪程序的特征，系统管理员，调试工程师，debugger在不知道源代码的情形下，也可以发挥它在解决问题上的无限能力。学生们，黑客以及那些过于好奇的家伙们，可以通过追踪那些普通程序来学习很多操作系统的知识。程序员们，同样也将会发现，因为系统调用和信号量都是发生在内核和用户态的交汇处，借此可以有效地隔离bug，做完整性的检查，以及发现死锁的发生。

追踪打印的每一行都包含了系统调用的名字，以及被括号包围的参数和其返回值。使用cat /dev/null 作为一个例子：

```
open("/dev/null",O_RDONLY) = 3
```
错误的发生（一般返回值为-1）会有错误符号和错误信息追加在后面。

```
open("/foo/bar", O_RDONLY) = -1 ENOENT (No such file or directory)
```
信号打印出来的是其信号符号和信号文字。sleep 666的一个启动和终止的例子：

```
sigsuspend([] <unfinished ...>
--- SIGINT (Interrupt) ---
+++ killed by SIGINT +++
```

如果一个系统调用发生时，另外的线程或者进程调用了一个系统调用，此时，strace会保存这些事件的顺序并且标记该调用为unfinished，当调用有返回时，再标记为resumed。

```
       [pid 28772] select(4, [3], NULL, NULL, NULL <unfinished ...>
       [pid 28779] clock_gettime(CLOCK_REALTIME, {1130322148, 939977000}) = 0
       [pid 28772] <... select resumed> )      = 1 (in [3])
```
接收到的信号中断一个（可重启的）系统调用与内核中断系统调用并在信号处理完毕之后立即重新执行是不同的

```
       read(0, 0x7ffff72cf5cf, 1)              = ? ERESTARTSYS (To be restarted)
       --- SIGALRM (Alarm clock) @ 0 (0) ---
       rt_sigreturn(0xe)                       = 0
       read(0, ""..., 1)                       = 0
```

参数以人性化的方式打印出来，一个shell执行>>xyzzy 输出重定向的例子：

```
open("xyzzy", O_WRONLY|O_APPEND|O_CREAT, 0666) = 3
```
在这里，open函数的3个参数之后的标志参数变成了按位或的形式且打印出了惯用的八进制的形式。在原始形式和惯用形式不同于ANSI或者POSIX标准时，更加偏好使用标准形式。在一些案例中，strace表现出了比源代码更易读的特性。

结构体指针会被解释，其成员会被以合适的方式打印出来。全部都是以类似于c语言的形式。例如，ls -l /dev/null 的实际执行被追踪如下：
```
lstat("/dev/null", {st_mode=S_IFCHR|0666, st_rdev=makedev(1, 3), ...}) = 0
```

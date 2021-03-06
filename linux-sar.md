---
title: Linux之sar
date: 2016-10-11
categories:
- 技术
---
一个好的linuxer想要熟悉自己正在玩弄的系统的状态，可不能只看360的悬浮球。linux下查看系统状态的命令多如牛毛！像top，lsof，fdisk，iostat，du，甚至乎cat一下/proc目录下文件，读读这些晦涩难懂的数据也是可以一窥系统全貌的。然而，居然有杀手般的sar命令在此，我们为何不来一发呢！

## sar
超长的用法如下：
```
sar  [ -A ] [ -b ] [ -B ] [ -c ] [ -d ] [ -i interval ] [ -p ] [ -q ] [ -r ] [ -R ] [ -t ] [ -u ] [ -v ] [ -V ] [ -w ] [ -W ] [ -y ] [ -n { DEV | EDEV | NFS | NFSD
       | SOCK | ALL } ] [ -x { pid | SELF | ALL } ] [ -X { pid | SELF | ALL } ] [ -I { irq | SUM | ALL | XALL } ] [ -P { cpu | ALL } ] [ -o [ filename ] | -f [ filename ]] [ -s [ hh:mm:ss ] ] [ -e [ hh:mm:ss ] ] [ interval [ count ] ]
```
用法乍看是有点繁杂的，其实，众多的用法是因为其可选项众多而已，可选项众多意味着其功能广泛。且其用法也是相当简洁的，众多参数对应的是：
```
-b:io状态
-B:页交换的状态
-c:任务创建的状态
-d:磁盘状态
-n:网络状态
-u:cpu状态(无任何时，默认是查看cpu状态)
-P:指定cpu的状态
-q:系统负载，任务队列的状态
-r:内存和交互分区状态
-R:内存状态
```
其中有[ interval [ count ]参数，当指定这俩个参数事，会以interval秒去查看count次当前的状态。不指定该参数则查看从当天开始到现在时间的历史数据。

### sar之查看网速
```
# sar -n DEV
12:00:01 AM     IFACE   rxpck/s   txpck/s   rxbyt/s   txbyt/s   rxcmp/s   txcmp/s  rxmcst/s
12:10:01 AM        lo     11.10     11.10    738.91    738.91      0.00      0.00      0.00
12:10:01 AM      eth0    167.08    170.74  29037.82  14849.14      0.00      0.00      0.00
12:20:02 AM        lo     11.03     11.03    734.42    734.42      0.00      0.00      0.00
12:20:02 AM      eth0    556.00    559.64  84234.67  46365.23      0.00      0.00      0.00
12:30:02 AM        lo     11.06     11.06    735.89    735.89      0.00      0.00      0.00
12:30:02 AM      eth0   3007.27   3010.13 433371.24 244914.96      0.00      0.00      0.00
12:40:02 AM        lo     11.05     11.05    735.69    735.69      0.00      0.00      0.00
12:40:02 AM      eth0    929.72    933.30 137352.27  76655.95      0.00      0.00      0.00
12:50:01 AM        lo     11.05     11.05    735.55    735.55      0.00      0.00      0.00
12:50:01 AM      eth0   3030.71   3034.50 434493.03 246797.75      0.00      0.00      0.00
```
选项-n需要指定额外的关键字：{ DEV | EDEV | NFS | NFSD | SOCK | ALL }，DEV指定查看网络设备上的状态。
```
# sar -n DEV 1 2
11:26:11 AM     IFACE   rxpck/s   txpck/s   rxbyt/s   txbyt/s   rxcmp/s   txcmp/s  rxmcst/s
11:26:12 AM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11:26:12 AM      eth0   1598.00   1602.00 226739.00 129800.00      0.00      0.00      0.00

11:26:12 AM     IFACE   rxpck/s   txpck/s   rxbyt/s   txbyt/s   rxcmp/s   txcmp/s  rxmcst/s
11:26:13 AM        lo     27.27     27.27   1857.58   1857.58      0.00      0.00      0.00
11:26:13 AM      eth0    100.00    135.35  30420.20  15892.93      0.00      0.00      0.00
```
以一秒的速度查看俩次网络状态，IFACE是指网络设备，其中rxbyt/s（recieved）以及txbyt/s（transimitted）是指网络流量。

---
title: bin-log备份脚本
date: 2018-01-29
categories: 
- 技术
- mysql
---
#bin-log备份脚本


## mysql bin log
mysql 有多种日志类型，在一个长期running的项目中，值得研究的两个日志是：慢日志（slow log）,二进制日志 (bin log)。前者相对较为简单，连接到mysql server或者在配置中配置好时间，并且开启，可以观察到运行较慢的sql，后者往往用于数据同步。二进制日志记录了每一个使数据变跟的sql，通过这一功能，我们可以恢复确定时间点的数据，本质上就是日志追踪了每一个使数据变更的操作，然后再重复所有变更操作。


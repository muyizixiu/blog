---
title: bin-log备份脚本
date: 2018-01-29
categories: 
- 技术
- mysql
---
# binlog备份脚本
出于想日常备份binlog的缘故，编写了一个可以定期连接远程mysql server并下载binlog日志的脚本。

## mysql bin log
mysql 有多种日志类型，在一个长期running的项目中，值得研究的两个日志是：慢日志（slow log）,二进制日志 (bin log)。前者相对较为简单，连接到mysql server或者在配置中配置好时间，并且开启，可以观察到运行较慢的sql，后者往往用于数据同步。binlog是一种mysql数据库记录数据变更操作的二进制日志，mysql server会记录一定量的日志，并利用二进制日志来实现数据主从数据同步。远程连接mysql server可以下载一定时间内的binlog日志，更长时间范围的需要直接取对应的存储位置下载。二进制日志记录了每一个使数据变跟的sql，通过这一功能，我们可以恢复确定时间点的数据，本质上就是日志追踪了每一个使数据变更的操作，然后再重复所有变更操作。

## 脚本内容

```
#!/bin/sh
#备份binlog日志
#利用sql 'show master logs' 查看binlog日志
#命令行输入密码时，如果存在特殊字符，除了转义外，还可以使用单引号，单引号里的内容不会被shell解释为特殊含义
mysql_binlog_filename=$(mysql -u username -p'password' -h host -e "show master logs"|grep "mysql-bin"|awk '{print $1}')
for file in $mysql_binlog_filename
do
	dirname=$(date "+%y-%m-%d")
	if ! test -d $dirname
	then
		mkdir $dirname
	fi
	#远程读取binlog日志
	`mysqlbinlog -u username -p'password' -h host --read-from-remote-server $file --result-file=$dirname/$file  -v --base64-output=decode-rows`
done
```

## 解析
1. mysqlbinlog 连接到远程服务器，可以下载指定的binlog日志
2. -v 参数可以导出更多详细信息
3. --base64-output指定解析binlog，decode-rows可以解析为带注释的sql，另外的可选值包括已经放弃的always，以及不解析的never。
4. 下载日志时,使用低版本的mysqlbinlog和高版本的mysql server会导致校验失败: Slave can not handle replication events with the checksum that master is configured to log;


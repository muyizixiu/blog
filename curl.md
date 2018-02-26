---
title: curl追踪http请求跳转
date: 2018-02-26
categories:
- 技术
---

# curl追踪http请求跳转
在网路中，http(s)跳转是常见的一种行为，例如：301永久重定向，302临时重定向。而curl是一种命令行发起请求的工具。在工作中，我们项目遇到一个难以理解的跳转行为，最终利用curl和nginx配置文件，理解了该跳转行为。

## 问题描述
项目中存在一个老的资源地址www.old.com，仅支持http访问。新的需求要求将一个新的地址www.new.com，无论http/https都要跳转至www.old.com。需求相对较为简单，团队内部开发人员采用的方式为：

1. 将https://www.new.com在nginx中rewrite至http://www.new.com.
2. 在http://www.new.com的虚拟server中使用nginx的proxy_pass代理至http://www.old.com

理论上这样便可以实现当前的要求. 测试时发现，https://www.new.com跳转至http://www.old.com正常，但http://www.new.com跳入了一个错误的地址, https://www.wrong.com.
实际导致该问题出现有两部分原因：

1. 开发人员实际设置https://www.new.com跳转至http://new.com. 而new.com的nginx配置与www.new.com一致，共享同一个server块。
2. 目的地地址www.old.com识别跳转地址，如果是特定的地址（本项目中为www.new.com）则跳转入www.wrong.com。

问题不是很复杂，但是每个原因都很偶然，且难以直接分辨, 表现形式也诡异。https://www.new.com跳转正常，http://www.new.com则失败。

## debug
处理该bug时，首要想法是弄清楚每一个跳转行为。三个方案可以实现该目标：
1. 浏览器开发者工具中查看网络请求
2. nginx可以设置rewrite_log，可以在error_log的info信息中查看跳转行为
3. curl请求模拟
因为跳转时301永久重定向，浏览器刷新查看网络请求就显得比较麻烦；而nginx rewrite_log 则需要在线上服务器中修改配置，请求量过多会导致日志追查困难。curl是个命令行工具，也无需考虑缓存问题。

```
curl -I http://www.new.com
# -I 选项只查看http head信息
```
仅查看http head信息发现，跳转至了一个移动端的地址m.new.com，而非原先的错误地址。随后查看nginx配置得知，移动端的跳转利用了user agent，根据user agent判断是否来自移动端，如果是，则进行移动端跳转。curl请求默认的user agent则为curl，在移动端跳转规则之内。

```
curl -A PC -I http://www.new.com
# -A 选项指定user agent
```
替换user agent发现, http://www.new.com跳转至错误地址https://www.wrong.com。问题复现！

```
curl -A PC -I -L http://www.new.com
# -L 选项使curl跟随每次跳转请求，这样我们可以观察到所有rewrite行为了。
```
排除user agent问题后，发现跳转行为一如nginx 配置，http://www.new.com直接跳转至http://wrong.com, 再接着升级为https://wrong.com。

```
curl -A PC -I -L https://www.new.com
# https 的请求追随
```
而https的请求跳转行为为：https://www.new.com, https://new.com, http://www.old.com。跳转成功。对比两个跳转行为，问题1则被发现了。

```
curl -v -A PC -I -L http://www.new.com
# -v 选项指定查看请求详细信息
```
在接下来的详细信息中，我们能观察到每次跳转请求的服务器地址。因为http://www.new.com并非跳转至http://www.old.com，而是代理（即proxy_pass）。无法观察挑任何跳转信息，而在服务器响应的response中，我们观察到，http header中有字段X-Powerd-By: PHP/5.60. 判断出该跳转乃是目标地址的php脚本行为。


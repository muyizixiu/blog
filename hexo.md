---
title: hexo博客指南
date: 2017-02-28
categories:
- 技术
---
最近因为着迷于银河系漫游指南，所以大言不惭地把标题标为指南。其实这里只是我利用hexo搭建博客的小小经验而已,我原来有在daocloud上搭建一个免费的博客，用来尝尝docker的鲜。现在博客资源过期，就鼓捣下把博客迁移到自己的服务器上来了！

## hexo
hexo的[文档](https://hexo.io)看上去很复杂，其实用起来是相当简单的，不像我原来使用的ghost要依赖数据库，整个目录也是干净透明的。

### 安装
```
npm install -g hexo-cli
```
有nodejs环境的话，一句话搞定。-g选项指明我们安装到全局路径中，默认会在环境变量中的path下安装一个可执行文件。
如果需要安装nodejs的话，可以参考各个平台的安装方式，我在centos下执行的是：
```
#yum -y install nodejs
```
也是相当简单的！

### 使用
使用的文档相对比较复杂,实际的操作其实是相当简单的,了解几个相对简单的概念就能灵活使用hexo了。
首先,大胆地建立一个工程
```
hexo init hello
```
建立以hello为根目录的工程之后，了解如下几个文件夹和文件的概念就可以顺利使用了！

1. 文章目录
在hello/source/_posts/下，就是你的MarkDown文件了，你在这里写md的文件，hexo会替你生产出文章来的

2. 网页目录
hello/publics/就是你的网页了，hexo generate之后，这里便有所有在md文章目录下的所有对应的网页了，不依赖其余目录，这就是一个可以独立导出的网页目录。配合nignx或者其他网络server便可以实现网页浏览博客的效果了！值得一提的是，官方自己也实现了一个webserver 命令：hexo server，可以用来直接访问该网页目录。

3. 配置文件
由于你只提供了文章，想要在样式和一些指定内容上做些自己的变更，就需要在配置文件上做相应的变更了。文件的地址是：hello/_config.yml（其实还有themes下面的配置）
网站的标题，副标题，描述等等个性化的设置都在这个文件里配置，详细的文档可以参观[https://hexo.io/docs/configuration.html](https://hexo.io/docs/configuration.html)

4. 主题
博客要想华丽起来，离不开[hexo themes](https://hexo.io/themes/)，大量的主题，在GitHub都有源，下载下来，或者git clone下来，到hello/themes/下面，然后在配置文件里面指定主题即可。
在下载下来的主题文件夹里面也会有_config.yml的配置文件，实现各个主题个性化！我使用的是一款简单的，文字友好型的[主题](https://github.com/gaoryrt/hexo-theme-pln)!

5. 我的策略
hexo generate依赖工程和你的md文件，生成了静态的网页，网页目录是可以脱离于工程存在的，同时，我也将md的文件分离了出来，用git进行管理，这样，我的文章便在github上保留了[一份源](https://github.com/muyizixiu/blog)，可以脱离这个工程进行写作。同时我将工程中的文章目录(hello/source/_posts),指向我的github地址，每次新的文章被提交时，我pull下来，同时用hexo generate生成静态的网页，再将其复制到webserver所指定的地址。当然，上面的操作已经脚本化了，简单地执行脚本即可！

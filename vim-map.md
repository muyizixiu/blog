---
title: vim之键映射
date: 2017-03-18
---
前面我有两篇文章，关于vim的[入门](http://muyizixiu.cn/2015/12/08/vim/)和[autocmd](http://muyizixiu.cn/2016/10/10/vim-autocmd/)，随着对vim使用的更加深入，vim的配置便不得不提了！

## keycodes
在vim脚本中，键的映射总是伴随着一些奇怪的符号！例如\<TAB\>,\<ESC\>这些从字面上可以揣测出来的tab键和esc键，还有些难以猜测的，\<S-...\>,\<A-...\>,\<CR\>。这些事vim的keycode。
\<S-...\>和\<A-...\>是组合键，前者是指shift键组合其余的按键，后者则是Alt键的组合。例如\<S-r\>,则是指shit组合字母r键。
查看每一个详细的按键，在命令模式下，输入:h keycodes 即可

## map
键映射的关键字是各种map。
|命令    |Normal(常规模式)|Visual(可视化)|Insert(插入模式)|Command(命令行模式)|
|:-------|:--------------:|:------------:|:--------------:|:------------------:|
|:map    | 有效           |   有效       |                |                    |
|:nmap   | 仅在该模式有效 |              |                |                    |
|:vmap   |                |仅在该模式有效|                |                    |
|:map!   |                |              |                |             有效   |
|:imap   |                |              |  仅在该模式有效|                    |    
|:cmap   |                |              |                | 仅在该模式有效     |

可以看出，map加不同的前缀和后缀组成了不同的映射方式。要注意的是，这里的前缀，n,i,v,c都是对应在各自的模式下才生效。以上所有的映射都是嵌套的（nested），当把一个键或者组合键映射到另一个键或者组合键时，被映射的键或者组合键又映射到另外的键时，会向下嵌套。
例如：

```
map a b
map b c
```
就等同于将a映射至了c,如果不想发生这种副作用时，在map前加入nore。inoremap,noremap,cnoremap等映射则不发生嵌套。

## 快捷键
仅仅是将一个键映射为另外一个键是没有多大意义的，map映射可以将键映射到命令上。

```
map <S-r> call reload()
```
shit组合字母r键，则调用了reload函数，当然，实际上vim没有reload这个函数。
这就是vim快捷键的配置。

## 清除映射
mapclear可以清除所有map的映射，单个映射的清楚则是在对应的映射命令字的map前面加上字母un。

```
map a b
umap a
imap <S-c>
iunmap <S-c>
"清除所有map的映射则使用mapclear
mapclear
"清除所有imap的映射则使用imapclear
imapclear
```

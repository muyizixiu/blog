---
title: vimrc之autocmd
date: 2016-10-10
---
有句话说得好，如果EMACS是神的编辑器，那么vim就是编辑器之神了！vim和emacs极佳的扩展性，以及众多活跃的插件，是经常切换多语言编程的利器。那么，vimrc是vimer进阶不得不掌握的战场了！

vimrc是vim的配置文件， 强大如编程语言一般！

## autocmd
autocmd是用于注册vim事件的处理方法，可以用于在特定事件（读，写，新建文本等）下触发特定的动作。
autocmd的定义
```
:au[tocmd] [group] {event} {pat} [nested] {cmd}
```
关键字au[tocmd]是少不了的，{group}是组名称，用于将当前所定义的注册命令归属于一个组，缺省的话默认是当前组，当前组可以用:augroup查看。event是指事件：
```
名字                    激活条件 

        读入
|BufNewFile|            开始编辑尚不存在的文件
|BufReadPre|            开始编辑新缓冲区，读入文件前
|BufRead|               开始编辑新缓冲区，读入文件后
|BufReadPost|           开始编辑新缓冲区，读入文件后
|BufReadCmd|            开始编辑新缓冲区前 |Cmd-event|

|FileReadPre|           用 ":read" 命令读入文件前
|FileReadPost|          用 ":read" 命令读入文件后
|FileReadCmd|           用 ":read" 命令读入文件前 |Cmd-event|

|FilterReadPre|         用过滤命令读入文件前
|FilterReadPost|        用过滤命令读入文件后

|StdinReadPre|          从标准输入读入缓冲区前
|StdinReadPost|         从标准输入读入缓冲区后

        写回
|BufWrite|              开始把整个缓冲区写回到文件
|BufWritePre|           开始把整个缓冲区写回到文件
|BufWritePost|          把整个缓冲区写回到文件后
|BufWriteCmd|           把整个缓冲区写回到文件前 |Cmd-event|

|FileWritePre|          开始把缓冲区部分内容写回到文件
|FileWritePost|         把缓冲区部分内容写回到文件后
|FileWriteCmd|          把缓冲区部分内容写回到文件前 |Cmd-event|

|FileAppendPre|         开始附加到文件
|FileAppendPost|        附加到文件后
|FileAppendCmd|         附加到文件前 |Cmd-event|

|FilterWritePre|        开始为过滤命令或 diff 写到文件
|FilterWritePost|       为过滤命令或 diff 写到文件后

        缓冲区
|BufAdd|                刚把缓冲区附加到缓冲区列表后
|BufCreate|             刚把缓冲区附加到缓冲区列表后
|BufDelete|             从缓冲区列表删除缓冲区前
|BufWipeout|            从缓冲区列表完全删除缓冲区前

|BufFilePre|            改变当前缓冲区名字前
|BufFilePost|           改变当前缓冲区名字后

|BufEnter|              进入缓冲区后
|BufLeave|              转到其它缓冲区前
|BufWinEnter|           在窗口显示缓冲区前
|BufWinLeave|           从窗口删除缓冲区前

|BufUnload|             卸载缓冲区前
|BufHidden|             刚把缓冲区变为隐藏后
|BufNew|                刚建立新缓冲区后

|SwapExists|            检测到交换文件已经存在

        选项
|FileType|              设置 'filetype' 选项时
|Syntax|                设置 'syntax' 选项时
|EncodingChanged|       'encoding' 选项改变后
|TermChanged|           'term' 的值改变后

        启动和退出
|VimEnter|              完成所有的初始化步骤后
|GUIEnter|              成功启动 GUI 后
|GUIFailed|             启动 GUI 失败之后
|TermResponse|          收到 |t_RV| 的终端应答后

|QuitPre|               用 `:quit` 时，决定是否退出之前
|VimLeavePre|           退出 Vim 前，在写入 viminfo 文件之前
|VimLeave|              退出 Vim 前，在写入 viminfo 文件之后

        杂项
|FileChangedShell|      Vim 注意到文件在编辑开始后被改变
|FileChangedShellPost|  对在编辑开始后被改变的文件的处理完成后
|FileChangedRO|         对只读文件进行第一次修改前

|ShellCmdPost|          执行外壳命令后
|ShellFilterPost|       用外壳命令执行完过滤后

|FuncUndefined|         调用没有定义的用户函数
|SpellFileMissing|      使用不存在的拼写文件
|SourcePre|             执行 Vim 脚本之前
|SourceCmd|             执行 Vim 脚本之前 |Cmd-event|

|VimResized|            Vim 窗口大小改变后
|FocusGained|           Vim 得到输入焦点
|FocusLost|             Vim 失去输入焦点
|CursorHold|            用户有一段时间没有按键
|CursorHoldI|           在插入模式下，用户有一段时间没有按键
|CursorMoved|           普通模式下移动了光标
|CursorMovedI|          插入模式下移动了光标

|WinEnter|              进入其它窗口后
|WinLeave|              离开窗口前
|TabEnter|              进入其它标签页后
|TabLeave|              离开标签页前
|CmdwinEnter|           进入命令行窗口后
|CmdwinLeave|           离开命令行窗口前

|InsertEnter|           开始插入模式前
|InsertChange|          在插入或替换模式下输入 <Insert> 时
|InsertLeave|           离开插入模式时
|InsertCharPre|         插入模式输入每个字符前

|ColorScheme|           载入色彩方案后

|RemoteReply|           得到了 Vim 服务器的应答

|QuickFixCmdPre|        执行 quickfix 命令前
|QuickFixCmdPost|       执行 quickfix 命令后

|SessionLoadPost|       载入会话文件后

|MenuPopup|             刚要显示弹出菜单前
|CompleteDone|          插入模式补全结束之后

|User|                  和 ":doautocmd" 一起使用

```
引用自[VIMCDOC](http://vimcdoc.sourceforge.net/doc/autocmd.html#autocmd-events)。

模式匹配则是对当前文本类型的一种匹配，支持通配符，eg: *.php匹配php文本。
{nested}则是指明可以嵌套事件，当所处理的动作也被注册为事件触发另一动作时，默认不触发。

{cmd}就是我们在vim命令模式下面经常使用的了，复杂的动作可以使用函数来处理。


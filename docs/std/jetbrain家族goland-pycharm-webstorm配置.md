---
title: "jetbrains"
date: "2018-11-23 01:00:13"
categories:
    - go
tags:
    - go
---

**pycharm**是我用得比较多的`go ide`. 因为有时候要写`python`, 想一个ide搞定。但是最近`go1.10`出来之后，`pycharm`不支持。而**goland**终于出了一个试用版本。用了一下很好用。于是记录一下使用的一些快捷键。

更新：
这里面包括了**JetBrains**家族里面的所有IDEA配置。包括pycharm, goland, webstorm, ide等所有都可以通用。需要安装gototabs插件。

---

# pycharm 修改键盘习惯
KeyMap选择**Default copy**, 也就是利用`Default`来复制进行修改最方便：

| 组合键              | 修改内容                                                                              |
| :------------------ | :---------------                                                                      |
| ctrl+cmd+]       | Move Caret to Code Block End                                                          |
| 删除                | Move Caret to Line End                                                                |
| Commnad+Shift+F, ~~Ctrl+\, E~~           | Find in Path...                                                                       |
| Ctrl+\, S           | Find Usages(用法)                                                                     |
| Command+E              | Recent Files                                                                          |
| Ctrl+Command+F      | Toggle Full Screen mode                                                               |
| Ctrl+T, T           | Back                                                                                  |
| Ctrl+]              | Declaration                                                                           |
| Command+[           | Previous Method                                                                       |
| Command+R           | Run...                                                                                |
| Command+1           | Go to TAb #1                                                                          |
| Ctrl+, F            | Project                                                                               |
| Ctrl+, S            | Structure                                                                             |
| Ctrl+\, T           | Search Everywhere                                                                     |
| Ctrl+Command+P      | Previous Occurrence 在Find Usage(查找用法)子窗口中，上一个查找点的快捷键              |
| Ctrl+Command+N      | Next Occurrence 在Find Usage(查找使用)子窗口中，下一个查找点的快捷键                  |
| Shift+Esc           | 这个键不用改，系统默认的。隐藏Active窗口。Hide Active tool Window                     |
| Ctrl+X,3            | 这个键不用改， **大屏这个垂直分屏用得比较多，看代码爽啊**。Split Vertically 垂直分屏 |
| Ctrl+X,2            | 这个键不用改，Split Horizontally 水平分屏                                            |
| Ctrl+X,1            | 这个键也不用改，取消所有分屏，UnSplite All                                            |
| Ctrl+W, HJKL        | 这个是vim自带的快捷键，分屏后就可以在各个屏之中来回跳转了。                           |
| /\cinitchain        | \c表示set ignorecase; \C 表示smartcase的意思。                                        |
| Command+/            | Comment Line 单行注释                                                                 |
| Ctrl+Shift+/              | Block Comment 块注释                                                                  |
| Ctrl+Option+Shift+P | go fmt project 格式化整个项目的代码                                                   |
| Ctrl+Option+Shift+F | go fmt file 格式化文件                                                                |
| Command+T           | open terminal, close terminal 打开终端                                                |

# 字体
- Scheme: Atome one dark
- Font: Go Mono for Powerline ~~Source Code for Powline~~
- size:15
- Line Space:1.4
- 取消import代码折叠功能： Editor->General->Code Folding->取消Imports前面的勾即可

> 字体我一般是这几种换着用：Courier New, Monaco, Menlo, Source Code Pro

# 自动补全
Settings >Editor >General >Code Completion    设置智能提示大小写敏感为"None"

# 自动换行
Settins > Editor > General > Soft Wraps > 打开 Soft-warp files: 增加 `*.md; *.go`

# 代码检测
在ide面板上点击右下角像`老人头`似的图标设置【代码检测警告提示】等级，建议开启**Inspections**检测功能。

# 自动导入以及Tab页功能
Settings 》Editor 》General 》Auto Import            设置包自动导入和优化导入
Settings 》Editor 》General 》Editor Tabs            设置打开的Tab页多行显示及打开的Tab页上限

# import以及注释前增加空格功能
Settings 》Editor 》Code Style 》Go            右侧Other标签页，勾选Add leading space to comments

# pycharm设置python3
输入interpreter里面点右边的Project Interpreter进行设置。

# 代理
plugin > htpp... > http proxy 设置http localhost 1023

# webstorm配置
- **react Library**的支持: Languages & Frameworks >> JavaScript >> Libraries >> Download >> 输入react 找合适的下载。 另外要想支持JSX的话，不要点Library了，直接点他的父类JavaScript之后在右边选择**React JSX**即可。

# 简单字符串查询
我平时用**vim**比较多,最爱vim的快捷键了。最喜欢**/**就直接查找了。但是默认是大小写敏感的。如果想忽略的话，需要输入**\c**才能忽略。
```vim
/\cinitchain
```

# 利用**C-\ e**(Find in Path...) 查看interface

示例：
```go
interface {
  SignTx(account Account, tx *types.Transaction, chainID *big.Int) (*types.Transaction, error)
}
```

查询用：
```perl
// func (ks *KeyStore) SignTx(a accounts.Account, tx *types.Transaction, chainID *big.Int) (*types.Transaction, error) {
^func\s.*SignTx\(.*accounts.Account
```
正则使用的`java`的正则：
[regex](https://docs.oracle.com/javase/1.5.0/docs/api/java/util/regex/Pattern.html)

## 修改vmoptions

vmoptions路径：

```bash
vim ~/Library/Preferences/CLion2019.2/clion.vmoptions
vim /Applications/GoLand.app/Contents/bin/goland.vmoptions
```

修改文件：`vim /Applications/GoLand.app/Contents/bin/goland.vmoptions`

```yaml
-Xms3g
-Xmx3g
-XX:ReservedCodeCacheSize=2g
-XX:+UseCompressedOops
-Dfile.encoding=UTF-8
-XX:+UseConcMarkSweepGC
-XX:SoftRefLRUPolicyMSPerMB=50
-ea
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-Xverify:none

-XX:ErrorFile=$USER_HOME/java_error_in_goland_%p.log
-XX:HeapDumpPath=$USER_HOME/java_error_in_goland.hprof
```

# 关于主题

在GoLand使用了很长一段时间都没有修改过主题。后来发现atom的真的很漂亮，于是又开始折腾主题搭配了。
- 主题在**plugin**里面搜`Material Theme UI`安装即可。非常好看。再在plugin里面搜`Rainglow Color Schemes`安装。
- 最后在**Color Scheme**里面选择**Atom One Dark**即可。完美

# 启动不了
如果手贱乱破解就会导致像我这样启动不了，那时候只能重新来过了,删除下面配置文件就可以，另外所有JetBrains家族的软件都适合这样：

```bash
rm -rf ~/Library/Preferences/GoLand*
rm -rf ~/Library/Application\ Support/GoLand*
rm -rf ~/Library/Caches/GoLand*
rm -rf ~/Library/Logs/GoLand*
```



## 关于goland modules路径问题

由于在golang1.12之后启用了modules功能，所以在配置的时候一定要注意modules的路径就是gopath的路径，它会自动去找pkg/mod目录下面的模块目录。



- [j](http://www.ifdll.com/)
- [h](http://mmm.lu)
- [d](http://www.iFdll.com)
- 推荐一个吧 http://lookdiv.com 钥匙是[lookdiv.com](http://lookdiv.com)

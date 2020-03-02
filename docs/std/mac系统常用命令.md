---
title: "mac系统常用命令"
slug: "mac-os-cmds"
date: "2018-11-23 01:00:21"
categories:
    - mac
tags:
    - mac
    - cmds
---

最近腰疼，真的很不舒服。才发现健康的重要性。还有好多事情好多代码等我去撸，可不能这样倒下了。

## 复制
**pbcopy**复制 **pbpaste**粘贴：

```bash
cat t.txt | pbcopy
pbcopy < ~/.ssh/id_rsa.pub
```

## 排列文件夹

最喜欢的是`ctrl+command+2`:
```bash
ctrl+command+1....9
```

## mac os 常用图标
- Command ⌘
- Shift ⇧
- Option ⌥
- Control ⌃
- Caps Lock ⇪
- Fn

## iterm
- Command+点击文件夹：在 finder 中打开该文件夹； 
- Command+Option 可以以矩形选中，类似于vim中的ctrl v操作。

## 路由器
http://192.168.252.16:8443

## firefox强制刷新
**shift+command+r**

## gofmt

```bash
# find . -name "*.go" -not -path "./vendor/*" -not -path ".git/*" | xargs gofmt -s -d
find . -name "*.go" -not -path "./vendor/*" -not -path ".git/*" | xargs gofmt -w
```
- **gofmt -w main.go** 会格式化该源文件的代码然后将格式化后的代码覆盖原始内容(如果不加参数 -w 则只会打印格式化后的结果而不重写文件) `write`
- **gofmt -w *.go** 会格式化并重写所有 Go 源文件
- **gofmt map1** 会格式化并重写 map1 目录及其子目录下的所有 Go 源文件。

```bash
gofmt -h
usage: gofmt [flags] [path ...]
  -cpuprofile string
    	write cpu profile to this file
  -d	display diffs instead of rewriting files
  -e	report all errors (not just the first 10 on different lines)
  -l	list files whose formatting differs from gofmt s
  -r string
    	rewrite rule (e.g., 'a[b:len(a)] -> a[b:]')
  -s	simplify code
  -w	write result to (source) file instead of stdout
```

## 查看端口占用

**linux**
```bash
netstat -anp | grep 3306
```

**mac**
```bash
lsof -i:80
sudo lsof -i -P | grep -i "listen"
sudo lsof -iTCP -sTCP:LISTEN | grep 11313
```

- `grep -i` 表示--ignore-case

## 多窗口切换
隐藏应用程序（Cmd+H）=将当前 App「最小化到本身图标上」（Dock 里此 App 的图标会变成半透明） ，而「单独缩小到 Dock 右端」只是最小化窗口（Cmd+M）



## 参考文献

- [像高手一样写go](http://colobu.com/2017/06/27/Lint-your-golang-code-like-a-mad-man/)
- [gofmt](http://wiki.jikexueyuan.com/project/the-way-to-go/03.5.html)

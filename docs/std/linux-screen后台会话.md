---
title: "linux-screen后台会话"
slug: "linux-screen"
date: "2018-11-23 01:00:30"
categories:
    - linux
tags:
    - linux
---

# screen
常常需要在后台临时运行一些程序，这时候用screen再方便不过了。

```bash
screen -S rand ./randomtxs ##screen -S 会话命名 command，然后Ctrl+a+d放到后台
screen -ls
```

## 参考文献

- [screen](http://feilong.tech/?p=121)

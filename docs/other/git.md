# git基本使用

一些基本的概念：

- 直接记录所有快照，而非差异比较
- 近乎所有操作都在本地执行
- 时刻保持数据完整性。
- 文件的三种状态
  + 已修改(modified)
  + 已暂存(staged)
  + 已提交(committed)

Git文件三个空间
- 工作区
- 暂存区
- 对象库(git的版本控制系统中)

版本管理
- git add(放入暂存中)
- git commit(放入版本控制系统中)
- git rm(删除)

查看信息
- git help
- git log
- git diff
- git status

git配置,优先级是越是下面越高
对于`user.name`,`user.email`有三个地方可以设置
- /etc/gitconfig(几乎不会使用) `git config --system`
- ~/.gitconfig(很常用) `git config --global`
- 针对于特定项目的，.git/config文件中 `git config --local`

查看本地文件中的配置: 
```git
git config --local -l
git config --local user.name 'liwei'
git config --local user.email 'liwei@gmail.com'
```

**删除相关**:

**git rm**
- 删除了一个文件
- 将被删除的文件自动纳入到暂存区(stage)

若想恢复被删除的文件，则需要进行两个动作：

1. git reset HEAD t2.md # 将待删除的文件从暂存区恢复到工作区中
2. git checkout -- t2.md # 将工作区中的修改丢弃

**rm**:

1. `rm t2.md`只删除了文件，并未曾纳入到暂存区
2. 需要手动`git add t2`将其纳入到暂存区
3. 如果想恢复，用上面的命令: `git reset HEAD t2.md && git checkout -- t2.md`

**commit修复**
如果`git commit -m "111111"` 这个消息你发现错了，需要重新生成一个消息进行替换，可以使用
`git commit --amend -m "222222"` 这样是一次提交。并不会在log里面产生两条提交信息

**git log查看提交历史**:
- git log
- git log --graph
- git log -3(-n) 仅显示最近的n次更新
- git log --pretty=oneline 显示在一行
- git log --pretty=format:"%h-%an,%ar:%s"
- git log --stat 仅显示简要的增发行数统计
- git log --graph --abbrev-commit
- git log --graph --abbrev-commit --pretty=oneline


场景1：修改之后，还未曾提交到stag之中，就取消修改：
```bash
git restore readme.md
# 如果已经暂存了想取消
git restore --staged readme.md
```

场景2：提交到暂存区之后，撤销操作
```bash
git add readme.md
# 并不会真正删除文件，只是从暂存区中删除而已
git rm --cached readme.md
# rm 'readme.md'

# 如果不撤销,提交到版本控制系统
git commit

# 查看提交信息
git log
```

场景3：配置git的配置信息
```bash
git config --local --list
git config --global --list
git config --system --list
```

## .gitignore

- vi .gitignore 将文件及目录添加到这个内容中
- git add .gitignore
- git commit -m "add .gitignore"


**语法规范**支持正则表达式以及通配符, 主要是为了增加忽略的文件及文件夹的：
- `空行`或是以`#开头`的行即注释行将被忽略。
- 可以在前面添加正斜杠`/`来避免递归。
- 可以在后面添加正斜杠`/`来忽略文件夹，比如`build/`即忽略`build文件夹`。
- 可以使用`!`来否定忽略,即比如在前面用了`*.apk`,然后使用`!a.apk`，则这个a.apk不会被忽略。
- `*`用来匹配零个或多个字符，比如`*.[oa]`忽略所有以".o"或者".a"结尾的文件。`*~`忽略所有以`~`结尾的文件。这种文件通常被许多编辑器标记为临时文件；`[]`用来匹配括号内的任一字符，如`[abc]`，也可以在括号内加连接符，如`[0-9]`匹配`0至9的数`；`?`用来匹配单个字符。
- doc/*.txt 会忽略doc/notes.txt但是不会忽略doc/server/arch.txt, 一个星只能忽略一层的，要想忽略所有层的需要用两个星。`doc/**/*.txt`

```git
# 忽略 .a 文件
*.a
# 但否定忽略 lib.a, 尽管已经在前面忽略了 .a 文件
!lib.a
# 仅在当前目录下忽略 TODO 文件， 但不包括子目录下的 subdir/TODO
/TODO
# 忽略 build/ 文件夹下的所有文件
build/
# 忽略 doc/notes.txt, 不包括 doc/server/arch.txt
doc/*.txt
# 忽略所有的 .pdf 文件 在 doc/ directory 下的
doc/**/*.pdf
```

## 分支

常用命令：
- git branch 查看本地分支
- git branch -r 查看远程分支
- git branch [name] 创建本地分支，注意新分支创建后不会自动切换为当前分支
- git checkout [name] 切换分支 
- git checkout - 切换到上一个分支
- git branch -m master master2 改名字给分支
- git checkout -b [name] 创建新分支并立即切换到新分支上
- git branch -d [name] 删除分支，`-d`选项只能删除已经参与合并的分支，对于未合并的分支是无法删除的，如果想强制删除需要用`-D`
- git merge [name] 合并分支，将名称为`[name]`的分支的内容合并到当前分支上。这个顺序是很重要的。
    + Fast-forward: 简称ff 快进
    + 如果可能，合并分支时Git会使用fast-forward模式
    + 在这种模式下，删除分支时会丢掉分支信息
    + 合并时加上--no-ff参数会禁用fast-forward,这样会多出一个commit id
- git branch -v 当前分支提交的最新的一条历史记录
- git push origin [name] 创建远程分支(本地分支push到远程)
- git push origin :heads/[name] 删除远程分支

我从master分支创建了一个issue5560分支，做了一些修改后，使用git push origin master提交，但是显示的结果却是'Everything up-to-date'，发生问题的原因是git push origin master 在没有track远程分支的本地分支中默认提交的master分支，因为master分支默认指向了origin master 分支，这里要使用git push origin issue5560：master 就可以把issue5560推送到远程的master分支了。

如果想把本地的某个分支test提交到远程仓库，并作为远程仓库的master分支，或者作为另外一个名叫test的分支，那么可以这么做。

- git push origin test:master // 提交本地test分支作为远程的master分支  [这句命令会删掉远端的master分支。。。]
- git push origin test:test // 提交本地test分支作为远程的test分支
如果想删除远程的分支呢？类似于上面，如果:左边的分支为空，那么将删除:右边的远程的分支。
- git push origin :test // 刚提交到远程的test将被删除，但是本地还会保存的，不用担心

## 回退

- git reset --hard HEAD^
- git reset --hard HEAD^^
- git reset --hard HEAD~1
- git reset --hard commit-id
- git reflog # 查看操作日志，来回退到最新的版本

## checkout
checkout作用：discard掉相对于暂存区中最后一个添加的文件内容所做的变更。
git reset HEAD test.txt 
作用：将之前添加到暂存区(stage,index)的内容从暂存区移除到工作区中。
detached 游离分支的意思，最好的新建一个分支进行保存

## stash 保存工作现场
如果你正在你的dev分支上面做开发，突然有个bug提交过来，你需要切换到另外一个master分支上做修改。
此时你当前dev分支上面的修改还不应该commit提交，而需要暂时隐藏存储起来stash，事后再恢复。所以你需要用到stash命令。

- git stash save '' # 隐藏存储
- git stash save 'hello basic'
- git stash list # 查看记录
- git stash pop 将之前的恢复，并且将stash里面的删除
- git stash apply 将之前的恢复，但是并不删除之前保存的状态
- git stash apply stash@{0} 恢复但是不删除，需要用drop手动删除
- git stash drop stash@{0}

## 什么是快照

git中的快照，就是一个备份，但这个备份不是像我们粘贴复制那么简单，git会处理，压缩，你可以使用这个快照恢复原来的状态。git会根据当前的内容生成一个校验和，是以此校验和为索引。每次提交，检测到校验和变化，就会生成一个新的快照，未更改的文件，则会链接到上一次的快照。这样就形成了一条链（这里先讨论没有其他分支的情况），git有一个HEAD指针，这个指针可以移动，这个指针移动到哪个快照，你就可以查看该快照也就是当时的状态。


## tag

一般代码到了一个里程碑的时候，需要打一个tag记录一下。

tag与branch的区别：

tag就像是一个里程碑一个标志一个点，branch是一个新的征程一条线；
tag是静态的，branch要向前走；
稳定版本备份用tag，新功能多人开发用branch（开发完成后merge到master）
tag就是给commit的hash校验和取的一个名字，比较直观，方便记忆和使用，
和branch不是一个维度，点与线的区别
branch是一个分支
tag是分支上的一个里程碑，一个点

- git tag 查看标签
- git tag [name] 创建标签
- git tag -d [name] 删除标签
- git tag -r 查看远程标签
- git push origin --tags 推送所有tags到远程去
- git tag -l "v2*" 查询标签
- git push origin [name] 创建远程标签(本地标签push到远程)
- git push origin :refs/tags/[name] 删除远程标签


## blame
git blame [文件名] 查看文件的修改记录

## diff
先明白linux命令的diff每一个含义，才能明白`git diff`的结果含义。

- git diff: 比较的是暂存区与工作区文件之间的差别。
- git diff [commit_id] 比较commit与工作区的区别。
- git diff HEAD 比较当前最新的版本与当前工作区的区别。
- git diff --cached commit_id 比较commit上面与暂存区的区别。


## 远程remote

将本地的git仓库推到远程上去。

- 第一步需要在gitlib上面创建自己的仓库。
- `git remote add origin https://github.com/emacsvi/lw-lotus` # remote是远程的意思，origin 是用origin 来代码后面的地址。
- `git push -u origin master` 将本地的master推送到远程origin上面。-u表示把它们进行关联，下次直接推送即可。下次直接`git push`即可。

- 注意：因为是https所以每次要让你输入用户名和密码，如果将ssh的公钥放上去，并且用ssh来做提交，即不用再提示你输入了。


如果之前提交的帐户信息有误，希望修改可以使用--reset-author
```git
git log
git config --local user.name "xxx"
git config --local user.email "xxx"
git commit --amend --reset-author # 修改提交的帐户信息
git config --unset  user.mail # 删除一个key
git commit -am "info" # -am 是 add+commit的组合 
```

## 远程操作

由于本地仓库可以与远程仓库进行一对多的关联，你可以推送到多个远程仓库中。所以查看远程仓库的命令: `git remote show`

- git remote show
- git remote show origin # 列出与origin关联的远程仓库的详细信息

开发模型：
- Gitflow
- 基于git分支的开发模型：
  + delelop分支：频繁变化的一个分支 
  + test分支： 供测试人员测试用的，不是很频繁
  + master分支： 生产发布分支，变化非常不频繁的一个分支。
  + bugfix(hotfix)分支： 生产系统中出现了紧急Bug,用于紧急修复的分支。

## 协同工作

查看本地及远程分支：
- git branch -a
- git branch -av # 显示详细信息

## 关于Git分支最佳实践

## 关于Git分支最佳实践

1. 通常来说，Git分支会有如下几种：
  - master分支
  - test分支
  - develop分支
  - hotfix分支






# go module常用命令

**初始化模块**:

```bash
# 初始化模块
go mod init github.com/emacsvi/lw-lotus
```

**发布一个版本**:
```bash
# 将模块推送到github端
git tag v1.0.0
git push --tags
```

此时将会在`Github`的仓库上创建名为`v1.0.0`的标签。推荐的做法是**创建新的代码分支**，这样可以直接在分支上修改`v1.0.0`的问题，而不影响主分支的开发进度。

```bash
git checkout -b v1
git push -u origin v1
```

**发布修复版本**:
假设lw-lotus v1.0.0需要进行问题修复：
```go
// Hi returns a friendly greeting
func Hi(name string) string {
-   return fmt.Sprintf("Hi, %s", name)
+   return fmt.Sprintf("Hi, %s!", name)
}
```
我们在v1分支中进行此修复：
```bash
$ git commit -m "Emphasize our friendliness" testmod.go
$ git tag v1.0.1
$ git push --tags origin v1
```


# 参考

- [go mod](https://zhuanlan.zhihu.com/p/105556877)

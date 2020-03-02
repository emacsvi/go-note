# 自定义promu tool使用

promu是一个很好的交叉编译器。他利用了跨平台的特点来编译，真的是很爽。我总算是可以在mac下面编译任意平台的二进制代码了。他的想法很简单，利用了docker来做编译。但是我不需要这些强大的功能。

平时用得比较多的是用mac+linux两个版本。所以将源码修改一下即可。

最终用这条命令即可编译linux的版本了。
```bash
promu build -l
```

**步骤**：

```bash
git clone https://github.com/prometheus/promu.git
go build
cp promu $GOBIN/
```

修改一个文件`cmd/build.go`即可：
```go
// 增加一个flag
var (
	linuxFlag = buildcmd.Flag("linux", "build linux binary").Default("false").Short('l').Bool()
  )


func buildBinary(ext string, prefix string, ldflags string, binary Binary) {
	info("Building binary: " + binary.Name)
	binaryName := fmt.Sprintf("%s%s", binary.Name, ext)

  // 增加只对linux的编译的功能
	if *linuxFlag {
		_ = os.Setenv("GOOS", "linux")
		_ = os.Setenv("GOARCH", "amd64")

		defer os.Unsetenv("GOOS")
		defer os.Unsetenv("GOARCH")

		binaryName = binaryName+"-unix"
	}

  //.....
}
```

在个人项目中使用也比较方便。需要如下文件

- VERSION
- .promu.yml

在**VERSION**文件里面就是你的版本号
```text
2.16.0
```

在`.promy.yml`文件里面则是**promu**读取时需要编译的环境
```yaml
go:
    # Whenever the Go version is updated here,
    # .circle/config.yml should also be updated.
    version: 1.13
repository:
    # path: github.com/prometheus/prometheus
    path: .
build:
    binaries:
        - name: dchain
          path: .
        - name: promtool
          path: ./cmd/promtool
        - name: tsdb
          path: ./tsdb/cmd/tsdb
    # flags: -mod=vendor -a -tags netgo,builtinassets
    flags: .
    ldflags: |
        -X github.com/prometheus/common/version.Version={{.Version}}
        -X github.com/prometheus/common/version.Revision={{.Revision}}
        -X github.com/prometheus/common/version.Branch={{.Branch}}
        -X github.com/prometheus/common/version.BuildUser={{user}}@{{host}}
        -X github.com/prometheus/common/version.BuildDate={{date "20060102-15:04:05"}}
tarball:
    files:
        - consoles
        - console_libraries
        - documentation/examples/prometheus.yml
        - LICENSE
        - NOTICE
        - npm_licenses.tar.bz2
crossbuild:
    platforms:
        - linux/amd64
        - linux/386
        - darwin/amd64
        - darwin/386
        - windows/amd64
        - windows/386
        - freebsd/amd64
        - freebsd/386
        - openbsd/amd64
        - openbsd/386
        - netbsd/amd64
        - netbsd/386
        - dragonfly/amd64
        - linux/arm
        - linux/arm64
        - freebsd/arm
        - openbsd/arm
        - linux/mips64
        - linux/mips64le
        - netbsd/arm
        - linux/ppc64
        - linux/ppc64le
        - linux/s390x

```

## 运行命令

**promu build**

 编译项目

**promu build && promu tarball** 

将项目编译好后，文件里面的tarball依赖文件打包。
```bash
$ tar -zxvf lw-lotus-1.0.1.darwin-amd64.tar.gz
x lw-lotus-1.0.1.darwin-amd64/
x lw-lotus-1.0.1.darwin-amd64/dchain
x lw-lotus-1.0.1.darwin-amd64/chain.yml
```


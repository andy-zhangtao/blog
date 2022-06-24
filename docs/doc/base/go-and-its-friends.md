# Go 和他的小伙伴们

## 工具总览

> 先看最后的结论吧

| 名称        | 描述                     | 备注                           |
| ----------- | ------------------------ | ------------------------------ |
| go get      | 下载/安装                | GOPATH 模式<br>Module 模式     |
| go install  | 下载/编译/安装           |                                |
| go clean    | 清理缓存                 | build \ test \ benchmark       |
| go env      | 查看 golang 相关环境变量 |                                |
| go fmt      | 代码优化                 |                                |
| go generate | 自动生成代码             | 需要确保操作幂等性，不建议使用 |
| go run      | 构建并运行               |                                |
| go mod      | 依赖管理                 |                                |
| go doc      | 文档维护                 |                                |
| go test     | 运行测试案例和性能测试   |                                |
| go vet      | 代码静态检查             |                                |

**以下按照一个典型的开发生命周期：安装 --> 依赖 --> 开发 --> 调试 --> 测试 --> 构建 --> 文档化 这样的顺序展开叙述**

## 安装

### 源码安装

下载 golang 源码，交叉编译标准库和工具。 高阶操作，主要用于 sdk 裁剪，或者用于特殊指令集(比如龙芯 CPU)时操作的方式。

### 二进制安装

下载对应操作系统的二进制安装包，注意环境变量的设置问题：

1. PATH ,默认安装在 `/usr/local/go` (windows 除外)，所以需要在 PATH 中追加 `/usr/local/go/bin`
2. 注意设置 GOBIN 目录

### 包工具安装

根据不同的操作系统，使用不同的包管理工具。 但需要注意，一般来说包管理工具默认都不安装 go tools。

## 依赖

1. go get (下载/安装依赖库)
   根据 golang 版本区分：GOPATH 模式和 Module 模式。 如果是 Module 模式，那么会更新当前项目的 `go.mod` 文件，并且将指定版本下载到 module cache(附录 A].

   早期版本中，go get 会下载和安装依赖。在 1.19 以后，go get 主要是调整 go.mod 里面的依赖关系。 go install 会执行下载和安装工作。

   同时 go get 只能安装远程仓库，go install 支持安装本地库

2. go install (下载/编译/安装依赖库)
   go install 默认将编译后的二进制文件放置到 GOBIN 目录中，GOBIN 遍历规则参考[附录 B]。 所以特别需要注意，在 PATH 中需要添加 GOBIN。否则 install 以后的工具将无法使用。

3. go clean (清理资源)
   go clean 可以清理当前节点构建、测试和性能测试过程中产生的中间状态文件。 所有的 cache 默认在 GOCACHE 目录中。 `go clean -cache` 可以清除上面三个阶段所产生的所有缓存。 也可以通过指定" -testcache \ -fuzzcache " 单独清理某个阶段的缓存。

通过 GODEBUG 可以在上述三个阶段中输出缓存信息，辅助确认是否使用缓存

## 开发

- go env 输出所有和 golang 相关的环境变量
- go fmt(gofmt 的快捷方式) 代码格式化工具. 重点：
  1. go fmt 默认不对代码优化，如果需要代码优化(-s)，需要配置-r rule，自己写优化规则。官方提供的规则参看[附录 C]。
  2. go fmt 默认将代码格式化后的结果显示在当前终端，需要配合-w 写入到源文件
  3. 建议团队使用相同的 IDE，或者相同的 gofmt 规则，否则代码合并时会出现大量冲突
- go mod [依赖管理工具]

## 调试

- GDB
  - 通用性 debug，不支持 golang 特性调试
- LLDB
  - Mac 通用性 debug，不支持 golang 特性调试
- Delve(dlv)
  - dlv debug 进入调试模式
  - break main.main 在指定函数地方设置断点 (panic 会默认存在断点)
  - vars xxx 查看具体变量值
  - continue / next 继续执行或者下一步
  - goroutine 查看当前协程信息
  - stack 查看当前堆栈信息
- 使用调试工具时，需要满足两个条件:
  - 二进制文件携带调试信息
  - 存在源码文件

## 测试

- go test 自动对指定的路径/目录进行测试，并输出结果
  - 正确性测试
    - xxxx_test.go 文件和源文件需要保持在同一个目录
    - Test 开头，参数\*tesing.T.
    - 通过断言验证是否符合预期
    - 同一个函数可以创建多个 Test Case，名称不重复即可
  - 性能测试
    - 也属于 xxx_test.go 一部分
    - Benchmark 开头，参数为\*testing.B
    - 不需要验证正确性
- go test -v ./... 执行当前目录以及子目录下面所有的 test case。
- go test -cover ./... 显示测试代码覆盖率
- go test -v -bench ./... 执行当前目录以及子目录下面所有的 benchmark case。
  _ 通过 `-bench="Raw$"` 可以执行特定的 case
  _ 通过 `-benchtime=1x` 可以执行特定的次数或者时间

  注意：

  **go test 执行时，满足局部编译通过即可。 即准备 test 的目录构建成功，不需要整个工程都构建成功**。

- go vet 静态代码分析，这些分析只是建议性质，不一定表示一定有错误，需要用户自行确定。同时 go vet 会使用当前操作系统的依赖，有可能会运行失败

## 构建

- go run 快捷的编译运行方式。 go run 会尝试构建当前目录的代码，如果构建成功则会将二进制文件放入/tmp 目录中。每次执行 go run 都会重新构建一个新的二进制文件。
  1. go run 后面必须添加构建的文件。例如: go run main.go cli.go 或者 go run .
  2. go run 默认以当前操作系统为目标系统进行构建，可能存在依赖不满足的情况。
- 交叉编译
  - GOOS 表示目标操作系统，常用的包括:linux、darwin 和 windows。 但最终支持哪些目标系统需要执行"go tool dist list"查看。
  - GOARCH 表示目标操作系统指令集，同理常用的包括：386、amd64 和 arm。具体支持需要执行"go tool dist list"查看
- 编译优化
  - 去除调试信息 -ldflags "-w -s"

## 文档

- go doc(godoc) 文档生成工具。
  - go doc 可以查看 package document。 例如 go doc cmd/gofmt
  - godoc(单独下载安装 `go install golang.org/x/tools/cmd/godoc@latest` ) 可以创建一个 server，展示当前项目文档
    - godoc 在当前项目任意目录执行
    - godoc 会展示所有符合规范的文档描述(标准库、业务库和第三方库)，规范参看[附录 D]
    - godoc 只显示公开作用域的变量注释，私有作用域不展示

## 附录

### A module cache 位置

默认是在 $GOPATH/pkg/mod 。可以通过 GOMODCACHE 修改。 目前 golang(1.18)不会自动清理 Cache 里面的内容，所以如果 cache 目录 size 过大时，需要自行清理。

默认情况下，当前节点所有的 golang 工程会复用同一个 module cache。所以在 cache 中会保存有 package 的不同版本，通过`go get xxxxx @vx.y.z`的方式可以指定安装不同版本。

### B GOBIN 位置

GOBIN 默认按照以下顺序遍历：

a. $GOPATH/bin

b. $HOME/go/bin

c. $GOROOT/bin

d. $GOTOOLDIR

### C gofmt 常见规则

| 规则内容                                           | 规则描述                         |
| :------------------------------------------------- | :------------------------------- |
| (a) -> a                                           | 移除多余括号                     |
| α[β:len(α)] -> α[β:]                               | 显式切片转化成隐式切片           |
| []T{T{}, T{}} -> []T{{}, {}}                       | 显式类型转化成隐式类型           |
| for x, \_ = range v {...} -> for x = range v {...} | 优化迭代(未验证)                 |
| for \_ = range v {...} -> for range v {...}        | 优化迭代(未验证)                 |
| foo -> Foo                                         | 代码重构(语义替换，并非字符替换) |

### D 注释/文档规范

- 包注释
  如果当前 package 的描述信息超过 3 行，建议单独创建 `doc.go` 在 doc.go 中描述 package 的作用。例如:

```go
// Package bitmap.
//
// 注意godoc是根据句末的'.'来判断是否换行。 所以一定要在第一行最后添加'.'，否则不会换行。
//
// bitmap是一个实现了基本位图的工具。
// 每个bitmap实例最大可以承受18446744073709551615个元素，每个元素可以承载64位
//
// 但并不表示其可以支持18446744073709551615*64个状态位。当状态位> 18446744073709551615 时会因为超过MaxUint64而溢出
//
// 所以最多可以表示18446744073709551615个状态。
package bitmap
```

- 结构（接口）注释
  自定义 struct 或者 interface{}应该通过注释描述具体含义。struct 注释放在上面，而成员变量注释可以放在上面一行，也可以放在右面。例如:

```go
// BitMap 简易版bitmap
// 最大可以承受18446744073709551615个元素
// 每个元素可以承载64位
type BitMap struct {
   // u uint64组成的数组
   u []uint64
   max uint64 // max 支持最大可以修改的数
}
```

- 函数（方法）注释
  函数注释需要包含三部分内容: 名称、描述和返回值说明。例如:

```go
// New 初始化一个新的Bitmap
// size是状态数量(每个元素可以表示64位状态)
// e.g. 要表示500 种状态位,那么size = 500. 数组元素个数 = (500/64)+1
func New(size uint64) *BitMap {
   return &BitMap{
      u:   make([]uint64, (size/64)+1),
      max: size,
   }
}
```

- 代码逻辑注释

  - 重要逻辑建议添加注释，如果 TODO 建议添加 `//TODO` 注释。
  - 单行注释不要太长，不建议超过 120 个字符
  - 标点符号建议使用英文。

- 如果需要在 godoc 进行注释展示，注意注释之间默认不换行。如果需要换行，需要多加一个空白行。例如:

```go
// comment1
// comment2

会显示 comment1 comment2

// comment1
//
// comment2

会显示
comment1
comment2
```

- example 注释
  通过在 xxx_test.go 中添加 Examplexxx 函数，可以自动在 godoc 中添加对应的 example 文档。例如:

```go
func ExampleBitMap_Set() {
   // 初始化500个状态位的bitmap
   bt := New(500)

   // 将第二个bit位置为1
   err := bt.Set(2, One)
   if err != nil {
      log.Panicln(err)
   }
}
```

# Go AST 学习总结之来自标准库的无形之手

# 前情提要

golang 语法的三个主体:

1. expression 表达式
2. statement 语句
3. declaration 声明

这三类主体都是 Node 下面的具体实现。

在下面库中出现的 Decls 属于 declaration 声明。 具体来说有四大类:

- Spec type
  - Import Spec
  - Value Spec
  - Type Spec

* BadDecl 错误申明
* GenDecl 一般申明(和 Spec 相关,比如 import “a”,var a,type a)
* FuncDecl 函数申明

# go/ast

- Decls

  > 所有对外导出的顶级元素，包括 Struct、Variable、Function

- FileExports

  > 判断给定的代码是否存在可以导出的元素，变量，函数均可。 但判断的是顶级变量，也就是说函数内部变量不算。

- FilterDecl

  > 对给定的 AST 变量元素(struct、interface。 不包括函数)进行裁剪，裁剪的规则由`f` 确定。 当 f 返回 true 时保留 decl，反之当返回 false 时不保留 decl

- FilterFile

  > 对给定的 AST 所有元素进行裁剪(struct、interface 和 function)。 如果 f 返回 true，则保留 decl。 返回删除 decl。
  > 针对此函数的 decl，是由 src 迭代而来。 src 一般是通过`parser.ParseFile` 获取。

- FilterPackage
  > 对跟定的`package`里面所有的文件进行`FilterFile` 操作。 在使用之前，需要自行封装`package` 结构体，类似于:

```golang
ast.FilterPackage(&ast.Package{
   Name: file.Name.Name,
   Files: map[string]*ast.File{
      "filename": file,
   },
}, func(s string) bool {
   println(s)
   return false
})
```

> 同样，f 返回 true 则保留。返回 false 则移除。

- Fprint
  > 输出 AST 数据，多用于调试。 `FieldFilter` 用于控制特定的数据是否对外输出。但无论是 true / false， 都不会修改 ast 数据。 一般`Fprint`的用法如下:

```golang
ast.Fprint(os.Stdout, token.NewFileSet(), file, func(name string, value reflect.Value) bool {
   // true 则输出
   // false 不输出
   return false
})
```

- Inspect

  > 按照深度优先策略，从给定的`Node`开始通过执行`f(Node)`进行元素检查。 如果` f(Node)``返回了true `，则继续下一个元素。 如果返回了`false`，则终止遍历。`Inspect`过程结束。在遍历过程中，需要注意`Node`可能为`nil` . 需要进行 `Node != nil`的判断

- IsExported

  > 判断给定的字符串是否可导出。只要是首字母大写的元素都可导出

- NotNilFilter

  > 判断给定的`value`是否是空指针。 如果为`非空指针`时,返回`true`。 如果是`空指针`，则返回`false`

- PackageExports

  > 判断给定的`Package`是否存在可导出的变量。 其内部调用逻辑是循环遍历`Package`里面的文件，并依次对每个文件执行`FileExports` 操作。

- Print

  > `Fprint`的简化调用. 具体调用的是`Fprint(os.Stdout, fset, x, NotNilFilter)` 。 参数列表中的`fset` 一般通过`token.NewFileSet()` 获取，`x` 则是`ParseFile` 返回的`*ast.File`

- SortImports
  > 对 Import 的包进行重排序，同时会对重复的包进行合并处理。 重复的规则是 `包名和Path`完全相同

```
  log    "github.com/sirupsen/logrus"
  --     | -----------------------------
  包名                  PATH
```

- Walk
  > 对给定的 Node 按照深度优先规则进行遍历操作。如果`Visit`函数返回 nil，则终止遍历。 否则持续遍历。
  > 一般需要 `Visit` 中判断 Node 的类型，根据不同的类型进行不同的判断。 Walk 内部会不停的递归调用 Walk 函数，所以在 Visit 中可以通过修改所有者指针的方式修改数据。
  > 代码实例可以参考 [[怎么分析golang源代码]] - 全类型匹配

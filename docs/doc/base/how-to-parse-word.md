# 从如何解析文件来看如何实现词法解析 - 论 PEG 的简单使用

> 本文以 Pegjs 为演示语言

PEG 是`Parsing expression grammar`的缩写，翻译过来是`解析表达文法`。通俗的解释一下，PEG 解析器是为了将一段文本，转换成一个另一种数据形式，比如抽象语法树(AST)。

通常说到解析器时，就会和编译器联系在一起。众所周知，计算机最擅长处理二进制数据，而不擅长处理字符数据。而人类则恰恰相反，人类非常不喜欢看二进制数据。编译器就是用来调和这种矛盾的。

编译器负责将人类编写的`字符`类型的代码转译成`二进制`数据，从而让计算机可以轻松理解人类的意图。但编译器并没有人工智能，它无法通过阅读代码来猜测出代码背后的含义，编译器如果想将`字符`翻译成`二进制`。那么第一步就是按照规则来理解`字符`代码。

`按照规则`理解代码就称为分析语义。 当完成语义分析后，就可以`生成目标`,最后`计算结果`。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5h15aws52j20w6078dgl.jpg ":size=300")

解析器就是用来`分析语义`的。

比如很多讲解 PEG 的文章最喜欢使用的例子:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5h199tbbvj20ca04ma9z.jpg)

这段混合运算，如果让人来看，非常简单就可以明白计算规则。 但如果让计算机来看，它其实是看不懂的(至少在没有做语义分析之前是看不懂的)。计算机如果想看懂，第一步就是做语义分析，解析器会将这段混合运算拆解
成下面的结构:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5h1dinqwlj20js0cwaaz.jpg)

下一步计算机就可以根据这个树形结构来生成相对应的数据结构。

## 解析器如何工作

解析器工作时会分成两个步骤：

- 词法分析
- 语法分析

* 词法分析
  > 将文本按照一定的规则生成一个一个独立的词(类似于中文分词)。 没一个独立的词都可以称之为 token。
  > 生成 token 的过程纯粹就是分割词的过程，只要确保分割规则正确，那么分割后的词(token)基本都没问题

- 语法分析
  > 这才是解析器重点工作的地方。 解析器需要将分割后的 token 按照一定语法规则排列起来。 语法分析的结果产物其实就是这段代码的抽象语法树。 而解析器所遵循的语法就称之为`文法`

一般来说，现行的编译器所遵循的`文法`有两类:

1. CFG
2. PEG

- CFG 是`上下文无关法`。每个 CFG 规则中，会有两种组成元素：非终结符和终结符。非终结符意思是它还可以再展开，终结符则表明它已经是最小的单位，无法再展开。 CFG 规范规定，每条规则左边有且只有一个非终结符，右边可以由非终结符和终结符组成。例如:

  ```
  // a/b都是不可分割的token(终结符)
  S -> aSb | ab
  ```

每个 S 都可以被右边代替。 只要有 S 就可以被替代，而无论 S 出现在什么位置，前后左右都是啥。 所以这种 CFG 规则就不会关心 S 的上下文，所以称为上下文无关法。(上下文指的是左边 S 的上下文)

- PEG 则和 CFG 不同，PEG 是`分析型文法`。 PEG 采用`自顶向下`的方式，对字符进行递归解析。在形式上，一个解析表达文法由以下部分组成：

  - 一个有限的非终结符的集合 N
  - 一个有限的终结符的集合 Σ，和 N 没有交集
  - 一个有限的解析规则的集合 P
  - 一个被称作开始表达式的解析表达式 eS

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5h1yyuwa1j20tc05mmxl.jpg)

`e`可以是多个解析表达式的组合，也可以是原子表达式。 当如果是原子表达式时，e 由以下组成:

- 任何的终结符，
- 任何的非终结符，
- 空字符串 ε.

当如果是组合表达式时，e 可以通过以下操作构成：

- 序列: e1 e2
- 有序选择: e1 / e2
- 零个或更多: e\*
- 一个或更多: e+
- 可选: e?
- 肯定断言: &e
- 否定断言: !e

## PEG 示例

### 基本场景

现在假设需要解析一个配置文件(为了方便描述，我们假设解析 yaml 文件). 文件内容如下:

```yaml
UserRegDeptId: '[{"cpId":1},{"cpId":1133638,"deptId":"dp-6a123fc70f06420b920ebb27a7217fdb"}]'
```

在这个示例中，`UserRegDeptId`不是固定的，可能是`UserRegDeptId`,也可能是`UserRegDeptNo`。 如果采用编程语言解析，那么要么固定 key 值，要么解析成 map 结构。 而使用 peg 进行解析的话，我们仅仅需要描述这段配置的组成就够了。

```pegjs
Start
    = (SimpleKey)*


SimpleKey
    = key:Word': '_ value:WordContent NewLine{
        return {
            kind:'simpleKey',
            key:key,
            value:value
        }
    }

Word
	= l:(ALPHA_NUMERIC_CHARS)+ {
        return l.join('');
    }

WordContent
	= l:(ALPHA_NUMERIC_CHARS/ALPHA_SPILT_IDX)+ {
        return l.join('');
    }

ALPHA_SPILT_IDX
	= [:{}",]

ALPHA_NUMERIC_CHARS "数字"
    = [\[\]a-zA-Z0-9.'/-]

NewLine "换行符" = [\r\n]*

_ "空格或者换行符" = [ \t]*
```

上面说了，`peg`是自顶向下型文法。 所以需要首先定义`最顶层`文法。 `Start= (SimpleKey)*` 表示可以按照`SimpleKey`的文法循环解析文件。

`SimpleKey`中`key:Word': '_ value:WordContent NewLine`用来描述`key: value`这种结构。 `key:Word`表示在':'前面的字符是 key，':'后面可以跟任意多的`空格`，而在`空格`和`NewLine(换行符)`之间的则是`value`。

通过这样的文法描述，PEG 就可以依次对文件数据进行解析。 在`pegjs`中执行一下，得到如下结果:

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5h2brsr4rj21710u0jts.jpg ":size=300")

再输入任意多行数据，也可以轻松拿到语法树

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5h2chvbevj216g0u0tc8.jpg ":size=300")

这个示例属于简单场景，我们再加大难度。看如何匹配 yaml 中的对象。

### 对象场景

在 yaml 中定义对象可以这么写:

```yaml
Log:
  Access: logs/access.log
  Error: logs/error.log
  Stat: logs/stat.log
```

按照上面的步骤，我们需要首先定义顶层文法。

```
key:Word':' NewLine
```

这样会匹配类似于`Log:\n`这样的结构，然后开始匹配 Object 里面的 Item(这里为了简化难度，忽略了空格的问题)。

```
_ value:(_ SimpleKey)+
```

'_'匹配任意多的空格，然后`(_ SimpleKey)+`表示至少匹配一组`key:value`格式的数据。这样就可以匹配出`Access: logs/access.log`这样的数据（Access: logs/access.log 符合 SimpleKey）。

因为使用了`(_ SimpleKey)+`，所以会按照数组的模式进行匹配，因此`Access\Error\Stat`都会被匹配出来。

所以完整的 peg 语法定义如下:

```
Start
    = (SimpleKey/Object)*

Object
	= key:Word':' NewLine _ value:(_ SimpleKey)+{
    	return {
            kind:'Object',
            key:key,
            value:value
        }
    }


SimpleKey
    = key:Word':'_ value:WordContent NewLine{
        return {
            kind:'simpleKey',
            key:key,
            value:value
        }
    }

Word
	= l:(ALPHA_NUMERIC_CHARS)+ {
    	console.log(l)
        return l.join('');
    }

WordContent
	= l:(ALPHA_NUMERIC_CHARS/ALPHA_SPILT_IDX)+ {
    	console.log(l)
        return l.join('');
    }

ALPHA_SPILT_IDX
	= [:{}",]

ALPHA_NUMERIC_CHARS "数字"
    = [\[\]a-zA-Z0-9.'/-]

NewLine "换行符" = [\r\n]*

_ "空格或者换行符" = [ \t]*
```

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5h2jocd37j20ou0y475y.jpg ":size=300")

添加再多的 Object 也没问题，

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5h2k84s0bj20p20xytal.jpg ":size=300")

再增加一点难度，我们如何匹配注释

### 注释场景

```yaml
#model0中使用clustee:sssss
```

按照步骤，定义语法。 注释是`#`开始(不考虑行尾，如果考虑前面补充空格或者字符)，然后换行结束。 所以定义 peg 文法:

```
Comment
	= _ '#'_ value:(AnyWord) NewLine{
    	return {
        	kind:'Comment',
            value:value
        }
    }
```

AnyWord 有些特殊，是因为注释可能存在中文(其他配置出现中文可能性不大，但注释可能性会大一些)。所以需要单独考虑中文的场景。

```
AnyWord
	= l:(ALPHA_NUMERIC_CHARS/Unicode/WordContent)+ {
    	return l.join('')
    }

Unicode
	= [\u4E00-\u9FA5]
```

完整的 PEG 配置如下:

```
Start
    = (SimpleKey/Object/Comment)*

Comment
	= _ '#'_ value:(AnyWord) NewLine{
    	return {
        	kind:'Comment',
            value:value
        }
    }

Object
	= key:Word':' NewLine _ value:(_ SimpleKey)+{
    	return {
            kind:'Object',
            key:key,
            value:value
        }
    }


SimpleKey
    = key:Word':'_ value:WordContent NewLine{
        return {
            kind:'simpleKey',
            key:key,
            value:value
        }
    }

Word
	= l:(ALPHA_NUMERIC_CHARS)+ {
        return l.join('');
    }

AnyWord
	= l:(ALPHA_NUMERIC_CHARS/Uncode/WordContent)+ {
    	return l.join('')
    }

Uncode
	= [\u4E00-\u9FA5]

WordContent
	= l:(ALPHA_NUMERIC_CHARS/ALPHA_SPILT_IDX)+ {
        return l.join('');
    }

ALPHA_SPILT_IDX
	= [:{}",]

ALPHA_NUMERIC_CHARS "数字"
    = [\[\]a-zA-Z0-9.'/-]

NewLine "换行符" = [\r\n]*

_ "空格或者换行符" = [ \t]*
```

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5h2q0t594j20jg0y6jt8.jpg ":size=300")

### 其他场景

其他还有数组场景，对象嵌套对象场景和对象包含数组场景，所以提供一个完整的 PEG 文法:

```
Start
    = (SimpleKey/Object/Comment/ArrayObject/NestingOjbct/SimpleNestingObjectItem)*

Comment
	= _ '#'_ value:(AnyWord) NewLine{
    	return {
        	kind:'Comment',
            value:value
        }
    }

SimpleNestingObjectItem
	= key:Word':' NewLine WS WS value:(SimpleNestingObjectItemDetail)+ NewLine {
    	return {
        	kind: 'NestingObjectItem',
            key:key,
            value:value
        }
    }
SimpleNestingObjectItemDetail
	= value:(SimpleNestingObjectItemArrayItem)+ NewLine{
    	return {
        	kind: 'NestingObjectItemDetail',
            value:value
        }
    }

SimpleNestingObjectItemArrayItem
	= key:Word':'NewLine WS WS WS WS  '-'  value:(WSS NestingObjectItemArrayItemDetail)+ NewLine{
    	return {
        	kind: 'NestingObjectItemArrayItem',
            key:key,
            value:value
        }
    }

NestingOjbct
	= key:Word':'NewLine WS WS value:(NestingObjectItem)+ NewLine{
    	return {
        	kind: 'NestingObject',
            key:key,
            value:value
        }
    }

NestingObjectItem
	= key:Word':' NewLine WS WS WS WS value:(NestingObjectItemDetail)+ NewLine {
    	return {
        	kind: 'NestingObjectItem',
            key:key,
            value:value
        }
    }

NestingObjectItemDetail
	= value:(NestingObjectItemArrayItem)+ NewLine{
    	return {
        	kind: 'NestingObjectItemDetail',
            value:value
        }
    }

NestingObjectItemArrayItem
	= key:Word':'NewLine WS WS WS WS WS WS  '-'  value:(WSS NestingObjectItemArrayItemDetail)+ NewLine{
    	return {
        	kind: 'NestingObjectItemArrayItem',
            key:key,
            value:value
        }
    }

NestingObjectItemArrayItemDetail
	= value:(AnyWord) NewLine{
   	return {
        	kind: 'NestingObjectItemArrayItemDetail',
            value:value
        }
   }
ArrayObject
	= key:Word':' NewLine _ value:(_ ArrayObjectItem)+ NewLine {
    	return {
            kind:'ArrayObject',
            key:key,
            value:value
        }
    }

ArrayObjectItem
	= '-'value:(_ SimpleKey)+ NewLine {
    	return {
            kind:'ArrayObjectItem',
            value:value
        }
    }

Object
	= key:Word':' NewLine _ value:(_ SimpleKey)+{
    	return {
            kind:'Object',
            key:key,
            value:value
        }
    }

SimpleKeyNotValue
	= '-' _ value:(AnyWord)+{
    	return {
            kind:'SimpleKeyNotValue',
            value:value
        }
    }

SimpleKey
    = key:Word':'_ value:AnyWord NewLine{
        return {
            kind:'simpleKey',
            key:key,
            value:value
        }
    }

Word
	= l:(ALPHA_NUMERIC_CHARS)+ {
        return l.join('');
    }

AnyWord
	= l:(ALPHA_NUMERIC_CHARS/Uncode/WordContent)+ {
    	return l.join('')
    }

Uncode
	= [\u4E00-\u9FA5]

WordContent
	= l:(ALPHA_NUMERIC_CHARS/ALPHA_SPILT_IDX)+ {
        return l.join('');
    }

ALPHA_SPILT_IDX
	= [:{}",_@()?=&%]

ALPHA_NUMERIC_CHARS "数字"
    = [\[\]a-zA-Z0-9.'/-]

NewLine "换行符" = [\r\n]*

_ "空格或者换行符" = [ \t]*

WSS "空格或者换行符" = [ \t]+
WS "空格或者换行符" = [ \t]
```

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5h2t472bpj20tw0y6ju1.jpg ":size=400")

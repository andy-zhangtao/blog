# PEG

> WIP

## 待支持

- 字符串中包含':'的场景

## PEGJS 示例

- 处理`Object`

> 支持单 Object 模式和数组模式

```yaml
Tdmq:
  EndPoint: cmq-bj
  Region: ap-bj
  SecretId: 123
  SecretKey: 456
Tdmq:
  EndPoint: cmq-bj
  Region: ap-bj
  SecretId: 123
  SecretKey: 456
QQ:
  SEC: 123
```

```pegjs
Start
	= (Array)*

Array
	= Object


Object
	= key:(Word)':' NewLine _ value:(_ Item)+ {
    		return {
            	key:key,
                value:value
            }
    }

Item
	= key:Word':'_ value:Word NewLine {
    	return {
        	key:key,
        	value: value
        }
    }


Word
	= l:ALPHA_NUMERIC_CHARS+ {
        return l.join('');
    }

ALPHA_NUMERIC_CHARS "数字"
    = [a-zA-Z0-9.'/-]

NewLine "换行符" = [\r\n]*

_ "空格或者换行符" = [ \t]*

```

- 处理简单数据和 Object 数据
  > 支持简单数据和 Object 数据，也支持简单数据(数组)+Object 数据(数组)

```yaml
AppEnv: test
AppEnv1: test
Log:
  Access: logs/access.log
  Error: logs/error.log
  Stat: logs/stat.log
Log1:
  Access1: logs/access.log
```

```pegjs
Start
	= (Array)*

Array
	= Object/SimpleItem


Object
	= key:(Word)':' NewLine _ value:(_ Item)+ {
    		return {
            	key:key,
                value:value
            }
    }

Item
	= key:Word':'_ value:Word NewLine {
    	return {
        	key:key,
        	value: value
        }
    }


SimpleItem
	= _ value:Item+ NewLine {
    	return {
        	kind: "simple",
            value: value
        }
    }

Word
	= l:ALPHA_NUMERIC_CHARS+ {
        return l.join('');
    }

ALPHA_NUMERIC_CHARS "数字"
    = [a-zA-Z0-9.'/-]

NewLine "换行符" = [\r\n]*

_ "空格或者换行符" = [ \t]*
```

- 处理 Object 内数组
  > 适用于对象中包含数组对象

```yaml
ModelCache:
  - Host: "172.19.2.79"
    Pass: 12344
    Weight: 100
```

```pegjs
Start
	= (Array)*

Array
	= ObjectArray


ObjectArray
	= key:(Word)':' NewLine value:(ObjectArrayItem){
      return {
      	kind: "ObjectArray",
        key:key,
        value:value
      }
    }


ObjectArrayItem
	= _'-' _ value:(_ Item)+ {
    	return {
            kind: "ObjectArrayItem",
            value:value
        }
    }


Object
	= key:(Word)':' NewLine _ value:(_ Item)+ {
    		return {
            	key:key,
                value:value
            }
    }

Item
	= key:Word':'_ value:Word NewLine {
    	return {
        	key:key,
        	value: value
        }
    }


SimpleItem
	= _ value:Item+ NewLine {
    	return {
        	kind: "simple",
            value: value
        }
    }

Word
	= l:ALPHA_NUMERIC_CHARS+ {
        return l.join('');
    }

ALPHA_NUMERIC_CHARS "数字"
    = [a-zA-Z0-9.'/-]

NewLine "换行符" = [\r\n]*

_ "空格或者换行符" = [ \t]*
```

- 添加转义字符

```pegjs
Start
	= (Array)*

Array
	= ObjectArray/Object/SimpleItem/Item


ObjectArray
	= key:(Word)':' NewLine value:(ObjectArrayItem){
      return {
      	kind: "ObjectArray",
        key:key,
        value:value
      }
    }


ObjectArrayItem
	= _'-' _ value:(_ Item)+ {
    	return {
            kind: "ObjectArrayItem",
            value:value
        }
    }


Object
	= key:(Word)':' NewLine _ value:(_ Item)+ {
    		return {
            	kind: "Object",
            	key:key,
                value:value
            }
    }

Item
	= key:Word':'_ value:Word NewLine {
    	return {
        	kind: "Item",
        	key:key,
        	value: value
        }
    }


SimpleItem
	= _ value:Item+ NewLine {
    	return {
        	kind: "SimpleItem",
            value: value
        }
    }

Word
	= l:ALPHA_NUMERIC_CHARS+ {
        return l.join('');
    }

ALPHA_NUMERIC_CHARS "数字"
    = [\-\(\)?=&%a-zA-Z0-9.'/-/@]

NewLine "换行符" = [\r\n]*

_ "空格或者换行符" = [ \t]*
```

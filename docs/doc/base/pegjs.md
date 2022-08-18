# PEG

> WIP

## PEGJS 示例

- 处理`Object`

> 示例

```yaml
Tdmq:
  EndPoint: cmq-bj
  Region: ap-bj
  SecretId: 123
  SecretKey: 456
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

```

```

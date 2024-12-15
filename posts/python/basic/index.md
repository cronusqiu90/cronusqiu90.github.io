# Python 基础知识





## 变量和类型

变量是一种存储数据的载体，是存储器中存储数据的一块内存空间，变量的值可以被读取和修改，这是所有计算和控制的基础。计算机能处理各种各样的数据类型，像数字、文本、音视频、图像...... 每种数据类型都有其独特的属性和结构，那就需要定义不同的存储类型。

Python中的数据类型很多，也支持自定义数据类型:
- 数字: 整数、浮点数、复数、布尔值。
- 字符串：str，用单引号或双引号括起来的任意文本，可以包含各种字符。
- 列表：list，用方括号括起来的元素序列，可以包含不同类型的数据。
- 元组：tuple，用圆括号括起来的元素序列，元素不能修改。
- 字典：dict，用花括号括起来的键值对，键必须是不可变类型，值可以是任意类型。
- 集合：set，用花括号括起来的元素序列，元素无序且不重复。

### 命名规则

对于每个变量我们需要给它取一个名字，就如同我们每个人都有属于自己的响亮的名字一样。在Python中，变量命名需要遵循以下规则：

- 硬性规则：
  - 变量名由字母、数字和下划线构成，数字不能开头；
  - 大小写敏感；
  - 不要跟关键字和系统保留字冲突；

- 非硬性规则:
  - 变量名尽量简短、好记忆；
  - 变量名要能表达实际意义；
  - 遵循项目组的风格...

Tips：重视给变量的命名，好习惯真的很重要！

### 使用方法

```python

# 数字
a = 10
b = 20
print(a &#43; b)
print(a - b)

# 字符串
s1 = &#34;hello world
print(s1)

```

### 类型转换

内置的函数对变量类型进行转换:
- `int()`：将一个数值或字符串转换成整数，可以指定进制。
- `float()`：将一个字符串转换成浮点数。
- `str()`：将指定的对象转换成字符串形式，可以指定编码。
- `chr()`：将整数转换成该编码对应的字符串（一个字符）。
- `ord()`：将字符串（一个字符）转换成对应的编码（整数）

## 运算符号

| 运算符 | 描述  |
| --- | ---- |
| `[]` `[:]`            | 下标，切片                     |
| `**`                  | 指数                           |
| `~` `&#43;` `-`           | 按位取反, 正负号               |
| `*` `/` `%` `//`      | 乘，除，模，整除               |
| `&#43;` `-`               | 加，减                         |
| `&gt;&gt;` `&lt;&lt;`             | 右移，左移                     |
| `&amp;`                   | 按位与                         |
| `^` `\|`               | 按位异或，按位或               |
| `&lt;=` `&lt;` `&gt;` `&gt;=`     | 小于等于，小于，大于，大于等于 |
| `==` `!=`             | 等于，不等于                   |
| `is`  `is not`        | 身份运算符                     |
| `in` `not in`         | 成员运算符                     |
| `not` `or` `and`      | 逻辑运算符                     |

Tip: 多用用就熟悉了，在实际开发中不用太纠结优先级，牢记基本原则，如果太复杂了，强烈建议使用括号！

### 赋值运算

将 `=` 号右侧的值赋值给左侧的变量；
注意: Python 对变量类型没有限制，可以随时给变量赋任何类型的值，灵活也比较坑爹；

### 比较运算

也称为关系运算符，包括`==、!=、&lt;、&gt;、&lt;=、&gt;=`，没啥好说的，就是比较大小；

### 逻辑运算

有三个，包括 `and, or, not`。
- and: 两个条件都为真，结果才为真；
- or: 两个条件有一个为真，结果就为真；
- not: 取反，如果条件为真，结果为假，反之为真；
需要注意的就是短路效果；

### 位运算

按位运算符，包括`&amp;、|、^、~、&lt;&lt;、&gt;&gt;`，就是对二进制位进行操作；

## 分支结构

关键词: `if,elif, else`，可以根据条件选择执行的代码块；

```python
name = &#34;&#34;
password = &#34;&#34;

if name == &#34;admin&#34; and password = &#34;123456&#34;:
    print(&#34;Welcome admin!&#34;)

elif name == &#34;user&#34; and password = &#34;654321&#34;:
    print(&#34;Welcome %s!&#34; % name)

else:
    print(&#34;Invalid username or password.&#34;)
```

Tip: 如果逻辑中有多个 `if` 分支，建议使用 `dict` 来替代，后续会讲到；

## 循环结构

关键词: `for,while, continue, break`，可以重复执行代码块；
场景: 需要重复执行某条或某些指令

```python
# for 
sum = 0
for i in range(101):
    sum &#43;= i
print(&#34;sum=&#34;, sum)


# while
sum = 0
index = 0
while index &gt; 100:
    sum &#43;= index
print(&#34;sum=&#34;, sum)

# 
for i in range(100):
    if i % 100 == 0:
        continue        # 跳过 100 的倍数
    if i &gt; 900:
        break           # 退出循环
    print(&#34;i=&#34;, i)
```

## 实际运用例子

```python
# 寻找水仙花数
# 它是一个3位数，该数字每个位上数字的立方之和正好等于它本身
for num in range(100, 1000):
    low = num % 10
    mid = num // 10 % 10
    high = num // 100
    if num == low ** 3 &#43; mid ** 3 &#43; high ** 3:
        print(num)


# 正整数的反转
# 将12345变成54321
num = int(input(&#39;num = &#39;))
reversed_num = 0
while num &gt; 0:
    reversed_num = reversed_num * 10 &#43; num % 10
    num //= 10
print(reversed_num)


#《百钱百鸡》问题
# 公鸡5元一只，母鸡3元一只，小鸡1元三只，用100块钱买一百只鸡，问公鸡、母鸡、小鸡各有多少只？
for x in range(0, 20):
    for y in range(0, 33):
        z = 100 - x - y
        if 5 * x &#43; 3 * y &#43; z / 3 == 100:
            print(&#39;公鸡: %d只, 母鸡: %d只, 小鸡: %d只&#39; % (x, y, z))
```

## 函数
函数：将某些功能的代码封装起来，可以重复使用，提高代码的复用性。例如`加、减、乘、除、开方、取余、取整......`；

### 定义
Signure: `关键词 函数名(参数1,参数2,参数3...)`
- 关键词: `def`
- 函数名: 自定义的函数名，可以是任意合法的标识符
- 参数: 传入函数的变量，可以有多个，类型可以不同，支持可变参数、默认值等特性

例如:
```python
def fac(num):
    r = 1
    for n in range(1, num &#43; 1):
        r *= n
    return r

m, n = 5, 10
print(fac(m) // fac(n) // fac(m - n))
```

## 模块
Python 中 每个 `py` 文件就是一个模块；
将一系列相关函数、变量、类封装到一个模块中，可以更方便地使用；例如将数学上的计算方法，都封装到`math.py` 文件中，通过 `import math` 导入模块，就可以使用 `math` 库中的函数了；


---

> 作者: 迷途小狼崽  
> URL: http://localhost:1313/posts/python/basic/  

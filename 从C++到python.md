# 关于变量与内存的思维转换

对于已掌握C++的学习者来说，理解Python的第一步是忘掉“赋值即拷贝”，转而接受“赋值即贴标签”。

## 1. 变量观转换

- **C++（内存盒子）**：当我们写 `int a = 5;` 时，编译器在栈或堆上分配了4 字节的**固定空间**。变量名 `a` 就是这个空间的代号。当执行 `int b = a;` 时，系统在另一个地方开了新盒子，并把值复制了过去。

- **Python（命名标签）**：在 Python 中，`5` 是一个已经在内存中创建好的对象（`PyObject`）。`a = 5` 并不是把 5 放入变量 `a`，而是把 `a` 这个标签（Name）贴在了对象 `5` 上。
  
  - 这意味着在 Python 中，多个变量名可以同时贴在同一个对象上，且对象本身知道有多少标签贴在它身上（这就是引用计数）。

## 2. `PyObject*` 的本质

- 在 C++ 里，我们可以选择传递值（Value）或传递指针（Pointer）。但在 Python 的底层 C 实现中，**所有变量本质上都是 `PyObject*` 指针**。

- **浅拷贝**：这解释了为什么执行 `list_b = list_a` 时，修改 `list_b` 会影响 `list_a`。
  
  - 在C++中， 这相当于 `std::vector<int>* b = a;`（指针赋值），而不是 `std::vector<int> b = a;`（深拷贝对象）。
  
  - 如果你需要 C++ 习惯里的“独立副本”，必须显式调用 `list_b = list_a[:]`（切片拷贝）或 `copy.deepcopy()`。

## 3. `is` 与 `==`

- **`==` (Value Equality)**：对应 C++ 中的 **运算符重载 `operator==`**。它比较的是两个对象的内容（值）是否相等。

- **`is` (Identity Equality)**：对应 C++ 中的 **地址比较 `&a == &b`**。它检查两个变量是否指向内存中的**同一个地址**。
  
  - 在 Python 中，小整数（-5 到 256）会被提前创建并缓存（Interning）。所以 `a = 10; b = 10; a is b` 为 `True`，因为它们贴在了同一个缓存对象上。但对于大对象，即使内容相同，`is` 也可能返回 `False`。

## 4. 可变性（Mutable）与不可变性（Immutable）

- **C++ 逻辑**：对象的 `const` 属性由类型声明决定。

- **Python 逻辑**：对象的“可变性”由对象本身的类型决定。
  
  - **不可变对象（int, string, tuple）**：当你修改一个字符串时，Python 并不是在原地址改动（像 C++ 的 `char[]` 那样），而是**创建一个新字符串对象**，并把你的标签移过去。
  
  - **可变对象（list, dict, set）**：则允许在原地址修改内容，类似于 C++ 的 `std::vector` 或 `std::map`。

---

# 注释

### 单行注释

`#`

### 多行注释

` """ 多行字符串用三个引号 包裹，也常被用来做多行注释 """`

---

# 字符串

## 创建

字符串可以使用单引号（'）或者双引号（"） 

```
"这是个字符串"

'这也是个字符串'
```

## 字符列表

 `"Hello world!"[0]  #=> 'H'`

## f-strings 格式化字符串

```
name = "Reiko" 
f"She said her name is {name}." 
# => "She said her name is Reiko" 

#你可以在大括号内几乎加入任何 python 表达式，
#表达式的结果会以字符串的形式返回 
f"{name} is {len(name)} characters long." 
# => "Reiko is 5 characters long."
```

#### .format 格式化字符串

```
"{} can be {}".format("strings", "interpolated") 
# 可以重复参数以节省时间 
"{0} be nimble, {0} be quick, {0} jump over the {1}".format("Jack", "candle stick") 
# => "Jack be nimble, Jack be quick, Jack jump over the candle stick"
# 如果不想数参数，可以用关键字 
"{name} wants to eat {food}".format(name="Bob", food="lasagna") 
# => "Bob wants to eat lasagna"
```

---

# 算术符号

## 除法（不取整）

```
35 / 5 # => 7.0

10.0 / 3 # => 3.3333333333333335
```

整数除法的结果才是是向下取整

```
5 // 3 # => 1 5.0 // 3.0 # => 1.0 # 浮点数也可以

-5 // 3 # => -2 -5.0 // 3.0 # => -2.0
```

 i % j 结果的正负符号会和 j 相同，而不是和 i 相同

`-7 % 3 # => 2`

## 用 not 取非

```
not True # => False

not False # => True
```

## 大小比较可以连起来

```
1 < 2 < 3 # => True

2 < 3 < 2 # => False
```

---

# 容器类型

### 1. 列表 list

- 有序、可修改

- 用 `[]`

lst = [1, 2, 3, "苹果"]

### 2. 元组 tuple

- **有序、不能修改**

- 用 `()`
  
  `t = (1, 2, 3)`

### 3. 字典 dict

- **存 键：值**

- 用 `{}`
  
  `person = {"name": "小明", "age": 20}`

### 4. 集合 set

- **不重复、无序**

- 自动去重

- 用 `{}`

`s = {1, 2, 2, 3}  # 自动变成 {1,2,3}`

## 列表

Python 的 `list` 就像一个 `std::vector<std::any>`，能同时塞进整数、字符串和对象。这种灵活性在处理非结构化数据（如 JSON 解析）时极其高效，省去了 C++ 中复杂的结构体定义。

- ### 切割

```
li[1:3] # => [2, 4]

#取尾 li[2:] # => [4, 3]
```

- ### 取尾

```
li[:3] # => [1, 2, 4]
```

- ### 隔一个取一个
  
  `li[::2] # =>[1, 4]`

- ### 倒排列表
  
  `li[::-1] # => [3, 4, 2, 1]`

- ### 可以用三个参数的任何组合来构建切割
  
   `li[始:终:步伐]`

- ### 简单的实现了单层数组的深度复制
  
  `li2 = li[:] # => li2 = [1, 2, 4, 3] ，但 (li2 is li) 会返回 False`
* ### 总结
  
  切片里不写的位置，Python 会自动填默认值：**开头 = 0，结尾 = 最后，步长 = 1**

* ### 语法：
1. li.append(值)

2. li.pop()

3. del li[值] //删除索引为对应值的元素

4. li.remove(值)

5. li.insert(索引，元素值)

6. li.index(值) //获取索引

7. 值 in li     //判断值是否在列表

8. len(li)

9. li.extend(other_li) //拼接列表

## 元组

元组类似列表，但不允许修改

```
tup = (1, 2, 3) 
tup[0] # => 1 
tup[0] = 3 # 抛出 TypeError
```

如果元素数量为 1 的元组必须在元素之后加一个逗号。其他元素数量的元组，包括空元组，都不需要

```python
type((1)) # => <class 'int'> 
type((1,)) # => <class 'tuple'> 
type(()) # => <class 'tuple'>
```

### 操作

#### （列表允许的操作元组大多数都可以）

```python
len(tup) # => 3 
tup + (4, 5, 6) # => (1, 2, 3, 4, 5, 6) 
tup[:2] # => (1, 2) 
2 in tup # => True
```

#### 变量赋值

```python
# 可以把元组合列表解包，赋值给变量 
a, b, c = (1, 2, 3) # 现在 a 是 1，b 是 2，c 是 3 

# 也可以做扩展解包 
a, *b, c = (1, 2, 3, 4) # 现在 a 是 1, b 是 [2, 3]， c 是 4


# 元组周围的括号是可以省略的 
d, e, f = 4, 5, 6 # 元组 4, 5, 6 通过解包被赋值给变量 d, e, f


# 交换两个变量的值
e, d = d, e # 现在 d 是 5，e 是 4
```

## 字典

字典的 `key `必须为不可变类型。 这是为了确保 `key` 被转换为唯一的哈希值以用于快速查询。

Python 的 `dict` 性能极强且语法简单，相当于 C++ 的 `std::unordered_map`。理解了 `key` 必须是不可变类型（Hashable），也就理解了为什么 `list` 不能做 `key` 而 `tuple` 可以。

不可变类型包括整数、浮点、字符串、元组 。

```
invalid_dict = {[1,2,3]: "123"}  => 抛出 TypeError: unhashable type: list

valid_dict = {(1,2,3):[1,2,3]} # 然而 value 可以是任何类型
```

### 1.取值

dict[键]

### 2. 获取所有键

list(dict.keys())

### 3. 获取所有值

list(dict.values())

### 4. 测试

键 in dict //不能测值

filled_dict.get("one") # => 1
filled_dict.get("four") # => None

### 5. 插入新值

`setdefault(键，值)`

只有当键不存在的时候插入新值

```
filled_dict.setdefault("five", 5) # filled_dict["five"] 设为5

filled_dict.setdefault("five", 6) # filled_dict["five"] 还是5
```

### 6. 字典赋值

dict.update(键 : 值)
dict[键]=值

### 7. 删除

del dic[键]

## 集合

 元素必须为不可变类型

`set = {(1,) , 1}`

1. ### 添加元素
   
   `set.add(元素) //只有非重复元素才会加`

2. ### 取交集
   
   ```python
   other_set = {3, 4, 5, 6}              
   filled_set & other_set # => {3, 4, 5}
   ```

3. ### 取并集
   
   set1 | set2

4. ### 补集
   
   set1-set2

5. ### 异或集
   
   set1 ^ set2

6. ### 判断
   
   ```python
   set1 >= set2                          
   
   set1 <= set2
   ```

7. ### 复制
   
   `set3 = set1.copy()`

****

# `with` 语句的使用场景（替代 try/finally）

`with` 语句是 Python 专门用来**自动管理资源**的语法，核心作用：**无论代码正常 / 异常退出，都会<mark>自动执行收尾操作</mark>**，替代 `try/finally`。

只要一个对象**实现了「上下文管理器协议」**（`__enter__` 和 `__exit__` 方法），就可以用 `with`。

## 一、常用场景

### 1. 文件操作

**替代手动打开 / 关闭文件**，防止忘记 `close()` 导致文件句柄泄漏。

```python
# 推荐写法（with 自动关闭文件）
with open("test.txt", "r", encoding="utf-8") as f:
    content = f.read()

# 等价的 try/finally（繁琐）
f = open("test.txt", "r", encoding="utf-8")
try:
    content = f.read()
finally:
    f.close()  # 必须手动写，容易忘
```

### 2. 线程 / 进程锁（防止死锁）

操作锁时，**无论是否报错，都会自动释放锁**，替代 `lock.acquire()` + `finally: lock.release()`。

```python
import threading
lock = threading.Lock()

# with 自动加锁+解锁
with lock:
    # 临界区代码
    print("安全操作共享数据")
```

### 3. 数据库连接 / 游标（自动关闭连接）

确保**连接 / 游标一定会关闭**，避免数据库资源泄漏。

python

主流库：`pymysql`、`psycopg2`、`SQLAlchemy` 全部支持 `with`。

### 4. 网络套接字（Socket）

自动关闭 socket 连接，防止端口占用。

```python
import socket

with socket.socket() as s:
    s.connect(("127.0.0.1", 8080))
    s.send(b"hello")
```

### 5. 临时文件 / 目录（自动删除）

`tempfile` 模块：**用完自动删除文件 / 目录**，无需手动清理。

```python
from tempfile import TemporaryFile

# 自动创建+自动删除临时文件
with TemporaryFile() as f:
    f.write(b"临时数据")
```

## 二、核心判断规则

只要满足 **「需要手动关闭 / 释放 / 清理的资源」**，都优先用 `with`：

- 文件、流
- 锁、信号量
- 数据库连接、网络连接
- 临时文件、临时目录
- 自定义需要自动收尾的对象（自己写上下文管理器）

## 三、C++ 视角思考：

- **Python 版的 RAII**：`with` 语句的本质就是C++的 **RAII（资源获取即初始化）**。

- **生存期管理**：在C++中我们靠析构函数自动释放资源，在Python中则是靠 `__enter__` 和 `__exit__` 协议。这种显式的上下文管理，比手动写 `try...finally`（C++的手动 `delete` 或 `close`）更安全，能有效防止 RAG 处理大量语料文件时的句柄泄漏。

****

# **文件读写 + JSON 序列化**

## 1. `with open(...) as file`

专门用来自动管理文件，用完**自动关闭**，不用写 `try/finally`。是最标准、最安全的文件操作。

### 格式

```python
with open(文件名, 模式) as 变量名:
    操作文件
```

### 好处

- 不用手动 `file.close()`
- 即使报错，文件也会安全关闭
- 代码简洁清晰

## 2. `open()` 的参数详解

### ① 文件名

`"myfile1.txt"`直接写就是**当前文件夹**写路径就是指定位置，如 `"data/file.txt"`

### ② 打开模式（重点）

- `w+`：**可写 + 可读**，文件不存在则创建，**存在则清空覆盖**
- `r+`：**可读 + 可写**，文件必须存在，否则报错

常用模式总结：

- `r`：只读（默认）
- `w`：只写，清空覆盖
- `a`：追加写
- `r+`：读写，文件必须存在
- `w+`：读写，不存在创建，存在清空
- `a+`：读写，追加模式

## 3. 两种写入方式

### 方式 1：`str(contents)` → 转成普通字符串写入

```python
contents = {"aa":12,"bb":21}
with open("myfile1.txt", "w+") as file:
    fike.write(str(contents)) # 写入字符串到文件
```

- 把字典 **变成字符串** 写入

- 文件里内容：
  
  ```plaintext
  {'aa': 12, 'bb': 21}
  ```

- 缺点：
  
  - 读回来**还是字符串**，不是字典
    
    ```
    with open("myfile1.txt", "r+") as file: 
        contents = file.read() # 从文件读取字符串
    print(contents) # print: {"aa": 12, "bb": 21}
    ```
  
  - 不能直接用 `contents["aa"]`
  
  - 必须用 `eval()` 才能转回字典（不安全）

所以：**存普通文本可以，存数据不推荐**

### 方式 2：`json.dumps(contents)` → **标准 JSON 格式写入**

```python
with open("myfile2.txt", "w+") as file:
    file.write(json.dumps(contents))
```

- 把 Python 对象 → **标准 JSON 字符串**

- 文件里内容：
  
  ```plaintext
  {"aa": 12, "bb": 21}
  ```

- 优点：
  
  - 所有语言都能读（JS/Java/PHP…）
  
  - 读回来**直接变回字典**
    
    ```python
    with open("myfile2.txt", "r+") as file:
        contents = json.load(file)
    print(contents)
    # print: {"aa": 12, "bb": 21}
    ```

---

# 可迭代对象

**可迭代对象**：只要能被 `for` 循环遍历、能放进 `for x in 对象` 里跑的，就是可迭代。比如：<mark>字符串、列表、元组、字典 keys/values、集合、生成器</mark>等。

### 判断方法 1： 看是否继承 Iterable

导入模块，用**类型判断**：

```python
from collections.abc import Iterable

a = [1,2,3]
b = 123
c = {"one":1}.keys()

print(isinstance(a, Iterable))  # True
print(isinstance(b, Iterable))  # False
print(isinstance(c, Iterable))  # True
```

原理：**实现了 `__iter__` 方法的对象，都属于 Iterable，就是可迭代**。

### 判断方法 2：尝试迭代（容错法）

 `try-except` 试能不能遍历，不报错就是可迭代：

```python
def is_iterable(obj):
    try:
        iter(obj)
        return True
    except TypeError:
        return False

print(is_iterable([1,2]))   # True
print(is_iterable(100))     # False
```

### 特点

- **可以用 `for` 循环遍历**
- **可以用 `iter()` 生成一个迭代器**

### 可迭代对象的转化

```python
it = ["a", "b", "c"]
```

1. **tuple()** 转元组

```python
tuple(it)   # ('a','b','c')
```

2. **set()** 转集合（自动去重）

```python
set([1,2,2,3])  # {1,2,3}
```

3. **dict()** 转字典（要求元素是成对 (k,v)）

```python
dict([("one",1), ("two",2)]
```

### 内置函数：直接作用于可迭代对象

#### 1. for 循环（底层就是迭代器，最常用）

```python
for x in 可迭代对象:
    print(x)
```

#### 2. sum () 求和

```python
sum([1,2,3,4])
sum(range(1,101))
```

#### 3. max () /min () 最大最小值

```python
max([5,2,9])
min(range(5))
```

#### 4. any() / all()

- `any()`：只要有一个为真就返回 True
- `all()`：全部为真才返回 True

```python
any([0, "", False, 1])   # True
all([1,2,3])             # True
```

#### 5. sorted () 排序（返回新列表）

```python
sorted([3,1,2])   # [1,2,3]
```

#### 6. len () 长度（仅限**可迭代容器**，纯迭代器不能用 len）

```python
len([1,2,3])
len("abc")
len(iter([1,2,3]))  #报错 迭代器没有长度
```

### 迭代工具：itertools 标准库

专门**操作迭代器 / 可迭代对象**，不用自己写循环。先导入：

```python
import itertools
```

1. **itertools.count()** 无限自增迭代器

```python
for i in itertools.count(1, 2):
    print(i)
    if i > 10: break
```

2. **itertools.cycle()** 无限循环迭代

```python
for x in itertools.cycle([1,2,3]):
    print(x)
```

3. **itertools.chain()** 把多个可迭代对象拼在一起

```python
list(itertools.chain([1,2], [3,4]))  # [1,2,3,4]
```

4. **itertools.islice()** 切片迭代器（替代下标）
   
   迭代器不能 `[1:3]`，用这个：

```python
it = iter([1,2,3,4,5])
list(itertools.islice(it, 1, 3))  # [2,3]
```

### 推导式（本质就是迭代遍历）

都是基于可迭代对象：

1. 列表推导式（返回列表）

```python
[x*2 for x in range(5)]
```

2. 集合推导式

```python
{x%3 for x in range(10)}
```

3. 字典推导式

```python
{k:v for k,v in zip("abc", [1,2,3])}
```

### zip () 打包多个可迭代对象

把多个可迭代对象<mark>按位置配对</mark>，返回迭代器：

```python
a = [1,2,3]
b = ["x","y","z"]
list(zip(a,b))  # [(1,'x'),(2,'y'),(3,'z')]
```

### enumerate () 带下标迭代

返回<mark> 下标 + 元素</mark> 的迭代器：

```python
for idx, val in enumerate(["a","b","c"]):
    print(idx, val)
```

### 高阶函数：map /filter

1. **map**：对每个元素做加工

```python
list(map(str, [1,2,3]))  # ['1','2','3']
```

2. **filter**：过滤元素，保留满足条件的

```python
list(filter(lambda x: x>2, [1,2,3,4]))  # [3,4]
```

---

# 迭代器

**可迭代对象生成的、能记住遍历位置的对象**。

### 生活比喻

**迭代器 = >看书用的书签**

- 书签**只能往后翻**，不能跳页、不能回头
- 翻到最后一页，再翻就报错（读完了）
- 读完一遍，书签就失效了

## 生成

用 `iter(可迭代对象)` 生成：

```
our_iterator = iter(our_iterable)
```

## 读取

- **`next(迭代器)`**：取下一个元素，**自动记住位置**
- 读完所有元素后，再调用 `next()` 会抛出 `StopIteration`

```python
next(our_iterator)  # one （第一次，取第一个）
next(our_iterator)  # two （记住位置，取第二个）
next(our_iterator)  # three
next(our_iterator)  # 报错！读完了
```

## 与可迭代对象对比

| 特性           | 可迭代对象 (Iterable) | 迭代器 (Iterator)      |
| ------------ | ---------------- | ------------------- |
| 本质           | 存放所有元素的容器        | 记住读取位置的指针           |
| 能否反复遍历       | **可以**（读多少次都一样）  | **不可以**（只能读一遍）      |
| 能否随机访问 `[1]` | 部分可以（列表），部分不行    | **不行**              |
| 生成方式         | 直接创建（列表 / 字典键等）  | `iter(可迭代对象)`       |
| 取值方式         | `for` 循环         | `next()` / `for` 循环 |

```python
our_iterator = iter(our_iterable)
list(our_iterable)   # ["one","two","three"]  ✅ 可迭代对象，每次都完整

# 用 "next()" 获得下一个对象 
next(our_iterator) # => "one" 
# 再一次调取 "next()" 时会记得位置 
next(our_iterator) # => "two" 
next(our_iterator) # => "three"

list(our_iterator)   # []                     ❌ 迭代器，读完就空了
```

## `for` 循环的底层秘密

```python
for i in 可迭代对象:
    print(i)
```

**Python 内部流程**：

1. 调用 `iter(可迭代对象)` 生成迭代器
2. 不断调用 `next(迭代器)` 取值
3. 遇到 `StopIteration` 自动停止循环

**`for` 循环本质上是在自动使用迭代器**

---

# 函数

## 定义、调用、关键字参数调用

```
# 用def定义新函数 
def add(x, y): 
    print("x is {} and y is {}".format(x, y)) 
    return x + y # 用 return 语句返回
# 调用函数 
add(5, 6) # => 打印 "x is 5 and y is 6" 并且返回 11


# 也可以用关键字参数来调用函数 
add(y=6, x=5) # 关键字参数可以用任何顺序
```

## 可变参数函数

- `*args` = 装**普通参数** → 变成**元组 ()**
- `**kwargs` = 装 **名字 = 值** 参数 → 最后变成**字典 {}**

#### 1. `*args` 普通可变参数

```python
def varargs(*args):
    return args

varargs(1, 2, 3)  # (1, 2, 3)
```

意思：

- 函数里写 `*args`
- 无论传多少个**普通数字 / 值**进去
- 它都会自动打包成一个**元组**

example：`1,2,3` → 自动打包 → `(1,2,3)`

#### 2. `**kwargs` 关键字可变参数

```python
def keyword_args(**kwargs):
    return kwargs

keyword_args(big="foot", loch="ness")
# 结果：{"big": "foot", "loch": "ness"}
```

意思：

- 函数里写 `**kwargs`
- 传 `名字=值` 格式的参数
- 函数自动打包成字典

example

`big="foot", loch="ness"` → 打包 → `{"big":"foot", "loch":"ness"}`

#### 3. 两个一起用

```python
def all_the_args(*args, **kwargs):
    print(args)
    print(kwargs)
```

调用：

```python
all_the_args(1, 2, a=3, b=4)
```

输出：

```plaintext
(1, 2)        ← args 装普通参数
{"a": 3, "b":4} ← kwargs 装名字=值
```

#### 4. 反向操作：展开参数

有一个元组、一个字典：

```python
args = (1, 2, 3, 4)
kwargs = {"a": 3, "b": 4}
```

想把它们**拆开放进函数**：

* 用 `*` 展开元组

```python
all_the_args(*args)
# 等于：all_the_args(1,2,3,4)
```

* 用 `**` 展开字典

```python
all_the_args(**kwargs)
# 等于：all_the_args(a=3, b=4)
```

* 一起展开

```python
all_the_args(*args, **kwargs)
# 等于：all_the_args(1,2,3,4,a=3,b=4)
```

#### 总结

- `*args` = 收**一堆普通值** → 变成**元组**
- `**kwargs` = 收**一堆 名字 = 值** → 变成**字典**
- `*变量` = 把元组 / 列表**拆成一个个值**
- `**变量` = 把字典**拆成 名字 = 值**

从 C++ 视角看，Python 的 `*args` 与 `kwargs` 是对 **变长模板（Variadic Templates）** 的工程级简化：C++ 需要在编译期通过递归或折叠表达式处理类型包，而 Python 凭借动态特性在运行期将参数自动打包为 `tuple` 或 `dict`。这种机制配合 `*` 和 `**`解包操作符，完美实现了类似 C++ **完美转发（Perfect Forwarding）** 的功能，使开发者能像“透传代理”一样，仅用一行代码即可将参数原封不动地转发给底层 API。

## 用法

- 函数是一等公民：**函数可以赋值、可以当返回值、可以当参数**
- 嵌套函数 + 闭包：内层函数记住外层变量
- lambda：**一行临时小函数，不用起名**
- map 逐个加工元素；filter 按条件筛选元素

`map(函数, 可迭代对象)`作用：**把列表里每一个元素，都丢给这个函数处理**

```python
list(map(add_10, [1, 2, 3]))
```

`add_10` 是加 10 的函数：

- 1 → 1+10

- 2 → 2+10

- 3 → 3+10
  
  结果：`[11,12,13]`

map 传两个列表：

```python
list(map(max, [1,2,3], [4,2,1]))
```

两两配对取最大值：

- max(1,4)

- max(2,2)

- max (3,1)
  
  结果：`[4,2,3]`

### filter

`filter(判断函数, 可迭代对象)`作用：**只保留函数返回 True 的元素**

```python
list(filter(lambda x: x > 5, [3,4,5,6,7]))
```

规则：保留大于 5 的数剩下：`6,7`

---

# 模块

## 1. 整体导入模块

`import 模块名`

```python
import math
print(math.sqrt(16))  # => 4.0
```

## 2. 从模块导入指定函数

`from 模块名 import 函数1, 函数2`

```python
from math import ceil, floor
print(ceil(3.7))   # => 4.0  向上取整
print(floor(3.7))  # => 3.0  向下取整
```

使用方式：**直接写函数名**，不用加模块名

## 4. 模块别名（简化模块名）

`import 模块名 as 别名`

```python
import math as m
math.sqrt(16) == m.sqrt(16)  # => True
```

后续可以用**别名**代替原模块名，简写方便

## 5. 模块本质

1. Python 模块本质就是**一个普通的 .py 文件**
2. 可以自己写 Python 文件，当作模块导入使用
3. 模块名 = 对应的文件名（不带 `.py`）

## 6. 查看模块内部所有功能

使用 `dir(模块名)` 可以查看模块里所有的函数、变量

```python
import math
dir(math)
```

## 7. 模块导入优先级

1. **当前脚本所在文件夹** 的自定义模块，优先级**高于** Python 内置标准库

2. 建议：**不要用 Python 内置模块名给自己的文件命名**

---

# 类与面向对象

## 1. 什么是类（class）

用来批量创建 “对象” 的模板。比如：`Human` 类 → 可以创建无数个 “人” 对象（小明、小红、小李）

```python
class Human:
```

## 2. 类字段（类属性）

**所有实例共享的属性**改一次，所有人都变。

```python
species = "H. sapiens"
```

特点：

- 属于**类**
- 所有对象共用
- 改 `Human.species = 新值`，全部实例都会变

## 3. 构造方法 `__init__`

**创建对象时自动执行的方法**用来给对象**初始化属性**（名字、年龄、性别等）

```python
def __init__(self, name):
    self.name = name    # 给对象绑定名字
    self._age = 0       # 内部属性
```

- `self` = **当前这个对象自己**
- `self.name` = 这个对象的名字

## 4. 实例方法

**对象自己能用的方法**第一个参数永远是 `self`。

```python
def say(self, msg):
    print(f"{self.name}: {msg}")
```

使用：

```python
i = Human("Ian")
i.say("hi")
# 输出：Ian: hi
```

## 5. 类方法 `@classmethod`

**整个类共用，不单独属于某个对象**第一个参数是 `cls`（代表类本身）

```python
@classmethod
def get_species(cls):
    return cls.species
```

特点：

- 所有实例共享
- 可以通过**类名**或**对象**调用

```python
i.say(i.get_species())  # "Ian: H. sapiens"
```

## 6. 静态方法 `@staticmethod`

**跟对象无关、跟类也无关**，就是个普通函数放类里而已。

```python
@staticmethod
def grunt():
    return "*grunt*"


# 运行静态方法 (staticmethod) 
print(Human.grunt()) # => "*grunt*"

# 实例上也可以执行静态方法 
print(i.grunt()) # => "*grunt*"
```

- 不需要 `self` 或 `cls`
- 类、对象都能调用

## 7. `@property` 装饰器

它让**方法可以像属性一样使用**

### ① 只读属性（getter）

```python
@property
def age(self):
    return self._age
```

用法：

```python
print(i.age)   # 不用加 ()
```

### ② 可写属性（setter）

```python
@age.setter
def age(self, age):
    self._age = age
```

用法：

```python
i.age = 42  # 直接赋值

# 访问实例的属性 
i.say(i.age) # => "Ian: 42" 
j.say(j.age) # => "Joel: 0"
```

### ③ 可删除属性（deleter）

```python
@age.deleter
def age(self):
    del self._age
```

用法：

```python
del i.age
```

## 8. 实例化对象（创建人）

```python
i = Human("Ian")
j = Human("Joel")
```

- `i`、`j` 都是 Human 的**实例（对象）**
- 它们都有自己的 `name`、`age`
- 共用 `species`、类方法、静态方法

## 9. 类属性

**类属性是共享的**

```python
Human.species = "H. neanderthalensis"
i.say(i.get_species()) # => "Ian: H. neanderthalensis"
j.say(j.get_species()) # => "Joel: H. neanderthalensis"
```

## 10. `if __name__ == '__main__':`

意思：**只有直接运行这个文件时，下面的代码才执行**。如果被别的文件导入，不执行

作用：

- 测试代码
- 防止模块被导入时自动运行

## 11. 完整运行流程

```python
i = Human("Ian")    # 创建 Ian
i.say("hi")         # Ian 说 hi

i.age = 42          # 设置年龄
i.say(i.age)        # 输出 Ian: 42

del i.age           # 删除年龄
i.age               # 报错：属性没了
```

## 12. 总结

```plaintext
类（Human）
  ├─ 类属性：species（共享）
  ├─ __init__：构造函数
  ├─ 实例方法：say()、sing()
  ├─ 类方法：get_species()
  ├─ 静态方法：grunt()
  └─ property：age（可读、可写、可删）
```

1. **`__init__(self)`**：初始化对象
2. **`self`**：代表当前对象
3. **实例方法**：对象用
4. **`@classmethod`**：类共用
5. **`@staticmethod`**：普通函数放类里
6. **`@property`**：让方法变属性（. 属性名）
7. **类属性**：所有对象共享
8. **`if __name__ == '__main__'`**：主程序入口

---

# 类与继承

## 一、类继承基础概念

**继承**是Python面向对象编程的核心特性，简单来说：**子类（派生类）可以继承父类（基类/超类）的所有属性和方法**，无需重复编写代码，同时子类还能**重写父类内容**或**新增专属内容**，实现代码复用和功能扩展。
**核心术语**：

- **父类**：被继承的原始类（如示例中的Human人类类）
- **子类**：继承父类的新类（如示例中的Superhero超级英雄类）
- **方法重写**：子类定义和父类同名的方法/属性，覆盖父类原有内容
- **super()**：专门用于子类调用父类方法的内置函数

## 二、继承的基本语法

### 1. 单继承语法

```python
# 父类定义
class 父类名:
    父类的属性和方法

# 子类继承父类，括号内写父类名
class 子类名(父类名):
    # 子类新增/重写的属性和方法
    pass  # 无新增内容时用pass占位，直接继承父类所有内容
```

### 2. 跨文件导入父类继承

如果父类写在单独的.py 文件中（模块化开发），需先导入再继承，文件名不带.py 后缀：

```python
# 从human.py文件中导入Human父类
from human import Human

# 子类继承导入的父类
class Superhero(Human):
    pass
```

## 三、子类的核心操作

以 Human 为父类，Superhero 为子类，结合示例代码拆解核心操作：

### 1. 重写父类属性

子类直接定义和父类同名的属性，即可覆盖父类的属性值，所有子类实例都会使用新属性。

```python
# 父类Human
class Human:
    species = "H. sapiens"  # 父类属性：人类

# 子类Superhero
class Superhero(Human):
    species = 'Superhuman'  # 重写父类属性，变为超人类
```

### 2. 重写父类构造方法__init__

构造方法是实例化对象时自动执行的初始化方法，子类重写__init__时，**必须用 super () 调用父类构造方法**，否则父类的初始化逻辑不会执行，父类属性无法正常使用。

```python
class Superhero(Human):
    # 重写构造方法：保留父类的name参数，新增专属参数
    def __init__(self, name, movie=False, superpowers=["super strength", "bulletproofing"]):
        # 新增子类专属属性
        self.fictional = True
        self.movie = movie  # 是否为电影角色，默认False
        self.superpowers = superpowers  # 超能力列表

        # 关键：调用父类的构造方法，完成父类属性初始化
        super().__init__(name)
```

 **注意**：子类构造方法的默认参数如果是可变类型（如列表），会被所有实例共享，尽量避免直接用可变对象做默认值，或在构造方法内重新赋值。

### 3. 重写父类普通方法

子类定义和父类同名的方法，实例调用该方法时，优先执行子类重写后的逻辑，而非父类原有方法。

```python
# 父类Human的sing方法
def sing(self):
    return 'yo... yo... microphone check... one two... one two...'

# 子类Superhero重写sing方法
def sing(self):
    return 'Dun, dun, DUN!'
```

### 4. 子类新增方法

子类可以定义父类没有的方法，仅子类实例能调用，父类实例无法使用。

```python
# 子类专属方法：展示超能力
def boast(self):
    for power in self.superpowers:
        print("I wield the power of {pow}!".format(pow=power))
```

### 5. 子类调用父类方法的方式

子类想使用父类**被重写**或**原有**的方法，必须用**super()**，这是最规范、最推荐的方式，无需手动传入 self。

常用场景：

- 调用父类构造方法：`super().__init__(参数)`
- 调用父类普通方法：`super().方法名(参数)`

```python
# 子类中调用父类的say方法
def parent_say(self, msg):
    super().say(msg)  # 直接调用父类原版say方法
```

## 四、继承相关核心内置函数与属性

### 1. isinstance ()：判断实例归属

判断一个对象是否是某个类 / 其父类的实例，返回布尔值，**子类实例同时属于子类和父类**。

```python
sup = Superhero(name="Tick")
# 判断sup是否是Human的实例
print(isinstance(sup, Human))  # True
# 判断sup是否是Superhero的实例
print(isinstance(sup, Superhero))  # True
```

### 2. type ()：获取实例真实类型

获取对象的原始创建类，只会返回当前实例的直接类，不会向上追溯父类。

```python
print(type(sup))<class '__main__.Superhero'>
print(type(sup) is Superhero)  # True
```

### 3. **mro**：方法解析顺序

全称 Method Resolution Order，即 Python 查找方法 / 属性的顺序，**先找子类自身，再找父类，最后找顶级父类 object**，所有类默认继承 object 基类。

```python
# 查看Superhero的方法解析顺序
print(Superhero.__mro__)
# 输出结果：<class '__main__.Super'>,<class '__main__.Human'>, <class 'object'>)
```

## 五、完整示例代码

```python
# 父类：人类
class Human:
    species = "H. sapiens"  # 类属性（所有实例共享）

    def __init__(self, name):
        self.name = name  # 实例属性
        self._age = 0

    def say(self, msg):
        print("{name}: {message}".format(name=self.name, message=msg))

    def sing(self):
        return 'yo... yo... microphone check... one two... one two...'

    @classmethod
    def get_species(cls):
        return cls.species

    @staticmethod
    def grunt():
        return "*grunt*"

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, age):
        self._age = age

# 子类：超级英雄，继承Human
class Superhero(Human):
    # 重写父类属性
    species = 'Superhuman'

    # 重写构造方法
    def __init__(self, name, movie=False, superpowers=["super strength", "bulletproofing"]):
        self.fictional = True
        self.movie = movie
        self.superpowers = superpowers
        # 调用父类构造方法
        super().__init__(name)

    # 重写父类方法
    def sing(self):
        return 'Dun, dun, DUN!'

    # 子类专属方法
    def boast(self):
        for power in self.superpowers:
            print("I wield the power of {pow}!".format(pow=power))

# 主程序测试
if __name__ == '__main__':
    sup = Superhero(name="Tick")

    # 类型判断
    if isinstance(sup, Human):
        print('I am human')
    if type(sup) is Superhero:
        print('I am a superhero')

    # 查看方法解析顺序
    print("方法解析顺序：", Superhero.__mro__)

    # 调用父类方法
    print(sup.get_species())  # 输出Superhuman（子类重写后的属性）
    print(sup.sing())  # 输出子类重写后的唱歌方法
    sup.say('Spoon')  # 调用父类say方法
    sup.boast()  # 调用子类专属方法

    # 使用继承的属性
    sup.age = 31
    print(sup.age)  # 输出31
    print('Am I Oscar eligible? ' + str(sup.movie))  # 输出False
```

---

# 多重继承

## 一、核心概念

多重继承是Python面向对象编程的扩展特性，指**一个子类可以同时继承多个父类**，无需重复编写多个父类的属性和方法，实现更灵活的代码复用。

## 二、多重继承的基本语法

### 1. 基础语法格式

```python
# 导入需要继承的多个父类（跨文件导入时，需确保文件路径正确）
from 父类文件1 import 父类1
from 父类文件2 import 父类2
# 多重继承：子类括号内依次写入多个父类class 子类名(父类1, 父类2, ...):    # 子类的属性、方法（可重写、可新增）    pass
```

## 2. 代码示例

```python
# 导入两个父类：Superhero（超级英雄类）和 Bat（蝙蝠类）
from superhero import Superhero
from bat import Bat
# 定义Batman子类，同时继承Superhero和Bat
class Batman(Superhero, Bat):    
# 子类的具体实现    
  pass
```

## 三、核心知识点

### 1. 先明确两个父类的核心内容

### 父类1：Bat（蝙蝠类，来自bat.py）

```python
class Bat:    
species = 'Baty'  # 类属性：蝙蝠物种    
# 构造方法，初始化“是否会飞”，默认值为True    
def __init__(self, can_fly=True):        
    self.fly = can_fly  # 实例属性：是否会飞    

# 与Human类同名的say方法    
def say(self, msg):        
    msg = '... ... ...'        
    return msg    

# Bat类独有方法：声呐    
def sonar(self):        
    return '))) ... ((('
```

### 父类2：Superhero（超级英雄类，来自superhero.py）

继承自Human类，核心特点：

- 继承Human的属性（name、age）和方法（say、get_species等）；

- 重写Human的species（改为Superhuman）和sing方法；

- 拥有自身属性（movie、superpowers）和方法（boast）。

### 2. 多重继承的构造方法

多重继承与单继承的核心区别：**不能仅靠super()初始化所有父类**，因为super()只会按MRO顺序调用“下一个父类”的构造方法，无法同时初始化多个父类，需显式调用每个父类的构造方法。

```python
class Batman(Superhero, Bat):    
    def __init__(self, *args, **kwargs):        
    # 1. 显式调用第一个父类Superhero的构造方法        
        Superhero.__init__(self, 'anonymous', movie=True, superpowers=['Wealthy'], *args, **kwargs)        
    
    # 2. 显式调用第二个父类Bat的构造方法        
        Bat.__init__(self, *args, can_fly=False, **kwargs) 

    # 3. 子类重写name属性，覆盖Superhero构造方法中的默认值        
        self.name = 'Sad Affleck'
```

关键细节：

- 显式调用父类构造时，必须传入self（直接调用类的方法，需手动绑定实例）；

- 可在调用时修改父类构造的默认参数（如Bat的can_fly改为False）；

- *args、**kwargs：接收不确定的参数，避免参数传递报错，提升代码灵活性。

### 3. MRO 方法解析顺序

MRO（Method Resolution Order）即方法解析顺序，指Python查找子类调用的方法/属性时，遵循的固定顺序，用于解决多重继承中“同名方法/属性”的冲突。

#### 查看MRO的方法

```python
# 格式：子类名.__mro__print(Batman.__mro__)
```

输出结果（对应示例代码）：

```python
#(<class '__main__.Batman'>, 
#<class 'superhero.Superhero'>, <class 'human.Human'>, 
#<class 'bat.Bat'>, <class 'object'>)
```

#### MRO顺序解读

查找顺序：子类 → 第一个父类 → 第一个父类的父类 → 第二个父类 → ... → 顶级父类object

对应示例顺序：Batman → Superhero → Human → Bat → object

#### MRO优先级验证

Bat类和Human类都有say方法：

```python
sup = Batman()sup.say('I agree')  # 输出：Sad Affleck: I agree
```

原因：MRO中Human类在Bat类前面，优先执行Human的say方法（拼接name和msg），而非Bat的say方法（仅返回省略号）。

### 4. 多重继承的方法/属性调用

结合示例代码，逐类说明调用规则：

- 子类重写的方法：优先执行（如Batman的sing方法，覆盖父类）；

- 父类的方法：按MRO顺序查找，找到第一个匹配的执行（如say方法执行Human的）；

- 父类独有的方法：子类可直接调用（如Bat的sonar方法）；

- 父类的属性：子类可继承、修改（如Human的age属性，Bat的fly属性）。

```python
# 调用子类重写的sing方法
print(sup.sing())  
# 输出：nan nan nan nan nan batman!

# 调用第二个父类Bat的独有方法
sonarprint(sup.sonar())  
# 输出：))) ... (((

# 继承并修改第一个父类的age属性
sup.age = 100
print(sup.age)  # 输出：100
```

## 四、多重继承注意事项

- 父类顺序决定MRO优先级：括号内第一个父类的优先级高于后续父类，其方法/属性会被优先查找，不可随意调整顺序；

- 构造方法必须显式调用：需手动调用每个父类的__init__(self, ...)，否则父类的属性无法正常初始化，仅用super()无法满足多重继承需求；

- 同名方法/属性冲突：完全依赖MRO顺序解决，无需额外配置，记住“谁在MRO前面就执行谁”；

- 参数传递要灵活：使用*args和**kwargs接收不确定的参数，避免调用父类构造时出现参数不匹配的错误；

- 避免滥用多重继承：若多个父类功能重叠过多，会导致代码逻辑混乱、难以维护，必要时可用“单继承+组合”替代；

- 跨文件导入注意：确保父类文件路径正确，避免导入失败；同时避免自定义类名、文件名与Python内置模块/类同名，防止覆盖。

## 五、总结

1. 多重继承语法：class 子类名(父类1, 父类2, ...)，括号内父类顺序决定优先级；

2. MRO是多重继承的核心规则，决定方法/属性的查找顺序，可用子类名.__mro__查看；

3. 构造方法需显式调用每个父类的__init__，不能仅依赖super()；

4. 子类可继承多个父类的所有内容，可重写父类方法/属性，也可新增专属内容；

5. 核心原则：灵活复用代码，明确MRO顺序，规避同名冲突和构造方法初始化问题。

---

# 学习总结

作为C++背景的学习者，初看Python也许会觉得‘太松散’，但深入后发现它的核心在于抽象能力。很多在 C++ 中需要关注的底层内存分配（如 RAII、指针解引用、哈希碰撞处理），Python 都通过 `with`、`dict` 和内置协议（Protocol）封装成了直觉式的语法。学习 Python 不是为了放弃性能，而是为了在不需要手动管理内存的场景下，去关注更高层的逻辑。

Python 的“鸭子类型”（只要走起来像鸭子，就是鸭子）极大地降低了模块间的耦合成本。在处理 LLM 返回的不确定格式的 JSON 数据时，Python 的灵活性展现出极大优势，不再需要写繁琐的 `struct` 或 `class` 映射。

在 C++ 里配置一个第三方（CMake/vcpkg）有时很累。而Python 的 `import` 机制和第三方生态（尤其是 itertools 这种高效迭代工具），给开发带去更多便捷性。不再需要重复造轮子，而是站在巨人的肩膀上，把精力集中在解决业务问题上。

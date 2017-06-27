<center>**python语法小结**</center>

[TOC]
# 变量类型
## 赋值
Python 中的变量赋值不需要类型声明。

每个变量在内存中创建，都包括变量的标识，名称和数据这些信息。

每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建。

等号（=）用来给变量赋值。

- is操作
== 值相等is,两个变量名指向同一个地址的时候返回True

- 赋值的引用和拷贝

```python
 L = [1, 2, 3]
 M = ['a', L, "c"]
 print M
#['a', [1, 2, 3], 'c']
 L[0] = 4
 print M
#['a', [4, 2, 3], 'c']

#无限制条件的分片生成一个高级拷贝
 L = [1, 2, 3]
 M = ['a', L[:], 'c']
 print M
#['a', [1, 2, 3], 'c']
 L[0] = 4
 print M
#['a', [1, 2, 3], 'c']
```

- 重复

```python
 l = [1, 2, 3]
 x = l * 4
 y = [l]*4
 print x
#[1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3]
 print y
#[[1, 2, 3], [1, 2, 3], [1, 2, 3], [1, 2, 3]] 
```

- 三元运算：

```python
x if y else z
```

## 字符串
python的字串列表有2种取值顺序:

从左到右索引默认0开始的，最大范围是字符串长度少1
从右到左索引默认-1开始的，最大范围是字符串开头
（同lua)

- enumerate, 偏移元素
每次迭代都会返回一个（index，value)的元组

```python
s = 'dhalk'
for (offset, item) in enumerate(s):
    print(item, 'appears at offset', offset)

('d', 'appears at offset', 0)
('h', 'appears at offset', 1)
('a', 'appears at offset', 2)
('l', 'appears at offset', 3)
('k', 'appears at offset', 4)
```

- rstrip()
Python rstrip() 删除 string 字符串末尾的指定字符（默认为空格）
str.rstrip([chars])

- s[1:10:2]
从s[1]到s[10],步距为2

### 格式化字符串
将一个值插入到一个有字符串格式符 %s 的字符串中。（同sprintf)

```python
print "My name is %s and weight is %d kg!" % ('Zara', 21) 
```

### 三引号
python中三引号可以将复杂的字符串进行复制:

python三引号允许一个字符串跨多行，字符串中可以包含换行符、制表符以及其他特殊字符。
三引号的语法是一对连续的单引号或者双引号（通常都是成对的用）

同lua中的[[...]]

### 字符串内建函数
http://www.runoob.com/python/python-strings.html

## 数据格式
- 列表用 "[ ]" 标识类似 C 语言中的数组。

- 元组用 "( )" 标识。内部元素用逗号隔开。但是元组不能二次赋值，相当于只读列表。

- 字典用 "{ }" 标识。字典由索引 key 和它对应的值 value 组成。

- 任意数据格式之间可以进行任意的嵌套

### 列表
列表用 [ ] 标识，是 python 最通用的复合数据类型。

列表中值的切割也可以用到变量 [头下标:尾下标] ，就可以截取相应的列表，
从左到右索引默认 0 开始，从右到左索引默认 -1 开始，下标可以为空表示取到头或尾。

加号 + 是列表连接运算符，星号 * 是重复操作。

| Python 表达式               |  结果                       | 描述 |
|----------------------------|-----------------------------|-----|
|len([1, 2, 3])              |3                            |长度|
|[1, 2, 3] + [4, 5, 6]       |[1, 2, 3, 4, 5, 6]           |组合|
|['Hi!'] * 4                 |['Hi!', 'Hi!', 'Hi!', 'Hi!'] |重复|
|3 in [1, 2, 3]              |True                         |元素是否存在于列表中|
|for x in [1, 2, 3]: print x |1 2 3                        |迭代|

#### 列表函数&方法
http://www.runoob.com/python/python-lists.html

### 元组
元组是另一个数据类型，类似于List（列表）。

元组用"()"标识。内部元素用逗号隔开。但是元组不能二次赋值，相当于只读列表

- 元组，列表搜索

```python
 dict = ('1da', '2fr', 3)
 dict.index('1da')
#0
 dict.index('8')
#Traceback (most recent call last):
  #File "<stdin>", line 1, in <module>
#ValueError: tuple.index(x): x not in tuple 

 dict = ['12', '43', 'hfks']
 dict.index('12')
#0
 dict.index('hdka')
#Traceback (most recent call last):
  #File "<stdin>", line 1, in <module>
#ValueError: 'hdka' is not in list

```

### 字典
字典(dictionary)是除列表以外python之中最灵活的内置数据结构类型。列表是有序的对象结合，字典是无序的对象集合。

两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。

字典用"{ }"标识。字典由索引(key)和它对应的值value组成。
（同lua table)

字典是另一种可变容器模型，且可存储任意类型对象。

字典的每个键值(key=>value)对用冒号(:)分割，每个对之间用逗号(,)分割，整个字典包括在花括号({})中 

- 格式如下所示：

```python
dict = {'Name': 'Zara', 'Age': 7, 'Class': 'First'}; 
print "dict['Alice']: ", dict['Alice'];
```

- 字典取值防呆

```python
Ma = {
    '909':'4342'
}
print(Ma.get('123', '0'))
#key‘123’不存在的时候返回默认值‘0’
```

- 字典分别打印key和value

```python
 dict = {'a':'1', 'b':'2', 'c':3}
 print(list(dict.keys()))
#['a', 'c', 'b']
 print(list(dict.values()))
#['1', 3, '2']
 print(list(dict.items()))
#[('a', '1'), ('c', 3), ('b', '2')]
```

- zip得到字典

```python
keys = ['a', 'b', 'c']
values = [1, 2, 3]
dd = dict(zip(keys, values))
print dd
#{'a': 1, 'c': 3, 'b': 2}
```

#### 内置方法
http://www.runoob.com/python/python-dictionary.html

### map
在序列中映射函数

map()函数接收两个参数，一个是函数，一个是序列，map将传入的函数依次作用到序列的每个元素，并把结果作为新的list返回

```python
 print map(lambda x: x % 2, range(7))
#[0, 1, 0, 1, 0, 1, 0] 
```

# 条件语句
## if

```python
if 判断条件1:
    执行语句1……
elif 判断条件2:
    执行语句2……
elif 判断条件3:
    执行语句3……
else:
    执行语句4……
```

**python 并不支持 switch 语句，所以多个条件判断，只能用 elif 来实现**
## switch case实现

```python
 branch = {
    's': '1',
    'd': '2',
    'g': '7'
  }
 print branch.get('s', 'noSuchThing')
#1
 print branch.get('w', 'noSuchThing')
#noSuchThing 
```

# 循环语句
## While
while … else 在循环条件为 false 时执行 else 语句块：

```python
while True:
    pass
else:
    pass 
    #在不是break退出循环的时候使用
```

## for
Python for循环可以遍历任何序列的项目，如一个列表或者一个字符串。

```python
# 遍历字符串
for letter in 'Python':     
   print '当前字母 :', letter

# 遍历列表
fruits = ['banana', 'apple',  'mango']
for fruit in fruits:       
   print '当前水果 :', fruit

# 通过序列索引迭代
fruits = ['banana', 'apple',  'mango']
for index in range(len(fruits)): 
   print '当前水果 :', fruits[index]

# 字典,数组
for key in Ks:
    print (key, '%s'%Ks[key])
```
在 python 中，for … else 表示这样的意思，for 中的语句和普通的没有区别，else 中的语句会在循环正常执行完（即 for 不是通过 break 跳出而中断的）的情况下执行，while … else 也是一样。

**range不包括边界**

```python
for i in range(3):
    print i

0
1
2
```

# 函数
以下是简单的规则：

1. 函数代码块以 def 关键词开头，后接函数标识符名称和圆括号()。
2. 任何传入参数和自变量必须放在圆括号中间。圆括号之间可以用于定义参数。
3. 函数的第一行语句可以选择性地使用文档字符串—用于存放函数说明。
4. 函数内容以冒号起始，并且缩进。
5. return [表达式] 结束函数，选择性地返回一个值给调用方。不带表达式的return相当于返回 None。

```python
def functionname( parameters ):
   "函数\_文档字符串"
   function\_suite
   return [expression]
```

## 参数传递
在 python 中，strings, tuples, 和 numbers 是不可更改的对象，而 list,dict 等则是可以修改的对象。

- 不可变类型

变量赋值 a=5 后再赋值 a=10，这里实际是新生成一个 int 值对象 10，再让 a 指向它，而 5 被丢弃，不是改变a的值，相当于新生成了a。

- 可变类型

变量赋值 la=[1,2,3,4] 后再赋值 la[2]=5 则是将 list la 的第三个元素值更改，本身la没有动，只是其内部的一部分值被修改了。

python 函数的参数传递：

- 不可变类型

类似 c++ 的值传递，如整数、字符串、元组。如fun（a），传递的只是a的值，没有影响a对象本身。
比如在 fun（a）内部修改 a 的值，只是修改另一个复制的对象，不会影响 a 本身。

- 可变类型

类似 c++ 的引用传递，如 列表，字典。
如 fun（la），则是将 la真正的传过去，修改后fun外部的la也会受影响

```python
#传不可变对象实例
def ChangeInt( a ):
    a = 10
b = 2
ChangeInt(b)
print b # 结果是 2

# 传可变对象实例
def changeme( mylist ):
   "修改传入的列表"
   mylist.append([1,2,3,4]);
   print "函数内取值: ", mylist
   return
 
mylist = [10,20,30];
changeme( mylist );
print "函数外取值: ", mylist
#函数内取值:  [10, 20, 30, [1, 2, 3, 4]]
#函数外取值:  [10, 20, 30, [1, 2, 3, 4]]
```

## 参数
以下是调用函数时可使用的正式参数类型：
必备参数
关键字参数
默认参数
不定长参数
### 必备参数
必备参数须以正确的顺序传入函数。调用时的数量必须和声明时的一样
（同正常的函数参数传递）
### 关键字参数
使用关键字参数允许函数调用时参数的顺序与声明时不一致，因为Python解释器能够用参数名匹配参数值

```python
#可写函数说明
def printinfo( name, age ):
   "打印任何传入的字符串"
   print "Name: ", name;
   print "Age ", age;
   return;
 
#调用printinfo函数
printinfo( age=50, name="miki" );
```
### 缺省参数
调用函数时，缺省参数的值如果没有传入，则被认为是默认值。下例会打印默认的age，如果age没有被传入：

```python
#可写函数说明
def printinfo( name, age = 35 ):
   "打印任何传入的字符串"
   print "Name: ", name;
   print "Age ", age;
   return;
 
#调用printinfo函数
printinfo( age=50, name="miki" );
printinfo( name="miki" );
```
### 不定长参数
- 基本语法

```python 
def functionname([formal\_args,] *var\_args\_tuple ):
   "函数\_文档字符串"
   function\_suite
```

加了星号（*）的变量名会存放所有未命名的变量参数。选择不多传参数也可

```python 
# 可写函数说明
def printinfo( arg1, *vartuple ):
   "打印任何传入的参数"
   print "输出: "
   print arg1
   for var in vartuple:
      print var
   return;
 
# 调用printinfo 函数
printinfo( 10 );
printinfo( 70, 60, 50 );
```

- 函数参数匹配

```python
def func(*name) #匹配并收集（在元组中）所有包含位置的参数
def func(**name) #匹配并收集（在字典中）所有包含位置的参数
def func(*args, name) #参数必须在调用中按照关键字传递

def func(*argc):
    print(argc)

func(1, 2, 3)
 (1, 2, 3)

def f2(**arge):
    print(arge)

f2(a=1, b=2, c=3)
 {'a': 1, 'c': 3, 'b': 2}

def f3(a, *args, **kargs):
    print(a, args, kargs)

f3(1, 2, 3, x=1, y=2)
 (1, (2, 3), {'y': 2, 'x': 1})
```

- 函数通用性
根据不同情况动态调用函数

```python
if <test>:
    action = func1
    args = (1)
else:
    action = func2
    args = (1, 2, 3)

action(*argc)
```

## 匿名函数
python 使用 lambda 来创建匿名函数。

- lambda只是一个表达式，函数体比def简单很多。
- lambda的主体是一个表达式，而不是一个代码块。仅仅能在lambda表达式中封装有限的逻辑进去。
- lambda函数拥有自己的命名空间，且不能访问自有参数列表之外或全局命名空间里的参数。
- 虽然lambda函数看起来只能写一行，却不等同于C或C++的内联函数，后者的目的是调用小函数时不占用栈内存从而增加运行效率。
- lambda主体必须是一个单个表达式而不是一些语句

```python
 f = lambda x, y, z: x+y+z

f(2, 3, 4)
 #9
```
lambda通常被用来编写跳转表

```python
l = [
    lambda x: x ** 2,
    lambda x: x ** 3,
    lambda x: x ** 4
]
for f in l:
    print(f(2))
4
8
6
print(l[0](3))
 9
```
以字典来构建行为表

```python
 dict2 = {
...     '111' : lambda x: x+1+1,
...     '222' : lambda x: x+2+2
... }
 
 print dict2['111'](3)
#5
```

## 函数装饰器@
是后边函数运行时的声明

```python
class c:
    @staticmethod
    def meth():

#等价于
class c:
    def meth():
        meth = staticmethod(meth)
```

# 模块
Python 模块(Module)，是一个 Python 文件，以 .py 结尾，包含了 Python 对象定义和Python语句。

模块让你能够有逻辑地组织你的 Python 代码段。

把相关的代码分配到一个模块里能让你的代码更好用，更易懂。

模块能定义函数，类和变量，模块里也能包含可执行的代码。
## import 语句
程序第一次导入文件时候会执行三个步骤：

1. 找到模块文件
2. 编译成位码.pyc文件（需要时）
3. 执行模块的代码来创建其所定义的对象

python将载入的模块存储到一个sys.modules的表中

- \_\_init\_\_.py
扮演一个初始化的勾子，替目录产生模块命名空间以及使用目录导入时实现from *
首次导入某个目录的时候，会自动执行该目录下\_\_init\_\_.py文件中的所有程序代码

- \_\_all\_\_
from *语句只会把\_\_all\_\_列表中的变量名复制出来

- \_\_name\_\_
每个模块都有名为\_\_name\_\_的内置属性（自动设置）
1. 文件以顶层程序文件执行，在启动时，\_\_name\_\_会设置为字符串"\_\_main\_\_"
2. 如果文件被导入，\_\_name\_\_就被设为客户端说了解的模块名

# 文件I/O
## 标准输入输出
### 打印到屏幕
print函数

```python
print "dhaklhdasklhl"
print('-'*80) #print多个
```

### 读取键盘输入
#### raw\_input函数
raw\_input([prompt]) 函数从标准输入读取一个行，并返回一个字符串（去掉结尾的换行符)
#### input函数
input([prompt]) 函数和 raw\_input([prompt]) 函数基本类似，但是 input 可以接收一个Python表达式作为输入，并将运算结果返回。

## 文件操作
http://www.runoob.com/python/file-methods.html
### open()

```python
#确保退出后可以自动关闭文件
with open('test.txt') as myfile:
    for line in myfile:
        pass
```
### write()
### read()
read（）方法从一个打开的文件中读取一个字符串。需要重点注意的是，Python字符串可以是二进制数据，而不是仅仅是文字。

语法：
fileObject.read([count]);

在这里，被传递的参数是要从已打开文件中读取的字节计数。该方法从文件的开头开始读入，如果没有传入count，它会尝试尽可能多地读取更多的内容，很可能是直到文件的末尾。
### 文件定位
- tell()

tell()方法告诉你文件内的当前位置；换句话说，下一次的读写会发生在文件开头这么多字节之后。

- seek()

seek（offset [,from]）方法改变当前文件的位置。Offset变量表示要移动的字节数。From变量指定开始移动字节的参考位置。

如果from被设为0，这意味着将文件的开头作为移动字节的参考位置。如果设为1，则使用当前的位置作为参考位置。如果它被设为2，那么该文件的末尾将作为参考位置。

### 重命名和删除文件
os.rename(current\_file\_name, new\_file\_name)
os.remove(file\_name)
### 目录
- mkdir()
可以使用os模块的mkdir()方法在当前目录下创建新的目录们。
os.mkdir("newdir")
- chdir()
可以用chdir()方法来改变当前的目录。chdir()方法需要的一个参数是你想设成当前目录的目录名称。
- getcwd()
getcwd()方法显示当前的工作目录。
- rmdir()
rmdir()方法删除目录，目录名称以参数传递。在删除这个目录之前，它的所有内容应该先被清除。

### 文件方法
http://www.runoob.com/python/os-file-methods.html
http://www.runoob.com/python/python-files-io.html



# 面向对象
## 类

```python
class Employee:
   '所有员工的基类'
   empCount = 0
 
   def __init__(self, name, salary):
      self.name = name
      self.salary = salary
      Employee.empCount += 1
   
   def displayCount(self):
     print "Total Employee %d" % Employee.empCount
 
   def displayEmployee(self):
      print "Name : ", self.name,  ", Salary: ", self.salary
```
- empCount变量是一个类变量，它的值将在这个类的所有实例之间共享。你可以在内部类或外部类使用 Employee.empCount 访问。
- \_\_init\_\_()方法是一种特殊的方法，被称为类的构造函数或初始化方法，当创建了这个类的实例时就会调用该方法
- self
代表类的实例，self 在定义类的方法时是必须有的，虽然在调用时不必传入相应的参数。(this指针)

## 类的属性
- getattr(obj, name[, default]) : 访问对象的属性。
- hasattr(obj,name) : 检查是否存在一个属性。
- setattr(obj,name,value) : 设置一个属性。如果属性不存在，会创建一个新属性。
- delattr(obj, name) : 删除属性。
- \_\_dict\_\_ : 类的属性（包含一个字典，由类的数据属性组成）
- \_\_doc\_\_ :类的文档字符串
- \_\_name\_\_: 类名
- \_\_module\_\_: 类定义所在的模块
（类的全名是'\_\_main\_\_.className'，如果类位于一个导入模块mymod中，那么className.\_\_module\_\_ 等于 mymod）
- \_\_bases\_\_ : 类的所有父类构成元素（包含了一个由所有父类组成的元组）

## 垃圾回收
Python 使用了引用计数这一简单技术来跟踪和回收垃圾。

在 Python 内部记录着所有使用中的对象各有多少引用。

一个内部跟踪变量，称为一个引用计数器。当对象被创建时，就创建了一个引用计数，当这个对象不再需要时，也就是说，这个对象的引用计数变为0 时，它被垃圾回收。但是回收不是"立即"的，由解释器在适当的时机，将垃圾对象占用的内存空间回收。
（同java)

**rec = 0，内存会自动清理变量内存**

## 继承
继承语法 
**class 派生类名（基类名）：**
//... 基类名写在括号里，基本类是在类定义的时候，在元组之中指明的。

在python中继承中的一些特点：

1. 在继承中基类的构造（\_\_init\_\_方法）不会被自动调用，它需要在其派生类的构造中亲自专门调用。
2. 在调用基类的方法时，需要加上基类的类名前缀，且需要带上self参数变量。区别于在类中调用普通函数时并不需要带上self参数
3. Python总是首先查找对应类型的方法，如果它不能在派生类中找到对应的方法，它才开始到基类中逐个查找。（先在本类中查找调用的方法，找不到才去基类中找）。

## 重载

```python
class Parent:        # 定义父类
   def myMethod(self):
      print '调用父类方法'
 
class Child(Parent): # 定义子类
   def myMethod(self):
      print '调用子类方法'
 
c = Child()          # 子类实例
c.myMethod()         # 子类调用重写方法
```

## 类属性与方法
### 类的私有属性
\_\_private\_attrs：
两个下划线开头，声明该属性为私有，不能在类的外部被使用或直接访问。在类内部的方法中使用时 self.\_\_private\_attrs。
### 类的方法
在类的内部，使用 def 关键字可以为类定义一个方法，与一般函数定义不同，类方法必须包含参数 self,且为第一个参数
### 类的私有方法
\_\_private\_method：

两个下划线开头，声明该方法为私有方法，不能在类地外部调用。在类的内部调用 self.\_\_private\_methods

单下划线、双下划线、头尾双下划线说明：

- \_\_foo\_\_: 
定义的是特列方法，类似 \_\_init\_\_() 之类的。
- \_foo: 
以单下划线开头的表示的是protected类型的变量，即保护类型只能允许其本身与子类进行访问，不能用于 from module import *
- \_\_foo: 双下划线的表示的是私有类型(private)的变量, 只能是允许这个类本身进行访问了。

# 异常
- try／except
捕捉异常并恢复

```python
try:
    raise IndexError #触发内置异常
except IndexError:
    print('got exception')
```

- try/finally
无论异常是否发生，执行清理行为。
无论try中是不是有异常，都会执行finally中的语句
- raise
手动在代码分中触发异常
- assert
有条件的在程序中触发异常
- with/as
环境管理器
- try／except/else

```python
try:
    pass #主要动作
except <name1>:
    pass #出现异常的处理器
except <name2>:
    pass
else:
    pass #没有发生异常时要执行的处理器
```

- try语句分句形式

| 分句形式                       | 说明                               |
| ----------------------------- | --------------------------------- |
| except:                       | 捕捉所有异常（容易出错）              |
| except name:                  | 捕捉特定异常                        |
| except name, value:           | 捕捉所列的异常和其额外的数据          |
| except (name1, name2):        | 捕捉列出的任何异                     |
| except (name1, name2), value: | 捕捉列出的任何异常，并获取其额外数据    |
| else:                         | 如果没有引发异常就运行                |
| finally:                      | 总是运行该代码块                     |

# 正则表达式
http://www.runoob.com/python/python-reg-expressions.html 














































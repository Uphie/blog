---
title: Python中的list小结
author: Uphie
date: 2017-07-11 21:30:00 +0800
categories: [技术]
tags: [python]
math: true
toc: true
---


# 分片

和其它语言类似，列表的下标也从 `0` 开始，先取一个列表作为示例

```
>>> l=['a','b','c','d','e','f','g','h']
```
1.指定要截取的起始索引，以下取的是 [2,5) 区间

```
>>> l[2:5]
['c', 'd', 'e']
```
听过你写过 django？然而 django 中的分片和这个分片有点不太一样。django 不是查询出来再从结果中分片（效率太渣了），它是用了懒查询，也就是说它会把分片翻译成数据库执行语句中的分段查询语句。
```
class Book(models.Model):
    name = models.CharField(max_length=40)

    class Meta:
        db_table = 'book'

books = Book.objects.all()[2:10]
```
上面语句执行时，会翻译成下面的sql 语句进行查询（以mysql为例）
```
SELECT `book`.`id`, `book`.`name` FROM `book` LIMIT 8 OFFSET 2
```
2.指定要截取的结束索引，以下取的是 [0,5) 区间

```
>>> l[:5]
['a', 'b', 'c', 'd', 'e']
```
3.指定要截取的结束索引，但是结束索引为负数，如下，结束索引为-2，则结果去除从后往前的两个元素

```
>>> l[:-2]
['a', 'b', 'c', 'd', 'e', 'f']
```
4.获取全部
```
>>> l[:]
['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
```
这时发现得到的结果和原列表的元素一样。那么这两个列表是不是一样呢？？？

```
>>> L=l[:]
>>> id(l)
4322675656
>>> id(L)
4322656072
```
可以看到，新的列表和旧的列表不是同一个数组，因为内存地址不同。可是，还有没有其它联系呢？？？
```
>>> id(l[0])
4321323528
>>> id(L[0])
4321323528
>>> id(l[-1])
4321419648
>>> id(L[-1])
4321419648
```
这里看到，虽然两个列表的内存地址不同，但是其元素的内存地址还是一样的。那么列表它在分片的时候其实是把元素的引用做成新的列表并返回的。


5.可以使用步长，如步长为3的话，则表示每三个取一个元素
```
>>> l[::3]
['a', 'd', 'g']
```
当步长为负数时，可以实现倒序
```
>>> l[::-1]
['h', 'g', 'f', 'e', 'd', 'c', 'b', 'a']
```
6.如果索引超出范围，不会抛出异常
```
>>> l[:20]
['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
>>> l[20:]
[]
>>> l[10:20]
[]

```
# 索引

继续拿上面的列表 `l` 举例

1.非负索引
```
>>> l[0]
'a'
```
2.负索引
```
>>> l[-1]
'h'
```
3.索引超出范围时会抛出异常
```
>>> l[20]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
>>> l[-10]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
```
# 嵌套

列表不仅可以是一维的，还可以是多维的，还可以是混合的
```
>>> l.append(['i','j','k'])
>>> l
['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', ['i', 'j', 'k']]

```
下面我们做个笛卡尔乘积来创造出一个二维列表
```
>>> m=['A','B']
>>> n=['a','b','c']
>>> l=[[x,y] for x in m for y in n]
>>> l
[['A', 'a'], ['A', 'b'], ['A', 'c'], ['B', 'a'], ['B', 'b'], ['B', 'c']]
```


# 操作符运算

## +
```
>>> l=['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
>>> m=[1,2,3]
>>> id(l)
4322868616
>>> l=l+m
>>> n
['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 1, 2, 3]
>>> id(l)
4322848584
```
可以看到通过 `+` 运算符得到的是一个完全新的列表。

## +=

列表还可以这样相加
```
>>> l=['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
>>> m=[1,2,3]
>>> id(l)
4322868168
>>> l+=m
>>> l
['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 1, 2, 3]
>>> id(l)
4322868168
```
这次得到了和上面一样的结果，但是 `l` 的内存地址没有发生变化，说明 `+=` 是在自身进行扩展元素，而且列表是`可变对象`。

## *

列表可以乘以某个倍数实现复制扩增
```
>>> l=['a','b','c']
>>> L=l*3
>>> L
['a', 'b', 'c', 'a', 'b', 'c', 'a', 'b', 'c']
```
那么扩增出来的新列表还是原来那个列表吗？
```
>>> id(l)
4322848648
>>> id(L)
4322868168
```
看来答案是否定的，使用`*`操作符得到的是一个新的列表。


可不可以相减呢？
```
>>> L=['a','b','c']
>>> n=l-L
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for -: 'list' and 'list'
```
奥，不行

## in

in 用于判断某一对象是否在列表中
```
>>> l=[1,2,3,4,5]
>>> 5 in l
True
>>> 7 in l
False
```

其实上面的操作本质是重载了运算符方法，如 `+` 重载了`__add__()` 方法，`in` 是重载了`__contains__()`方法。

参考 list，我们也可以模仿一个
```
class Vector():
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        self.x = self.x + other.x
        self.y = self.y + other.y
        return self

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __str__(self):
        return '({0},{1})'.format(self.x, self.y)


if __name__ == '__main__':
    a = Vector(4, 8)
    print('a:', a)
    b = Vector(2, 7)
    print('b:', b)
    c = a + b
    print('a+b=', c)
    d = Vector(6, 10)
    print('d:', d)
    print('a+b=d? ', c == d)
```
运行结果是
```
a: (4,8)
b: (2,7)
a+b= (6,15)
d: (6,10)
a+b=d?  False
```

## > == <
这个不是表情符号，这个是大小比较符号 :)

列表可以使用 `>` 、`<` 、`==` 来比较大小，这时候你应该会恍然想到在许多语言中 字符串可以比较大小，依据是字符串序列中每个字符的 `ASCII` 值的大小，那么 list 是不是也是也类似呢？

```
>>> a=[1,4,6,2]
>>> b=[2]
>>> c=[1,3]
>>> d=[1,4,6,4]
>>> e=[1,4,6,2]
>>> a<b
True
>>> a>c
True
>>> a>d
False
>>> a==e
True
```
这时候可以看到，列表比大小，是按照依次比较元素的大小来决定列表的大小的，如果两个列表的第一个元素相等，则分出胜负，如果不相等则比较下一个，依次如此。

等下，如果不同类型的元素比较呢？
```
>>> a=[1,4,6,2]
>>> f=[1,4,6,'a']
>>> a>f
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unorderable types: int() > str()
```
结果是不可以的。

对于元素为 str 的列表来说也是如此
```
>>> g=['a','b']
>>> h=['b','b']
>>> i=['a','c']
>>> j=['a','b','cook']
>>> g<h
True
>>> g>i
False
>>> g<j
True
```

如果列表中为类的对象呢？
```
>>> class Apple():
...     pass
...
>>> a=Apple()
>>> b=a
>>> c=Apple()
>>> l=[a]
>>> L=[b]
>>> l1=[a]
>>> l2=[b]
>>> l3=[c]
>>> l1>l2
False
>>> l1==l2
True
>>> l1>l3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unorderable types: Apple() > Apple()
```
这时候，你会发现，如果元素为相同的类实例对象的引用的话，那么是可以比较大小，就是等于嘛！但是如果不是相同的类实例对象的引用的话，就不能比较大小了。

等等，这个推论确定对吗？看看上面出过的两次错误 `TypeError: unorderable types`，你应该想到根本问题应该是这里。之所以不能比较，是因为 `int()` 和 `str()` 与 `Apple()` 和 `Apple()` 都是 `unorderable`的，即不可比较的。不同类型的不能比较可以理解，对于同类型的我们可以改造下，让它 `orderable`。
```
>>> class Apple():
...     def __init__(self,price):
...             self.price=price
...     def __lt__(self,other):
...             return self.price<other.price
...     def __gt__(self,other):
...             return self.price>other.price
...     def __le__(self,other):
...             return self.price<=other.price
...     def __ge__(self,other):
...             return self.price>=other.price
...     def __eq__(self,other):
...             return self.price==other.price
...
>>> a=Apple(10)
>>> b=Apple(20)
>>> l1=[a]
>>> l2=[b]
>>> l1<l2
True
>>> a<b
True
```
这样就好啦 :-D
# 排序

## list.sort()
```
>>> l=[3,5,2,12,9,45]
>>> l.sort()
>>> l
[2, 3, 5, 9, 12, 45]
```
如果列表中的元素较复杂，可以指定排序方式
```
>>> l=[{'chinese':87,'math':76},{'chinese':83,'math':66},{'chinese':93,'math':96}]
>>> l.sort(key=lambda d:d['chinese'])
>>> l
[{'math': 66, 'chinese': 83}, {'math': 76, 'chinese': 87}, {'math': 96, 'chinese': 93}]
```
还可以实现降序排序
```
>>> l.sort(reverse=True)
>>> l
[45, 12, 9, 5, 3, 2]
```
## sorted(list)
```
>>> l=[3,5,2,12,9,45]
>>> id(l)
4322848648
>>> L=sorted(l)
>>> L
[2, 3, 5, 9, 12, 45]
>>> id(L)
4322868360
```
此函数会返回一个新的有序列表。

和1中的类似，`sorted()`也可以实现降序
```
>>> sorted(l,reverse=True)
[45, 12, 9, 5, 3, 2]
```

## 分片
```
>>> l[::-1]
[45, 9, 12, 2, 5, 3]
```
# 列表修改

## 增加元素

增加单个元素
```
>>> l=['a','b','c']
>>> l.append('d')
>>> l
['a', 'b', 'c', 'd']
```
增加多个元素
```
>>> l.extend(['e','f'])
>>> l
['a', 'b', 'c', 'd', 'e', 'f']
```
插入元素
```
>>> l=['a','b','c']
>>> l.insert(1,'d')
>>> l
['a', 'd', 'b', 'c']
```

## 删除元素

1.使用 `del ele` 语句来删除
```
>>> l=['a','b','c']
>>> del l[1]
>>> l
['a', 'c']
```
那如果索引超出范围了呢？
```
>>> l=['a','b','c']
>>> del l[5]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list assignment index out of range
```
笨蛋！上面不是说过了列表用索引来取元素不能超出索引范围么……

2.使用列表的 `remove(ele)` 函数来删除
```
>>> l=['a','b','c']
>>> l.remove('c')
>>> l
['a', 'b']
```
那如果要删除的元素不存在呢？
```
>>> l=['a','b','c']
>>> l.remove('d')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: list.remove(x): x not in list
```
那如果边遍历边删除呢？Java 中是不允许的，那 Python 中呢？
```
>>> l=['a','b','c']
>>> for x in l:
...     l.remove(x)
...
>>> l
['b']
```
额，好像有点出乎意外

通过调试发现，执行第一次循环之后，`l` 变成了`['b','c']`。然而，执行第二次循环的时候，`x` 为 `'c'`，而不是为`'b'`，第三次循环没有执行。

第一次循环时，for 循环从 `['a','b','c']` 中去取索引为0的元素`'a'`，并删除；
第二次循环时，for 循环从`['b','c']` 中去取索引为1的元素`'c'`，并删除；
第三次循环时，for 循环从 `['b']` 中去取索引为2的元素，但不存在，会停止for 循环。
总之，每循环一次，会迭代出下一索引的对象，若没有会停止循环。

不要在迭代列表时修改列表！！！
否则会得到奇怪的结果。[详情请看Stackoverflow](https://stackoverflow.com/questions/2896752/removing-item-from-list-during-iteration-whats-wrong-with-this-idiom)。

如果想循环删除，可以这样
```
>>> l=['a','b','c']
>>> for x in l[:]:
...     l.remove(x)
...
>>> l
[]
>>>
```
即，使用列表的拷贝来做迭代。


3.使用列表的 `pop(index)` 函数来删除元素，同时获得删除的元素
```
>>> l=['a','b','c']
>>> l.pop(1)
'b'
>>> l
['a', 'c']
```
上面是删除指定索引的元素，当不指定时，会删除最后一个元素
```
>>> l=['a','b','c']
>>> l.pop()
'c'
>>> l
['a', 'b']
```

## 修改元素

```
>>> l=['a','b','c']
>>> l[0]='d'
>>> l
['d', 'b', 'c']
```

# 列表推导式

列表推导式（List Comprehension）或者叫列表解析式，可以在 `[]` 添加一个推导语句来返回一个列表。

使用 for-in 循环来生成数组
```
>>> l=[x for x in range(10)]
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
同时，for-in 循环还可以配上条件语句
```
>>> l=[x for x in range(10) if x>3]
>>> l
[4, 5, 6, 7, 8, 9]
```
此外，也支持多个 for-in 循环
```
>>> l=[(x,y,z) for x in [1,2,3] for y in ['a','b'] for z in ['@','#']]
>>> l
[(1, 'a', '@'), (1, 'a', '#'), (1, 'b', '@'), (1, 'b', '#'), (2, 'a', '@'), (2, 'a', '#'), (2, 'b', '@'), (2, 'b', '#'), (3, 'a', '@'), (3, 'a', '#'), (3, 'b', '@'), (3, 'b', '#')]
```
多层 for-in 循环也可以，但是这里要注意两个 for-in 循环的前后位置
```
>>> l=[x for y in [[1,4,6],[2,3,9]] for x in y]
>>> l
[1, 4, 6, 2, 3, 9]
```
其等价于
```
>>> l=[]
>>> for y in [[1,4,6],[2,3,9]]:
...     for x in y:
...             l.append(x)
...
>>> l
[1, 4, 6, 2, 3, 9]
```

# 生成器表达式

生成器表达式（Generator expressio）可以理解为一个算法，通过这个算法可以以小的性能代价来迭代推演。在大数据量迭代处理时比较高效。
```
>>> g=(x**2 for x in range(20))
>>> for i in g:
...     print(i)
...
0
1
4
9
16
25
36
49
64
81
100
121
144
169
196
225
256
289
324
361
```

# 小结

有几个性质可以概括描述 Python 中 list 对象：

1.是一个序列（sequence），即有序元素集合。

序列代表着它可以使用索引、分片，以及一些算术操作。类似的，同为序列的string、tuple也可以使用这些通用的操作。

2.是一个可变对象（mutable）

可变意味着它可以改变自己的内容，但内存地址并不变，如 dict 也是。

与mutable对立的是immutable，如int、float、tuple、str、bytes，它们被赋值之后，值不可以被修改。

有人会说，str 可以被修改呀！你看，下面的字符串的值被修改了呀！
```
>>> s='uphie'
>>> s+='.studio'
>>> s
'uphie.studio'
```
深究一下会发现不是的
```
>>> s='uphie'
>>> id(s)
4330018440
>>> s+='.studio'
>>> s
'uphie.studio'
>>> id(s)
4330016304
```
其实，这样做是改变了s的引用地址，而`'uphie.studio'`就占用了这块内存地址。

3.存储对象的索引

这意味着如果元素在其它地方被修改，那么list也会被修改，如
```
>>> a=[20]
>>> l=[a,4,5]
>>> l
[[20], 4, 5]
>>> a.append(10)
>>> l
[[20, 10], 4, 5]
```
所以编码的时候，要留意下。

4.可迭代（iterable）

可迭代意味着它可以使用 for-in 循环，元素可以被依次迭代出来，其内部是使用了`__iter__()` 或 `__getitem__()` 方法。

判断一个对象是不是iterable，可以使用下面的方法进行判断。

```
>>> from collections import Iterable
>>> isinstance('abc',Iterable)
True
>>> isinstance({1,2,3},Iterable)
True
>>> isinstance((1,2,3),Iterable)
True
>>> isinstance([1,2,3],Iterable)
True
>>> isinstance(123,Iterable)
False
```
可迭代的对象有：list、tuple、dict、set、str、range等。

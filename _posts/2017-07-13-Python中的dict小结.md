---
title: Python中的dict小结
author: Uphie
date: 2017-07-13 18:30:00 +0800
categories: [技术]
tags: [python]
math: true
toc: true
---

dict 类似于Java中的HashMap，提供通过 key 来查找 value 的功能。

# 字典操作

## 读取

dict 读取元素类似于 list 通过索引来读取元素，不同的是 dict 通过 key 来读取
```
>>> d={'name':'Bob'}
>>> d['name']
'Bob'
```
但是，如果 key 不存在呢？
```
>>> d['name']
'Bob'
>>> d['age']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'age'
```
额，可以换个更优雅的方法
```
>>> d.get('age')
>>>
```
没出错，但是返回了None，我们可以指定如果不存在就返回一个默认值
```
>>> d.get('age',20)
20
```
## 增加元素

```
>>> d={}
>>> d['name']='Bob'
>>> d
{'name': 'Bob'}
```
但是，如果要增加的 key 已经存在，那么原先的 key-value 就会被覆盖
```
>>> d={'name':'Bob'}
>>> d['name']='Julia'
>>> d
{'name': 'Julia'}
```
有一种读值和赋值结合在一起的方法
```
>>> d={'name':'Bob'}
>>> d.setdefault('name','Andy')
'Bob'
```
由于 key 已经存在了，那么会返回 key 对应的 value 来。那么如果 key 不存在呢？
```
>>> d={'name':'Bob'}
>>> d.setdefault('age',20)
20
>>> d
{'name': 'Julia', 'age': 20}
```

## 删除元素

和 list 类似，python 也为 dict 提供了 `pop` 、`del` 来删除元素。

`del dict[key]` 语句
```
>>> d={'name':'Bob','age':21,'gender':'M'}
>>> del d['age']
>>> d
{'name': 'Bob', 'gender': 'M'}
```

`dict.pop(key)` 方法可以删除并返回删除的 value
```
>>> d={'name':'Bob','age':21,'gender':'M'}
>>> d.pop('age')
21
>>> d
{'name': 'Bob', 'gender': 'M'}
```
如果 key 不存在呢？
```
>>> d={'name':'Bob','age':21,'gender':'M'}
>>> d.pop('height')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'height'
```
出错了，可以设置一个默认值来解决
```
>>> d={'name':'Bob','age':21,'gender':'M'}
>>> d.pop('height',176)
176
>>> d
{'name': 'Bob', 'gender': 'M', 'age': 21}
```
除了可以删除单个元素，还可以清空所有元素
```
>>> d={'name':'Bob','age':21,'gender':'M'}
>>> d.clear()
>>> d
{}

```
## 修改元素

这个已经在上面提过了啊
```
>>> d={'name':'Bob'}
>>> d['name']='Julia'
>>> d
{'name': 'Julia'}
```

## 合并

如果想把两个 dict 合并怎么办？

根据上面学到的，你可能想到了下面这个方法……
```
>>> s={'name':'Bob','age':21,'gender':'M'}
>>> h={'chinese':90,'math':79}
>>> d={}
>>> for k1,v1 in s.items():
...     d[k1]=v1
...     for k2,v2 in h.items():
...             d[k2]=v2
...
>>> d
{'name': 'Bob', 'age': 21, 'chinese': 90, 'gender': 'M', 'math': 79}
```
嗯可以，只是不是那么 Pythonic …，可以这样：
```
>>> s={'name':'Bob','age':21,'gender':'M'}
>>> h={'chinese':90,'math':79}
>>> s.update(h)
>>> s
{'name': 'Bob', 'math': 79, 'chinese': 90, 'gender': 'M', 'age': 21}
```
当然，合并的时候如果有重复的 key，那么后面的 key-value 会覆盖前面的 key-value。

对了，合并出来的新的 dict 还是原来的 dict 么？
```
>>> s={'name':'Bob','age':21,'gender':'M'}
>>> id(s)
4300911048
>>> h={'chinese':90,'math':79}
>>> s.update(h)
>>> id(s)
4300911048
```
答案是肯定的，同时也证明dict是可变对象（mutable）。

## 类型转换

序列（str、list、tuple、set等） 转成 dict
```
>>> l=['google','yahoo','apple','amazon']
>>> d=dict.fromkeys(l,1)
>>> d
{'amazon': 1, 'google': 1, 'yahoo': 1, 'apple': 1}
```
序列中的元素会转换为 dict 中的 key，`dict.fromkeys(l,1)` 中的1表示填充的 value 值。由于 dict 中 key 不可变，所以**序列中的元素必须不可变**。否则会出现下面这种情况
```
>>> l=[[1,2],4,5]
>>> d=dict.fromkeys(l,'s')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'list'
```
# 操作符运算

## in
判断 key在 dict 中是否存在
```
>>> d={'class':'math','marks':90,'level':'a'}
>>> 'class' in d
True
>>> 'rank' in d
False
```

# 迭代

dict 和 list 一样可以使用 for 循环

## 迭代 key-value

它每次迭代出来的是一个 (key,value) 的元组（tuple）

```
>>> d={'a':'apple','b':'banana','c':'cabbage'}
>>> for k,v in d.items():
...     print('{0}-->{1}'.format(k,v))
...
b-->banana
a-->apple
c-->cabbage
```
## 迭代 key
```
>>> d={'a':'apple','b':'banana','c':'cabbage'}
>>> for k in d:
...     print(k)
...
b
a
c
```
效果等同于
```
>>> for k in d.keys():
...     print(k)
...
b
a
c
```
## 迭代 value
```
>>> for v in d.values():
...     print(v)
...
banana
apple
cabbage
```

# 字典推导式

除了 list 可以使用推导式以外，dict 也可以。
```
>>> d={k:k*2 for k in ['a','b','c'] }
>>> d
{'b': 'bb', 'a': 'aa', 'c': 'cc'}
```

除了可以用推导式生成字典之外，还有一些其它妙用

```
>>> d={'a':'apple','b':'banana','c':'cabbage'}
>>> d={v:k for k,v in d.items()}
>>> d
{'banana': 'b', 'apple': 'a', 'cabbage': 'c'}
```
这样的话，就把 dict 中的 key 和 value 对调了。但是注意一点，dict 中的 key 是不重复的，value 是可以重复的，如果有 value 相同那么在对调 key 和 value 的时候，就会导致有的 key 被覆盖
```
>>> d={'friend':'Jobs','brother':'Andy','teacher':'Jobs'}
>>> d={v:k for k,v in d.items()}
>>> d
{'Jobs': 'teacher', 'Andy': 'brother'}
```
将两个列表合并为一个dict
```
>>> l=['a','b','c']
>>> L=['apple','banna','cabbage']
>>> d={k:v for k,v in zip(l,L)}
>>> d
{'b': 'banna', 'a': 'apple', 'c': 'cabbage'}
```

# 小结

1.dict 是一个无序集合

dict 作为无序集合意味着它不能像 list 那样通过索引来查找元素，但通过 key 来查找元素。

dict 占用的空间比较大，list占用空间比较小。

```
>>> l=[x for x in range(100)]
>>> sys.getsizeof(l)
912
>>> d={x:x for x in range(100)}
>>> sys.getsizeof(d)
6240
```

dict 查找元素速度很快，不管数据量多大，但 list 查找速度在数据量很大的时候就会很慢。原因就是 dict 是通过 [哈希表](https://zh.m.wikipedia.org/wiki/%E5%93%88%E5%B8%8C%E8%A1%A8)（请自备梯子）的原理实现的。通过key计算出唯一的一个哈希值（实际上有可能会重复），这个哈希值决定了值的地址，这样就能很快地通过 key 找到与之对应的 value。因此 dict 对 key 的要求就是不重复，而且不可更改，所以 dict 中 `str`、`int`、`float`都可以做key。

```
>>> d={1:3,3.0:'a','t':'T',(2,3,4):9}
>>> d
{'t': 'T', 1: 3, (2, 3, 4): 9, 3.0: 'a'}
```

这篇 [博文](https://foofish.net/python_dict_implements.html) 有很详尽的讲解。

但是看下这个
```
>>> d={4:'I am 4',4.0:'I am 4.0'}
>>> d
{4: 'I am 4.0'}
```
![问号脸](https://img.uphie.studio/wenhaolian.jpeg)

再看下
```
>>> d[4]
'I am 4.0'
>>> d[4.0]
'I am 4.0'
```
原因是键被隐式的转换了
```
>>> hash(4)
4
>>> hash(4.0)
4
>>> hash(4.1)
230584300921368580
```

2.dict 可以被迭代（iterable）

Python2 和 Python3 中的迭代有些区别

- Python2
```
>>> import string
>>> d={x:x*3 for x in string.ascii_letters}
>>> d
{'A': 'AAA', 'C': 'CCC', 'B': 'BBB', 'E': 'EEE', 'D': 'DDD', 'G': 'GGG', 'F': 'FFF', 'I': 'III', 'H': 'HHH', 'K': 'KKK', 'J': 'JJJ', 'M': 'MMM', 'L': 'LLL', 'O': 'OOO', 'N': 'NNN', 'Q': 'QQQ', 'P': 'PPP', 'S': 'SSS', 'R': 'RRR', 'U': 'UUU', 'T': 'TTT', 'W': 'WWW', 'V': 'VVV', 'Y': 'YYY', 'X': 'XXX', 'Z': 'ZZZ', 'a': 'aaa', 'c': 'ccc', 'b': 'bbb', 'e': 'eee', 'd': 'ddd', 'g': 'ggg', 'f': 'fff', 'i': 'iii', 'h': 'hhh', 'k': 'kkk', 'j': 'jjj', 'm': 'mmm', 'l': 'lll', 'o': 'ooo', 'n': 'nnn', 'q': 'qqq', 'p': 'ppp', 's': 'sss', 'r': 'rrr', 'u': 'uuu', 't': 'ttt', 'w': 'www', 'v': 'vvv', 'y': 'yyy', 'x': 'xxx', 'z': 'zzz'}
>>> type(d.items())
<type 'list'>
>>> d.items()[0]
('A', 'AAA')
```
我们看到 dict 在 `for x in d.items()` 迭代的时候，其实是在迭代 key-value 组成的 tuple 的 list。或许你没发现什么问题，继续看
```
>>> import sys
>>> sys.getsizeof(d.items())
488
>>> letters=[''.join((x,y,z)) for x in string.ascii_letters for y in string.ascii_letters for z in string.ascii_letters]
>>> D={x:x*3 for x in letters}
>>> sys.getsizeof(D.items())
1124936
```
可以看到，遍历 dict 时所占用的内存随着 dict 的长度增大而增大。

- Python3
```
>>> import string
>>> d={x:x*3 for x in string.ascii_letters}
>>> d
{'d': 'ddd', 'O': 'OOO', 'Y': 'YYY', 'v': 'vvv', 'h': 'hhh', 'D': 'DDD', 'S': 'SSS', 's': 'sss', 'R': 'RRR', 'K': 'KKK', 'r': 'rrr', 'I': 'III', 'j': 'jjj', 'M': 'MMM', 'x': 'xxx', 'i': 'iii', 'q': 'qqq', 'N': 'NNN', 'X': 'XXX', 'T': 'TTT', 'l': 'lll', 'U': 'UUU', 'J': 'JJJ', 't': 'ttt', 'F': 'FFF', 'C': 'CCC', 'f': 'fff', 'z': 'zzz', 'y': 'yyy', 'A': 'AAA', 'a': 'aaa', 'V': 'VVV', 'b': 'bbb', 'k': 'kkk', 'H': 'HHH', 'u': 'uuu', 'n': 'nnn', 'm': 'mmm', 'p': 'ppp', 'P': 'PPP', 'o': 'ooo', 'Q': 'QQQ', 'c': 'ccc', 'B': 'BBB', 'w': 'www', 'Z': 'ZZZ', 'W': 'WWW', 'e': 'eee', 'g': 'ggg', 'G': 'GGG', 'E': 'EEE', 'L': 'LLL'}
>>> type(d.items())
<class 'dict_items'>
>>> import sys
>>> sys.getsizeof(d.items())
48
>>> letters=[''.join((x,y,z)) for x in string.ascii_letters for y in string.ascii_letters for z in string.ascii_letters]
>>> D={x:x*3 for x in letters}
>>> sys.getsizeof(D.items())
48
```
可以看到 Python3 中的 dict 在 `for x in d.items()` 遍历中使用的 `dict_items` 非常省内存，不管 dict 的长度大小。原因就是 Python3 中遍历 dict 使用了生成器，遍历时每次生成一个值，大大提高了效率和硬件资源利用率。

3.dict 是可变的（mutable）

刚刚说合并 dict 的时候已经说过啦

4.存储的 value 是对象的索引
```
>>> l=[1,2,3]
>>> d={'v':l}
>>> d
{'v': [1, 2, 3]}
>>> l.append(4)
>>> d
{'v': [1, 2, 3, 4]}
```

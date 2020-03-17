在列表、字典中根据条件筛选数据

```python
from random import randint
#列表
l = [randint(-10, 10) for _ in range(10)]
l2 = [x for x in l if x >= 0]
l3 = list(filter(lambda x: x>=0, l)) #filter返回迭代器对象
print(l)
print(l2)
print(l3)
#字典
s = {'student%d' %i : randint(50, 100) for i in range(1, 21)}
s2 = {k:v for k, v in s.items() if v >= 90}
s3 = dict(filter(lambda item:item[1] >= 90, s.items()))
print(s)
print(s2)
print(s3)
```

为元组元素命名，提高程序可读性

```python
#提供枚举命名空间
from enum import IntEnum
class StudentEnum(IntEnum):
    NAME = 0
    AGE = 1 
    SEX = 2
    EMAIL = 3
s = ('Jim', 16, 'male', 'jim@gmail.com')
print(s[StudentEnum.NAME])
#引入命名元组
from collections import namedtuple
Student = namedtuple('Student',['name','age','sex','email'])
s2 = Student('Jim',18,'male','jim@gmail.com')
print(isinstance(s2, tuple))
print(s2[0])
print(s2.name)
```

根据字典中值的大小，对字典中的项排序

```python
from random import randint
d = {k: randint(60, 100) for k in 'abcdefg'}
# 转化为元组排序
l = [(v, k) for k, v in d.items()]
l2 = list(zip(d.values(), d.keys()))
print(sorted(l, reverse = True))
print(sorted(l2, reverse = True))
# 指定字段排序
p = sorted(d.items(), key=lambda item: item[1], reverse = True)
print(p)
# 添加排名
for i, (k, v) in enumerate(p, 1):
    d[k] = (i, v)
print(d)    
print({k:(i, v) for i, (k, v) in enumerate(p, 1)})
```

统计序列中元素的频数，并查看频数最高的前n个

```python
from random import randint
from collections import Counter
data = [randint(0, 20) for _ in range(30)]
c = Counter(data)
print(c.most_common(3))
```

快速查找多个字典的公共键

```python
from random import randint, sample
from functools import reduce
d1 = {k:randint(1, 4) for k in sample('abcdefgh', randint(3, 6))}
d2 = {k:randint(1, 4) for k in sample('abcdefgh', randint(3, 6))}
d3 = {k:randint(1, 4) for k in sample('abcdefgh', randint(3, 6))}
print(d1)
print(d2)
print(d3)
dl = [d1, d2, d3]
print(reduce(lambda a, b: a & b, map(dict.keys, dl)))
```

让字典有序

```python
from collections import OrderedDict
from random import shuffle
from itertools import islice
players = ['a','b','c','d','e','f','g','h']
shuffle(players)
od = OrderedDict()
for i, p in enumerate(players, 1):
    od[p] = i
print(od)
def query_by_name(d, name):
    return d[name]
def query_by_order(d, begin, end = None):
    begin -= 1
    if end is None:
        end = begin + 1
    return list(islice(d, begin, end))
print(query_by_order(od, 3, 5))
```

实现历史记录功能

```python
from collections import deque
q = deque([], 5) //双端队列，最多存5个
//存储历史记录到磁盘
import pickle
pickle.dump(q, open('save.pkl', 'wb'))
q2 = pickle.load(open('save.pkl', 'rb'))
```

拆分有多种分割符的字符串

```python
s = 'ab;cd|efg|hi,jkl|mn\topq;rst,uvw\txyz'
#法1:
def my_split(s, seps):
    res = [s]
    for sep in seps:
        t = []
        list(map(lambda ss: t.extend(ss.split(sep)), res))
        res = t
    return res    	
#法2:
from functools import reduce
print(reduce(lambda l, sep: sum(map(lambda ss: ss.split(sep), l), []), ',;|\t', [s]))
```

拼接字符串

```python
l = ['abc','def','ghl','hij']
print(''.join(l))
```

字符串对齐

```python
#法1:
s = 'abc'
s.ljust(10, '*') 
s.rjust(10, '*') 
s.center(10, '*')
#法2:
format(s, '<10')
format(s, '>10')
format(s, '^10')
format(s, '*^10')
format(5, '*^10')
```

去掉字符串中不需要的字符

```python
# strip()去两端空白字符
# translate()接受一个字典
s = 'abc1234xyz'
s.translate(s.maketrans('abcxyz', 'XYZABC'))
s.translate({ord('a': None)}) #删除
# re去除空白符号
import re
re.sub('[ \t\n]+','',s)
re.sub('\s+','',s)
```

自定义可迭代对象并且打印素数

```python
from collections import Iterable
class PrimeNumber(Iterable):
    def __init__(self, a, b):
        self.a = a
        self.b = b
    def __iter__(self):
        for k in range(self.a, self.b + 1):
            if self.is_prime(k):
                yield k
    def is_prime(self, k):
        return False if k < 2 else all(map(lambda x: k % x, range(2, k)))

pn = PrimeNumber(1, 30)
for  n in pn:
    print(n)
#反向迭代：实现__reversed__
```

对可迭代对象切片

```python
def my_islice(iterable, start, end, step=1):
  tmp = 0
  for i, x in enumerate(iterable):
    if i >= end:
      break
    if i >= start:
      if temp == 0:
        temp = step
      	yield x
      temp -= 1
print(list(my_islice(range(100, 150), 10, 20)))
from itertools import islice
print(list(islice(range(100, 150), 10, 20)))
```

多个可迭代对象

```python
#并行：某班学生期末考试成绩语文、数学、英语存在3个列表中，计算学生总分
#串行：某年级三个班，英语成绩在3个列表中，统计全年级成绩高于90分人数
#并行
from random import randint
chinese = [randint(60, 100) for _ in range(20)]
math = [randint(60, 100) for _ in range(20)]
english = [randint(60, 100) for _ in range(20)]
[sum(s) for s in zip(chinese, math, english)]
list(map(lambda s1, s2, s3: s1 + s2 + s3, chinese, math, english))
#串行
c1 = [randint(60, 100) for _ in range(20)]
c2 = [randint(60, 100) for _ in range(25)]
c3 = [randint(60, 100) for _ in range(27)]
len([x for x in chain(c1, c2, c3) if x > 90])
```

文件读取

```python
s = "test"
f = open('test.txt', 'wt', encoding="gbk") #默认为utf8
f.write(s)
f.flush()
f = open('test.txt', encoding="gbk")
f.read()
```


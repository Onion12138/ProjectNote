python数据分析

## Numpy

创建数组和矩阵

```python
import numpy as np
arr = np.array([i for i in range(10)])
arr.dtype
np.zeros(10, dtype=int) #默认为float
np.zeros((3, 5)) #3行5列
np.zeros(shape=(3, 5))
np.ones((3, 5))
np.full((3, 5), fill_value=666) #fill_value可以省略
np.arange(0, 20, 2) #默认步长为1
np.arange(0, 1, 0.2)
np.arange(10) #默认起始点为0
np.linspace(0, 20, 10) #[0,20]区间里等长选取10个点
np.random.randint(0, 10) #1个数，[0,10)
np.random.randint(0, 10, 10) #生成10个数的向量
np.random.randint(4, 8, size=10)
np.random.randint(4, 8, size=(3, 5))
np.random.seed(666) #指定随机种子
np.random.random() #[0,1)间的浮点数
np.random.random(10) # 10个[0,1)间的浮点数
np.random.random((3, 5))
np.random.normal() #符合标准正态分布的随机数
np.random.normal(10, 100) #均值为10，方差为100的随机数
np.random.normal(0, 1, (3, 5)) #3行5列均值为0，方差为1的随机数
```

基本操作

```python
x = np.arange(10)
X = np.arange(15).reshape(3, 5) #reshape不改变原数组
x.ndim #查看维度
x.shape #(10,)一维
X.shape #(3, 5)二维
x.size
X[0,0]#访问(0，0)位置的元素
X[:5] #[0,1,2,3,4]
X[::2] #[0,2,4,6,8]
X[::-1] #[9,8,7,6,5,4,3,2,1,0]
X[:2, :3] #前两行前三列 [[0,1,2],[5,6,7]]
X[:2][:3] #[[0,1,2,3,4],[5,6,7,8,9]]
X[:2,::2] #[[0,2,4],[5,7,9]]
X[::-1,::-1] #[[14,13,12,11,10],[9,8,7,6,5],[4,3,2,1,0]]
X[0,:] #第一行[0,1,2,3,4]
X[:,0] #第一列[0,5,10]
subX = X[:2, :3].copy() #若不掉用copy，修改数据会相互影响
B = x.reshape(1, 10) # B为2维矩阵，(1, 10),和x不同
C = x.reshape(10, -1) #只关心10行，前提能整除
D = x.reshape(-1, 10) #只关心10列
```

合并

```python
x = np.array([1,2,3])
y = np.array([3,2,1])
np.concatenate([x,y]) #[1,2,3,3,2,1]
z = np.array([666,666,666])
a = np.array([[1,2,3],[4,5,6]])
np.concatenate([a,a]) #[[1,2,3],[4,5,6],[1,2,3],[4,5,6]]
np.concatenate([a,a],axis=1) #[[1,2,3,1,2,3],[4,5,6,4,5,6]]
np.concatenate([a,z.reshape(1,-1)]) #必须维数相同 [[1,2,3],[4,5,6],[666,666,666]]
np.vstack([a,z]) #竖直方向[[1,2,3],[4,5,6],[666,666,666]]
d = np.full((2,2),100)
np.hstack([a,d]) #水平方向[[1,2,3,100,100],[4,5,6,100,100]]
```

分割

```python
x = np.arange(10)
x1, x2, x3 = np.split(x,[3,7]) #x1=[0,1,2],x2=[3,4,5,6],x3=[7,8,9]
np.split(x,[5])#[0,1,2,3,4],[5,6,7,8,9]
a = np.arange(16).reshape(4,4)
np.split(a,[2]) #[[0,1,2,3],[4,5,6,7]] & [[8,9,10,11],[12,13,14,15]]
np.split(a,[2],axis=1) # [[0,1],[4,5],[8,9],[12,13]] & [[2,3],[6,7],[10,11],[14,15]]
np.vsplit(a,[2]) #[[0,1,2,3],[4,5,6,7]] & [[8,9,10,11],[12,13,14,15]]
np.hsplit(a,[2]) #[[0,1],[4,5],[8,9],[12,13]] & [[2,3],[6,7],[10,11],[14,15]]
np.hsplit(a,[-1])#[[0,1,2],[4,5,6],[8,9,10],[12,13,14]] & [[3],[7],[11],[15]]
```

运算

```python
x = np.arange(1,16).reshape(3,5)
x/2 #浮点除法
x//2 #整数除法
1/x 
np.abs(x)
np.sin(x)
np.exp(x)
np.pow(3,x) #3 ** x
np.log2(x) #log10(x),log(x)
a = np.arange(4).reshape(2,2)
b = np.full((2,2),10)
a * b #对应元素乘除
a.dot(b)
a.T 
v = np.array([1,2])
v + a #[[1,3],[3,5]] 向量与矩阵做加法，和矩阵每个向量相加
np.tile(v, (2,1)) #行方向堆叠2次，列方向堆叠1次 [[1,2],[1,2]]
a.dot(v) #自动判断相乘
np.linalg.inv(a) #求逆矩阵
np.linalg.pinv(a) #求伪逆矩阵

```

聚合

```python
l = np.random.random(100)
np.sum(l)
np.min(l)
l.min()
l.sum()
x = np.arange(16).reshape(4,-1)
np.sum(x,axis=0) #求每列的和
np.sum(x,axis=1) #求每行的和
np.prod(x) #每个元素的乘积
np.mean(x)
np.median(x) 
np.percentaile(x,q=50)#等价于中位数 q=100等价为最大值
np.var(x)
np.std(x)
```

索引

```python
x = np.random.normal(0,1,size=10000)
np.argmin(x) #最小值的索引
x = np.arange(16)
np.random.shuffle(x)
np.sort(x) #x本身未被修改
x.sort() #x本身被修改
y = np.random.randint(10,size=(4,4))
np.sort(y) #每一行被排序，axis默认为1
np.sort(y, axis=0) #每一列被排序
np.argsort(x) #索引排序
np.partition(x,0.5) #类似快速排序partition，左小右大
np.argpartition(x,0.5)

```

Fancy Indexing

```python
x = np.arange(16)
ind = [3,5,8]
x[ind]
x = x.reshape(4,-1)
row = np.array([0,1,2])
col = np.array([1,2,3])
x[row,col]
x[:2,col]
np.sum(x<=3) # 4
np.sum(x%2==0,axis=1) #[2,2,2,2] 每一行偶数的个数
np.sum((x>3)&(x<10)) # &
np.sum(~(x==0)) #取反
np.count_nonzero(x<=3) #4
np.any(x==0) # true np.any(x<0):false
np.all(x>=0) # true
x[x<5]
```

与线性代数相关

```python
from numpy as np
np.linalg.norm(vec) #求模长
vec/np.linalg.norm(vec) #规范化，numpy未封装
np.identity(2) #单位矩阵
np.linalg.inv(vec) #获取逆矩阵
```



## Matplotlib

```python
import matplotlib.pyplot as plt
x = np.linspace(0,10,100)
y = np.sin(x)
plt.plot(x,y)
plt.show()

cosy = np.cos(x)
siny = np.sin(x)
plt.plot(x,siny) #可以自定义颜色color=red 自定义风格：linestyle="--"
plt.plot(x,cosy) #自定义label：label="sin(x)"
plt.xlim(-5,15) #定义左右范围
plt.axis([-1,11,-2,2]) #x范围[-1,11],y范围[-2,2]
plt.xlabel("x axis")
plt.show()
plt.title("title")

plt.scatter(x,siny) #散点图
plt.show()
```

```python
#实战
from sklearn import datasets
iris = datasets.load_iris()
iris.keys()
print(iris.DESCR)
x = iris.data[:,:2] #看前两个维度
y = iris.target
plt.scatter(X[y==0,0],X[y==0,1],color="red",marker="o")
plt.scatter(X[y==1,0],X[y==1,1],color="blue",marker="+")
plt.scatter(X[y==2,0],X[y==2,1],color="green",markder="x")
plt.show
```


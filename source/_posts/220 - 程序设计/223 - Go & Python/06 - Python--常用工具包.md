---
title: Python--常用工具包
date: 2023-11-16 16:27:41
tags:
  - 程序设计
  - Python
categories:
  - Go & Python
published: false
---

# 1 Pandas 数据处理

## 1.1 Pandas 数据结构 - Series
Pandas Series 类似表格中的一个列（column），类似于一维数组，可以保存任何数据类型。

Series 由索引（index）和列组成，函数如下：

```python
pandas.Series( data, index, dtype, name, copy)
```

参数说明：
- **data**：一组数据(ndarray 类型)。
- **index**：数据索引标签，如果不指定，默认从 0 开始。
- **dtype**：数据类型，默认会自己判断。
- **name**：设置名称。
- **copy**：拷贝数据，默认为 False。

**示例：**

```python
# 从列表创建Series  
pd.Series(['a', 'b', 'c'])  
# 0    a  
# 1    b  
# 2    c  
# dtype: object  
# 自动创建索引  
  
# 通过字典创建带索引的Series  
s1 = pd.Series({'a':11, 'b':22, 'c':33})  
# a    11  
# b    22  
# c    33  
  
# 通过关键字创建带索引的Series  
s2 = pd.Series([11, 22, 33], index = ['a', 'b', 'c'])  
print(s1)  
# a    11  
# b    22  
# c    33  
print(s2['a']) # 根据索引取值 11# 获取全部索引  
print(s2.index) # Index(['a', 'b', 'c'], dtype='object')  
# 获取全部值  
print(s2.values) # [11 22 33]  
# 类型  
print(type(s1.values))    # <class 'numpy.ndarray'>  
print(type(np.array(['a', 'b'])))  # Pandas的数据结构是基于numpy  
# 转换为列表  
s2.values.tolist() # [11, 22, 33]
```

使用map函数，处理Series中的值，生成新的Series：

```python
# 取出email  
emails = pd.Series(['abc at amazom.com', 'admin1@163.com', 'mat@m.at', 'ab@abc.com'])  
import re  
pattern ='[A-Za-z0-9._]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,5}'  
mask = emails.map(lambda x: bool(re.match(pattern, x))) # 遍历emails中的values  
print(mask)  
# 0    False  
# 1     True  
# 2     True  
# 3     True  
print(emails[mask])  
# 1    admin1@163.com  
# 2          mat@m.at  
# 3        ab@abc.com
```

Pandas通过index可以提升查询性能：
1. 如果index唯一，pandas会使用哈希表优化，查询性能为O(1)
2. 如果index有序不唯一，pandas会使用二分查找算法，查询性能为O(logN)
3. 如果index完全随机，每次查询都要扫全表，查询性能为O(N)

## 1.2 Pandas 数据结构 - DataFrame
DataFrame 是一个表格型的数据结构，它含有一组有序的列，每列可以是不同的值类型（数值、字符串、布尔型值）。DataFrame 既有行索引也有列索引，它可以被看做由 Series 组成的字典（共同用一个索引）。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231110102650.png)
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231110102712.png)

```python
pandas.DataFrame( data, index, columns, dtype, copy)
```
参数说明：
- **data**：一组数据(ndarray、series, map, lists, dict 等类型)。
- **index**：索引值，或者可以称为行索引。
- **columns**：列索引，默认为 RangeIndex (0, 1, 2, …, n) 。
- **dtype**：数据类型。
- **copy**：拷贝数据，默认为 False。

**示例：**

```python
import pandas as pd  
  
# 列表创建dataframe，与Series不同的是有了列索引
df1 = pd.DataFrame(['a', 'b', 'c', 'd'])  
#    0  
# 0  a  
# 1  b  
# 2  c  
# 3  d

# 嵌套列表创建dataframe  
df2 = pd.DataFrame([  
                     ['a', 'b'],   
                     ['c', 'd']  
                    ])  
# 自定义列索引  
df2.columns= ['one', 'two']  
# 自定义行索引  
df2.index = ['first', 'second']  
print(df2)  
#        one two  
# first    a   b  
# second   c   d  
# 可以在创建时直接指定 DataFrame([...] , columns='...', index='...' )# 查看索引  
print(df2.columns) # Index(['one', 'two'], dtype='object')  
print(df2.index) # Index(['first', 'second'], dtype='object')  
print(type(df2.values)) # <class 'numpy.ndarray'>  
print(df2.values.tolist()) # [['a', 'b'], ['c', 'd']]
```


## 1.3 数据导入
Pandas支持大量格式的导入，使用的是`read_*()` 的形式，如：

```python
import pandas as pd  
# pip install xlrd # excel文件需要  
# 导入excel文件  
excel1 = pd.read_excel(r'1.xlsx')  
print(excel1)  
# 0     1    4.0     5     7  
# 1    33    NaN    55    66  
# 2   111  222.0   333   444  
# 显示前几行  
print(excel1.head(2))  
#    num1  num2  num3  num4  
# 0     1   4.0     5     7  
# 1    33   NaN    55    66  
  
# 行列数量  
print(excel1.shape) # (3, 4)  
  
# 详细信息  
# excel1.info()  
print(excel1.describe())  
  
# 指定导入哪个Sheet  
pd.read_excel(r'1.xlsx',sheet_name = 0)  
  
# 支持其他常见类型  
pd.read_csv(r'book_utf8.csv',sep=',', nrows=10, encoding='utf-8')  
  
pd.read_table( r'file.txt' , sep = ' ')  
  
import pymysql  
sql  =  'SELECT *  FROM mytable'  
conn = pymysql.connect('ip','name','pass','dbname','charset=utf8')  
# 从数据库中读取  
df = pd.read_sql(sql,conn)
```

1. `pd.read_excel(r'1.xlsx',sheet_name = 0)` 导入excel，并且指定sheet
2. `pd.read_sql(sql,conn)` 导入数据库


## 1.4 数据预处理
1. **处理缺失值**
    
    - `isnull()`: 返回布尔值DataFrame，指示每个元素是否为缺失值。示例: `df.isnull()`
    - `fillna(value)`: 用指定值填充缺失值。示例: `df.fillna(0)`
    - `dropna()`: 删除包含缺失值的行或列。示例: `df.dropna()`
2. **处理重复值**
    
    - `duplicated()`: 返回布尔Series，指示是否为重复行。示例: `df.duplicated()`
    - `drop_duplicates()`: 删除DataFrame中的重复行。示例: `df.drop_duplicates()`
3. **处理异常值**
    
    - 标识或替换超出范围的值为缺失值:
        - `df[(df < low) | (df > high)] = np.nan`
        - `df.replace(value_to_replace, new_value)`

**示例：**

```python
import pandas as pd  
import numpy as np  
  
# 空值的定义使用了 numpy的nan，也可以使用python的None  
x = pd.Series([ 1, 2, np.nan, 3, 4, 5, 6, None, 8])  
# 0    1.0  
# 1    2.0  
# 2    NaN  
# 3    3.0  
# 4    4.0  
# 5    5.0  
# 6    6.0  
# 7    NaN  
# 8    8.0  
  
#检验序列中是否存在缺失值  
print(x.hasnans) # True  
print(x.isnull())  
# 0    False  
# 1    False  
# 2     True  
# 3    False  
# 4    False  
# 5    False  
# 6    False  
# 7     True  
# 8    False  
  
# 将缺失值填充为平均值  
x.fillna(value = x.mean())  
  
# 前向填充缺失值  
df3=pd.DataFrame({"A":[5,3,None,4],   
                 "B":[None,2,4,3],   
                 "C":[4,3,8,5],   
                 "D":[5,4,2,None]})   
                   
df3.isnull().sum() # 查看缺失值汇总  
df3.ffill() # 用上一行填充  
df3.ffill(axis=1)  # 用前一列填充  
  
# 缺失值删除  
df3.info()  
df3.dropna()  
  
# 填充缺失值  
df3.fillna('无')  
  
# 删除DataFrame中的重复行  
df3.drop_duplicates()
```

**DataFrame的数据调整：**

```python
import pandas as pd  
# 行列调整  
df = pd.DataFrame({"A":[5,3,None,4],   
                 "B":[None,2,4,3],   
                 "C":[4,3,8,5],   
                 "D":[5,4,2,None]})   
# 列的选择,多个列要用列表  
print(df[ ['A', 'C'] ])  
#      A  C  
# 0  5.0  4  
# 1  3.0  3  
# 2  NaN  8  
# 3  4.0  5  
  
# 某几列  
print(df.iloc[:, [0,2]]) # :表示所有行，获得第1和第3列  
  
# 行选择  
print(df.loc[ [0, 2] ]) # 选择第1行和第3行  
print(df.loc[ 0:2    ]) # 选择第1行到第3行  
  
# 比较，输出 A 列值小于5，且C列值小于4的行  
print(df[ ( df['A']<5 ) & ( df['C']<4 )   ])
# 行列互换  
print(df.T)
```

## 1.6 计算操作

### 1.6.1 基本操作
```python
import pandas as pd  
df = pd.DataFrame({"A":[5,3,None,4],   
                 "B":[None,2,4,3],   
                 "C":[4,3,8,5],   
                 "D":[5,4,2,None]})  
# 算数运算  
# 两列之间的加减乘除  
print(df['A'] + df['C'])  
  
# 任意一列加/减一个常数值，这一列中的所有值都加/减这个常数值  
print(df['A'] + 5)  
  
# 比较运算  
print(df['A'] > df ['C'])  
  
# count非空值计数  
print(df.count())  
  
# 非空值每列求和  
df.sum()  
df['A'].sum()  
  
# mean求均值  
# max求最大值  
# min求最小值  
# median求中位数  # mode求众数  
# var求方差  
# std求标准差
```

### 1.6.2 聚合计算

```python
import pandas as pd  
import numpy as np  
  
# 聚合  
sales = [{'account': 'Jones LLC','type':'a', 'Jan': 150, 'Feb': 200, 'Mar': 140},  
         {'account': 'Alpha Co','type':'b',  'Jan': 200, 'Feb': 210, 'Mar': 215},  
         {'account': 'Blue Inc','type':'a',  'Jan': 50,  'Feb': 90,  'Mar': 95 }]  
  
df2 = pd.DataFrame(sales)  
print(df2.groupby('type').groups)  
  
for a, b in df2.groupby('type'):  
    print(a)  
    print(b)  
  
# 聚合后再计算  
df2.groupby('type').count()  
# df2.groupby('Jan').sum()  
  
# 各类型产品的销售数量和销售总额  
df2.groupby('type').aggregate( {'type':'count' , 'Feb':'sum' })  
  
group=['x','y','z']  
data=pd.DataFrame({  
    "group":[group[x] for x in np.random.randint(0,len(group),10)] ,  
    "salary":np.random.randint(5,50,10),  
    "age":np.random.randint(15,50,10)  
    })  
#   group  salary  age  
# 0     y      35   39  
# 1     z      20   28  
# 2     z       5   42  
# 3     y      28   34  
# 4     y      37   47  
# 5     y      24   48  
# 6     x      45   32  
# 7     x      47   19  
# 8     z      30   18  
# 9     y      11   31  
  
data.groupby('group').agg('mean') # 根据group进行聚合后在求和  
data.groupby('group').mean().to_dict() # {'salary': {'x': 46.0, 'y': 27.0, 'z': 18.333333333333332}, 'age': {'x': 25.5, 'y': 39.8, 'z': 29.333333333333332}}  
data.groupby('group').transform('mean')  
  
# 数据透视表  
pd.pivot_table(data,   
               values='salary',   
               columns='group',   
               index='age',   
               aggfunc='count',   
               margins=True    
).reset_index()
```


### 1.6.3 多表组合

```python
import pandas as pd  
import numpy as np  
  
group = ['x','y','z']  
data1 = pd.DataFrame({  
    "group":[group[x] for x in np.random.randint(0,len(group),10)] ,  
    "age":np.random.randint(15,50,10)  
    })  
  
data2 = pd.DataFrame({  
    "group":[group[x] for x in np.random.randint(0,len(group),10)] ,  
    "salary":np.random.randint(5,50,10),  
    })  
  
data3 = pd.DataFrame({  
    "group":[group[x] for x in np.random.randint(0,len(group),10)] ,  
    "age":np.random.randint(15,50,10),  
    "salary":np.random.randint(5,50,10),  
    })  
  
# 一对一  
pd.merge(data1, data2)  
  
# 多对一  
pd.merge(data3, data2, on='group')  
  
# 多对多  
pd.merge(data3, data2)  
  
# 连接键类型，解决没有公共列问题  
pd.merge(data3, data2, left_on= 'age', right_on='salary')  
  
# 连接方式  
# 内连接，不指明连接方式，默认都是内连接  
pd.merge(data3, data2, on= 'group', how='inner')  
# 左连接 left# 右连接 right# 外连接 outer  
# 纵向拼接  
pd.concat([data1, data2])
```

### 1.6.4 输出和绘图
- **输出**

```python
# 导出为.xlsx文件  
df.to_excel( excel_writer = r'file.xlsx')  
  
# 设置Sheet名称  
df.to_excel( excel_writer = r'file.xlsx', sheet_name = 'sheet1')  
  
# 设置索引,设置参数index=False就可以在导出时把这种索引去掉  
df.to_excel( excel_writer = r'file.xlsx', sheet_name = 'sheet1', index = False)  
  
# 设置要导出的列  
df.to_excel( excel_writer = r'file.xlsx', sheet_name = 'sheet1',   
             index = False, columns = ['col1','col2'])  
  
# 设置编码格式  
enconding = 'utf-8'  
  
# 缺失值处理  
na_rep = 0 # 缺失值填充为0  
  
# 无穷值处理  
inf_rep = 0  
  
# 导出为.csv文件  
to_csv()  
  
# 性能  
df.to_pickle('xx.pkl')   
  
agg(sum) # 快  
agg(lambda x: x.sum()) # 慢
```

- **绘图**


```python
import pandas as pd  
import numpy as np  
  
dates = pd.date_range('20200101', periods=12)  
df = pd.DataFrame(np.random.randn(12,4), index=dates, columns=list('ABCD'))  
df  
  
#                    A         B         C         D  
# 2020-01-01  0.046485 -0.556209  1.062881 -1.174129  
# 2020-01-02  1.066051 -0.343081  1.054913  1.601051  
# 2020-01-03  0.191064 -0.386905  0.516403  0.259818  
# 2020-01-04 -0.168462 -1.488041 -0.457658  0.913574  
# 2020-01-05 -0.502614  1.235633 -0.578284 -0.362737  
# 2020-01-06 -0.193310  0.652285 -0.346359  0.347364  
# 2020-01-07  2.308562 -0.679108  0.856449  0.490840  
# 2020-01-08  0.871489  0.338133 -0.163669  0.300147  
# 2020-01-09 -1.245250  0.667357 -1.287782  1.494880  
# 2020-01-10  0.387925 -1.058867 -0.397298  0.514921  
# 2020-01-11 -0.440884  0.904307  1.338720  0.612919  
# 2020-01-12 -0.864941 -0.358934 -0.203868 -1.191186  
  
import matplotlib.pyplot as plt  
plt.plot(df.index, df['A'], )  
plt.show()  
  
plt.plot(df.index, df['A'],   
        color='#FFAA00',    # 颜色  
        linestyle='--',     # 线条样式  
        linewidth=3,        # 线条宽度  
        marker='D')         # 点标记  
  
plt.show()  
  
# seaborn其实是在matplotlib的基础上进行了更高级的API封装，从而使绘图更容易、更美观  
import seaborn as sns  
# 绘制散点图  
plt.scatter(df.index, df['A'])  
plt.show()  
  
# 美化plt  
sns.set_style('darkgrid')  
plt.scatter(df.index, df['A'])  
plt.show()
```


# 2 Numpy 使用

## 2.1 Ndarray对象

NumPy 最重要的一个特点是其 N 维数组对象 ndarray，它是一系列同类型数据的集合，以 0 下标为开始进行集合中元素的索引。ndarray 对象是用于存放同类型元素的多维数组。ndarray 中的每个元素在内存中都有相同存储大小的区域。ndarray 内部由以下内容组成：

- 一个指向数据（内存或内存映射文件中的一块数据）的指针。
- 数据类型或 dtype，描述在数组中的固定大小值的格子。
- 一个表示数组形状（shape）的元组，表示各维度大小的元组。
- 一个跨度元组（stride），其中的整数指的是为了前进到当前维度下一个元素需要"跨过"的字节数。
  
**ndarray 的内部结构：**

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312131623123.png)

1. **`np.array()` 函数**：使用 Python 列表或元组创建数组。
3. **`np.zeros()` 和 `np.ones()` 函数**：创建特定形状的全零数组或全一数组。

```python
# 创建一个 3x4 的全零数组
zeros_arr = np.zeros((3, 4))
# 创建一个 2x3 的全一数组
ones_arr = np.ones((3, 4))
```

3. **`np.empty()` 函数**：创建指定形状的未初始化数组。

```python
# 创建一个 2x2 的未初始化数组
empty_arr = np.empty((2, 2))  
```

4. **`np.arange()` 函数**：类似于 Python 的 `range()` 函数，创建一个指定范围内的数组。

```python
# 创建一个从 0 到 10（不包括10），步长为2的数组
arr_range = np.arange(0, 10, 2)  
```

2. **`np.linspace()` 函数**：创建指定范围内均匀间隔的数组。

```python
 # 创建一个从 0 到 10（包括10），共5个元素的数组
arr_linspace = np.linspace(0, 10, 5) 
```

3. **`np.random` 模块**：生成随机数组。

- **生成随机整数数组：**

```python
# 创建一个 3x3 的随机整数数组（范围为1到100）
rand_int_arr = np.random.randint(1, 100, size=(3, 3))  
```

- **生成随机浮点数数组：**

```python
# 创建一个 2x2 的随机浮点数数组（范围在0到1之间）
rand_float_arr = np.random.rand(2, 2) 
```

- **生成符合正态分布的随机数组：**

```python
# 创建一个 2x2 符合正态分布的随机数组
rand_normal_arr = np.random.randn(2, 2)  

# array([[ 2.0794976 , -1.64938592],
#       [-0.17147869,  0.28570884]])
```

## 2.2 Ndarray 的操作方法

- Ndarray的基本属性

```python
X = np.arange(15).reshape(3,5)
# array([[ 0,  1,  2,  3,  4],
#       [ 5,  6,  7,  8,  9],
#       [10, 11, 12, 13, 14]])
x = np.arange(10)
# array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])

# 矩阵维度
x.ndim # 1
X.ndim # 2

# 数组维度
x.shape # (10, )
X.shape # (3,5)

# 数组元素个数
x.size  # 10
X.size  # 15

```

- Ndarray 基本操作

```python
# 切片
x[::2] # 2 表示步长
# 二维数组可以使用元组方式，或者 逗号，行列维度通过逗号分隔
X[ (1,1) ] # 6
X[1,1] # 6

X[:2, :3]  # 取前2行，前3列
# array([[0, 1, 2],
#       [5, 6, 7]])

subarray = X[:2, :3]
subarray[1][1] = 100  # 修改子矩阵，X 矩阵也会被修改
subX = X[:2, :3].copy()  # 可以使用copy，修改subX，不会修改X

a = np.arange(10)
a.reshape(2,5) # 输出 2 维矩阵
# array([[0, 1, 2, 3, 4],
#       [5, 6, 7, 8, 9]])
a.reshape(10,-1)  # 10指定行维度，-1 指定列维度由自动生成

# 合并操作
x = np.array([1,2,3])
y = np.array([4,5,6])
np.concatenate([x, y]) # array([1, 2, 3, 4, 5, 6])

# np.concatenate 要求维度相同，否则合并报错

# 垂直方向合并
np.vstack([x, y]) 
# array([[1, 2, 3],
#       [4, 5, 6]])
# 水平方向合并
np.hstack([x, y]) # array([1, 2, 3, 4, 5, 6])

x = np.arange(10)
x1, x2, x3 = np.split(x, [3,7]) # 会把 x 划分为 0-3, 3-7, 7-end

A = np.arange(16).reshape([4,4])
# array([[ 0,  1,  2,  3],
#       [ 4,  5,  6,  7],
#       [ 8,  9, 10, 11],
#       [12, 13, 14, 15]])
A1, A2 = np.split(A, [2])  # 垂直分割，A1 是前两行，A2 是后两行
A1, A2 = np.split(A, [2], axis=1) # 水平分割，A1 是前两列，A2 是后两列
A1, A2 = np.vsplit(A, [2]) # 垂直分割
A1, A2= np.hsplit(A, [2]) # 垂直分割
```

## 2.3 numpy.array 中的聚合运算

- np.percentile(big_array, q=50) ：百分位点，相当于求50%位点
- np.var(big_array)：方差，是每个样本值与全体样本值的平均数之差的平方值的平均数，即 `mean((x - x.mean())** 2)`。
- np.std(big_array)：标准差，标准差是一组数据平均值分散程度的一种度量。标准差是方差的算术平方根。
- np.median(big_array)：中位数
- np.mean()：平均数
- `numpy.average(a, axis=None, weights=None, returned=False)` ：函数根据在另一个数组中给出的各自的权重计算数组中元素的加权平均值。 
- np.bincount：众数
- np.prod：大数
- np.argmax()：返回最大值的索引
- np.argmin()
- np.random.shuffle(x)：乱序处理
- np.sort(x)：排序，x 未修改
- x.sort()：原地排序，x 被修改
- np.sort(x, axis=0)：每行排序
- np.sort(x, axis=1)：每列排序
- np.argsort(x)：排序数组，返回的是元素索引
- np.partition(x, 3)：快排中的partition逻辑，返回的数组中，3 前边的数据都小于3，3后边的元素都大于3
- np.argpartition(x, 3)：与上边的一样，只是返回的索引


## 2.4 FancyIndexing

```python
X = np.arange(16).reshape(4,4)
# array([[ 0,  1,  2,  3],
#       [ 4,  5,  6,  7],
#       [ 8,  9, 10, 11],
#       [12, 13, 14, 15]])
ind = np.array([[0,2],
         [1,3]])
X[ind] # 取出第0，2行 和 1，3行分别组成矩阵

row = np.array([0,1,2])
col = np.array([1,2,3])
X[row, col] # 返回 (0,1)、(1,2)、(2,3) 的点

# 通过bool数组选定值
col = [True, False, True, True]
X[1:3, col] # 输出第2、3行的数据，列受col控制

```


- numpy.array 的比较

```python
x = np.arange(16)
# array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14, 15])
x < 3
# array([ True,  True,  True, False, False, False, False, False, False,
#       False, False, False, False, False, False, False])
2 * x == 24-4 * x # x中那些值符合表达式
#array([False, False, False, False,  True, False, False, False, False,
#       False, False, False, False, False, False, False])

# 二维数组类似
X[X[:,2] > 3, :] # 取出第3列大于3的所有行
np.sum(X>3) # 二维数组中大于3的个数
np.sum(X > 3, axis=0) # array([3, 3, 3, 3]) 列维度上，每列大于3的个数都是3个
np.sum(X > 3, axis=1) # array([0, 4, 4, 4]) 行维度上，每行大于3个数

np.any(X < 0)  # X中没有小于0
np.all(X < 0) # False 因为有一个数等于0
np.sum((X>3) & (X < 10)) # 大于3小于10的个数
np.sum(~(X==0)) # 不等于0的个数
```

## 2.5 视图和副本
副本是一个数据的完整的拷贝，如果我们对副本进行修改，它不会影响到原始数据，物理内存不在同一位置。
视图是数据的一个别称或引用，通过该别称或引用亦便可访问、操作原有数据，但原有数据不会产生拷贝。如果我们对视图进行修改，它会影响到原始数据，物理内存在同一位置。

**视图一般发生在：**
- 1、numpy 的切片操作返回原数据的视图。
- 2、调用 ndarray 的 view() 函数产生一个视图。

**副本一般发生在：**
- Python 序列的切片操作，调用deepCopy()函数。
- 调用 ndarray 的 copy() 函数产生一个副本。


视图示例：
```python
a = np.arange(6).reshape(3,2)
b = a.view()
print(a)
# array([[0, 1],
#       [2, 3],
#       [4, 5]])
# 修改 b 的形状，并不会修改 a
b.shape = 2,3
print(b)
# array([[0, 1, 2],
#       [3, 4, 5]])
b[0][0] = 100 # 修改b后，a中对应位置也会被修改
```

副本示例：
```python
x = np.arange(6).reshape(2,3)
y = x.copy()
# 修改y不会影响x
y[0][0]=100
```

## 2.6 Numpy线性代数

|函数|描述|
|---|---|
|`dot`|两个数组的点积，即元素对应相乘。|
|`vdot`|两个向量的点积|
|`inner`|两个数组的内积|
|`matmul`|两个数组的矩阵积|
|`determinant`|数组的行列式|
|`solve`|求解线性矩阵方程|
|`inv`|计算矩阵的乘法逆矩阵|



# 3 Matplotlib 使用

## 3.1 折线图
```python
import numpy as np
import matplotlib.pyplot as plt

plt.plot(x, siny, color='red', linestyle='--', label='sin(y)') # 指定颜色和线条
plt.plot(y, sinx, label='sin(x)')
# plt.xlim(100, 200) # 指定x轴范围
# plt.ylim(0, 1) # 指定y轴范围
plt.axis([-1, 210, -2, 5]) # 指定 x 轴和y轴范围
plt.legend() # 图中添加标签图示
plt.xlabel("x siny") # 给X轴命名
plt.ylabel("x sinx") # 给X轴命名
plt.title("Welcome to the ML World!!") # 添加标题
plt.show()
```
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312141108094.png)


## 3.2 散点图

```python
x = np.random.normal(0, 1, 10000)
y = np.random.normal(0, 1, 10000)
plt.scatter(x, y, alpha=0.1) # 透明度为0.1
plt.show()
```

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312141109617.png)

# 4 数据加载和数据处理

```python
import numpy as np
import matplotlib as mpl
import matplotlib.pyplot as plt
from sklearn import datasets # 加载自带的一个数据集

iris = datasets.load_iris()
iris.keys() # 数据中包含的字段
# dict_keys(['data', 'target', 'frame', 'target_names', 'DESCR', 'feature_names', 'filename', 'data_module'])
iris.data  # 数据集
iris.data.shape # （105，4）数据维度
iris.target 
```

通过数据集绘图：
```python
X = iris.data
y = iris.target
plt.scatter(X[y==0,0], X[y==0,1], color='red', marker='o')
plt.scatter(X[y==1,0], X[y==1,1], color='blue', marker='+')
plt.scatter(X[y==2,0], X[y==2,1], color='green', marker='x')
plt.show()
```
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312141129223.png)

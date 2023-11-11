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



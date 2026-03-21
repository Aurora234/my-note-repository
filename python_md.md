# numpy

## numpy的线性代数

### 矩阵乘法(点积)

```python
# 方法1: @ 运算符 (推荐)
result1 = A @ B
print("A @ B:\n", result1)

# 方法2: dot 函数
result2 = np.dot(A, B)
print("np.dot(A, B):\n", result2)

# 方法3: matmul 函数
result3 = np.matmul(A, B)
print("np.matmul(A, B):\n", result3)
```
### 逆矩阵
```python
A_inv = np.linalg.inv(A)
print("逆矩阵 A⁻¹:\n", A_inv)
# 验证 A × A⁻¹ = 单位矩阵
identity = A @ A_inv
print("A × A⁻¹ (应该接近单位矩阵):\n", np.round(identity, 10))
```
### 行列式计算
```python
B = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
det_B = np.linalg.det(B)
```
### 线性方程组求解
```python
# 系数矩阵
A = np.array([[3, 1], [1, 2]])
# 常数向量
b = np.array([9, 8])
print("方程组:")
print("3x + y = 9")
print("x + 2y = 8")
# 求解
x = np.linalg.solve(A, b)
print("解向量 x:", x)  # [2. 3.]
```
### 矩阵的迹和秩
```python
A = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
# 迹 (主对角线元素之和)
trace_A = np.trace(A)
print("矩阵 A:\n", A)
print("迹 tr(A):", trace_A)  # 1 + 5 + 9 = 15

# 矩阵的秩
rank_A = np.linalg.matrix_rank(A)
print("秩 rank(A):", rank_A)  # 这个矩阵的秩为2

# 满秩矩阵的例子
B = np.array([[1, 2], [3, 4]])
print("矩阵 B 的秩:", np.linalg.matrix_rank(B))  # 2
```
### 特征值和特征向量
```python
A = np.array([[4, -2], [1, 1]])
print("矩阵 A:\n", A)

# 计算特征值和特征向量
eigenvalues, eigenvectors = np.linalg.eig(A)

print("特征值:", eigenvalues)
print("特征向量:\n", eigenvectors)

# 验证: A × v = λ × v
for i in range(len(eigenvalues)):
    λ = eigenvalues[i]
    v = eigenvectors[:, i]
    verification = A @ v
    expected = λ * v
    print(f"验证特征向量 {i}: A×v = {verification}, λ×v = {expected}")
```
---
## numpy的random模块
### 生成随机浮点数
```python
# 生成 [0.0, 1.0) 之间的随机浮点数
random_float = np.random.random()
print("随机浮点数:", random_float)

# 生成指定形状的随机数组
random_array = np.random.random((3, 4))
print("3x4随机数组:\n", random_array)

# 生成 [0.0, 1.0) 的5个随机数
random_5 = np.random.random(5)
print("5个随机数:", random_5)
```
### 生成随机整数
```python
random_int = np.random.randint(0, 10)  # 0到9的随机整数
print("0-9随机整数:", random_int)

# 生成多个随机整数
random_ints = np.random.randint(1, 100, 10)  # 10个1-99的随机整数
print("10个随机整数:", random_ints)

# 生成二维随机整数数组
random_matrix = np.random.randint(0, 2, (3, 3))  # 3x3的0-1矩阵
print("3x3随机整数矩阵:\n", random_matrix)
```

### 概率分布随机数
* 正态分布
```python
# 标准正态分布 (均值0, 标准差1)
normal_data = np.random.randn(10)  # 10个标准正态随机数
print("标准正态分布:", normal_data)

# 自定义正态分布
mu, sigma = 170, 5  # 均值170, 标准差5
height_data = np.random.normal(mu, sigma, 100)  # 100个身高数据
print("身高数据示例:", height_data[:5])

# 二维正态分布
normal_2d = np.random.normal(0, 1, (5, 3))  # 5x3的正态分布数组
print("二维正态分布:\n", normal_2d)
```
* 均匀分布
```python
# [0.0, 1.0) 均匀分布
uniform_1d = np.random.uniform(0, 1, 10)
print("均匀分布:", uniform_1d)

# 自定义范围的均匀分布
uniform_custom = np.random.uniform(10, 20, 5)  # [10, 20) 的5个随机数
print("10-20均匀分布:", uniform_custom)
```
* 二项分布
````python
# 二项分布: n次试验，每次成功概率p
# 模拟10次抛硬币，每次抛5枚，正面概率0.5
coin_toss = np.random.binomial(5, 0.5, 10)
print("抛硬币结果:", coin_toss)  # 每个数表示正面朝上的硬币数

# 模拟1000次抛10枚硬币
large_sample = np.random.binomial(10, 0.5, 1000)
print("平均正面数:", np.mean(large_sample))  # 应该接近5
````
* 泊松分布
```python
# 泊松分布: 单位时间内事件发生次数
# 模拟呼叫中心每小时接到的电话数 (λ=5)
calls_per_hour = np.random.poisson(5, 24)  # 24小时的数据
print("每小时电话数:", calls_per_hour)
print("平均电话数:", np.mean(calls_per_hour))
```
### 随机抽样和排列
* 从已有数组抽样
```python
data = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

# 随机选择一个元素
choice = np.random.choice(data)
print("随机选择:", choice)

# 随机选择多个元素 (无放回)
choices_no_replace = np.random.choice(data, size=3, replace=False)
print("无放回抽样:", choices_no_replace)

# 随机选择多个元素 (有放回)
choices_replace = np.random.choice(data, size=5, replace=True)
print("有放回抽样:", choices_replace)

# 按概率抽样
probabilities = [0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1]  # 均匀概率
weighted_choice = np.random.choice(data, size=3, p=probabilities)
print("按概率抽样:", weighted_choice)
```
* 随机排列
```python
arr = np.arange(10)
print("原始数组:", arr)

# 打乱数组 (原地修改)
np.random.shuffle(arr)
print("打乱后:", arr)

# 生成随机排列 (返回新数组)
permutation = np.random.permutation(10)
print("随机排列:", permutation)

# 对二维数组的行进行打乱
matrix = np.arange(12).reshape(4, 3)
print("原始矩阵:\n", matrix)
np.random.shuffle(matrix)  # 打乱行顺序
print("打乱行后:\n", matrix)
```

---
# pandas

**两种主要数据结构**

* Series：一维带标签数组
* DataFrame：二维数据表格

### 1. Series基本用法

* 创建Series
```python
# 从列表创建
s1 = pd.Series([1, 3, 5, np.nan, 6, 8])
print("从列表创建:\n", s1)

# 从字典创建（键成为索引）
s2 = pd.Series({'a': 1, 'b': 3, 'c': 5})
print("从字典创建:\n", s2)

# 指定索引
s3 = pd.Series([10, 20, 30], index=['x', 'y', 'z'])
print("指定索引:\n", s3)
```
* Series操作
```python
# 基本属性
print("值:", s3.values)
print("索引:", s3.index)
print("数据类型:", s3.dtype)

# 索引访问
print("s3['x']:", s3['x'])  # 10
print("s3[0]:", s3[0])      # 10

# 向量化运算
print("s3 * 2:\n", s3 * 2)
print("s3 > 15:\n", s3 > 15)
```
### 2.DataFrame的创建
* 从字典中创建
```python
# 最常用的创建方式
data = {
    '姓名': ['张三', '李四', '王五', '赵六'],
    '年龄': [25, 30, 35, 28],
    '城市': ['北京', '上海', '广州', '深圳'],
    '工资': [5000, 8000, 7000, 6000]
}

df = pd.DataFrame(data)
print("DataFrame:\n", df)
```
* 从其他数据结构创建
```python
# 从列表的列表创建
data_list = [
    ['张三', 25, '北京', 5000],
    ['李四', 30, '上海', 8000],
    ['王五', 35, '广州', 7000]
]

df2 = pd.DataFrame(data_list, columns=['姓名', '年龄', '城市', '工资'])
print("从列表创建:\n", df2)

# 从NumPy数组创建
np_array = np.random.randn(4, 3)
df3 = pd.DataFrame(np_array, columns=['A', 'B', 'C'])
print("从NumPy数组创建:\n", df3)
```

### 3.数据查看和基本信息
* 查看数据
```python
print("前3行:")
print(df.head(3))

print("\n后2行:")
print(df.tail(2))

print("\n数据形状:", df.shape)
print("列名:", df.columns.tolist())
print("索引:", df.index.tolist())
```
* 数据信息
```python
# 基本信息
print("数据类型:\n", df.dtypes)
print("\n基本信息:")
print(df.info())

print("\n描述性统计:")
print(df.describe())

print("\n唯一值:")
print("城市的唯一值:", df['城市'].unique())
print("城市的值计数:\n", df['城市'].value_counts())
```

### 4.数据选择与索引
* 选择列
```python
# 选择单列 (返回Series)
names = df['姓名']
print("姓名列:\n", names)

# 选择多列 (返回DataFrame)
subset = df[['姓名', '工资']]
print("姓名和工资列:\n", subset)
```
* 选择行
```python
# 按位置选择
print("第2行:\n", df.iloc[1])        # 单行
print("前3行:\n", df.iloc[0:3])      # 多行

# 按标签选择
print("按标签选择:\n", df.loc[0:2])   # 包含结束位置

# 布尔索引
high_salary = df[df['工资'] > 6000]
print("工资>6000的员工:\n", high_salary)

beijing_employees = df[df['城市'] == '北京']
print("北京员工:\n", beijing_employees)
```
* 条件选择
```python
# 多条件选择
condition = (df['工资'] > 6000) & (df['年龄'] < 35)
selected = df[condition]
print("工资>6000且年龄<35:\n", selected)

# 使用query方法
selected2 = df.query('工资 > 6000 and 年龄 < 35')
print("使用query方法:\n", selected2)

# 包含特定值的筛选
tech_employees = df[df['部门'].str.contains('技术')]
print("技术部门员工:\n", tech_employees)
```

### 5.数据修改
* 添加新列
```python
# 基于现有列计算
df['年薪'] = df['工资'] * 12
print("添加年薪列:\n", df)

# 使用apply函数
df['年龄组'] = df['年龄'].apply(lambda x: '青年' if x < 30 else '中年')
print("添加年龄组:\n", df)

# 使用np.where
df['高工资'] = np.where(df['工资'] > 6500, '高', '低')
print("添加工资等级:\n", df)
```
* 修改数据
```python
# 修改单个值
df.loc[0, '工资'] = 5500
print("修改后的数据:\n", df)

# 批量修改
df.loc[df['城市'] == '北京', '工资'] += 1000
print("北京员工加薪后:\n", df)

# 使用replace
df['城市'] = df['城市'].replace({'北京': 'Beijing', '上海': 'Shanghai'})
print("城市名替换后:\n", df)
```
* 删除数据
```python
# 删除列
df_dropped = df.drop(['年薪', '年龄组'], axis=1)
print("删除列后:\n", df_dropped)

# 删除行
df_dropped_rows = df.drop([0, 2])  # 删除索引0和2的行
print("删除行后:\n", df_dropped_rows)

# 删除缺失值
df_with_nan = df.copy()
df_with_nan.loc[0, '工资'] = np.nan
df_cleaned = df_with_nan.dropna()
print("删除缺失值后:\n", df_cleaned)
```

### 6.数据处理和清洗
* 处理缺失值
```python
# 创建包含缺失值的数据
df_nan = pd.DataFrame({
    'A': [1, 2, np.nan, 4],
    'B': [5, np.nan, np.nan, 8],
    'C': [10, 20, 30, 40]
})

print("原始数据（含缺失值）:\n", df_nan)
print("\n缺失值统计:\n", df_nan.isnull().sum())

# 填充缺失值
df_filled = df_nan.fillna({'A': 0, 'B': df_nan['B'].mean()})
print("填充缺失值后:\n", df_filled)

# 删除缺失值
df_dropped_na = df_nan.dropna()
print("删除缺失值后:\n", df_dropped_na)
```
* 数据类型转换
```python
# 查看数据类型
print("数据类型:\n", df.dtypes)

# 转换数据类型
df['年龄'] = df['年龄'].astype('float64')
print("转换年龄类型后:\n", df.dtypes)

# 字符串操作
df['姓名大写'] = df['姓名'].str.upper()
df['城市长度'] = df['城市'].str.len()
print("字符串操作后:\n", df[['姓名', '姓名大写', '城市', '城市长度']])
```

### 7.数据排序
```python
# 按单列排序
df_sorted = df.sort_values('工资', ascending=False)
print("按工资降序排序:\n", df_sorted)

# 按多列排序
df_sorted_multi = df.sort_values(['城市', '工资'], ascending=[True, False])
print("按城市升序、工资降序排序:\n", df_sorted_multi)

# 按索引排序
df_sorted_index = df.sort_index(ascending=False)
print("按索引降序排序:\n", df_sorted_index)
```

### 8.分组和聚合
* 基本分组操作
```python
# 按单列分组
grouped_city = df.groupby('城市')
print("按城市分组后的平均工资:\n", grouped_city['工资'].mean())

# 按多列分组
grouped_multi = df.groupby(['城市', '部门'])
print("按城市和部门分组:\n", grouped_multi['工资'].mean())
```
* 多种聚合函数
```python
# 对同一列使用多个聚合函数
agg_result = df.groupby('城市')['工资'].agg(['mean', 'sum', 'count', 'std'])
print("各城市工资统计:\n", agg_result)

# 对不同列使用不同聚合函数
agg_result_multi = df.groupby('部门').agg({
    '工资': ['mean', 'max'],
    '年龄': ['min', 'max', 'mean']
})
print("各部门多指标统计:\n", agg_result_multi)
```

### 9.数据透视表
```python
# 创建数据透视表
pivot_table = df.pivot_table(
    values='工资',
    index='城市',
    columns='部门',
    aggfunc='mean',
    fill_value=0
)
print("数据透视表:\n", pivot_table)

# 多维度数据透视
pivot_multi = df.pivot_table(
    values='工资',
    index=['城市', '部门'],
    aggfunc=['mean', 'count']
)
print("多维度数据透视:\n", pivot_multi)
```

### 10.数据合并
* 连接多个DataFrame
```python
# 创建第二个DataFrame
df2 = pd.DataFrame({
    '姓名': ['张三', '李四', '孙八'],
    '工龄': [3, 5, 2],
    '绩效': ['A', 'B', 'A']
})

print("df2:\n", df2)

# 内连接
merged_inner = pd.merge(df, df2, on='姓名', how='inner')
print("内连接结果:\n", merged_inner)

# 左连接
merged_left = pd.merge(df, df2, on='姓名', how='left')
print("左连接结果:\n", merged_left)
```
* 拼接DataFrame
```python
# 垂直拼接（添加行）
new_employee = pd.DataFrame({
    '姓名': ['周九'],
    '年龄': [29],
    '城市': ['杭州'],
    '工资': [6500],
    '部门': ['技术']
})

df_appended = pd.concat([df, new_employee], ignore_index=True)
print("拼接后的数据:\n", df_appended)
```

### 11.文件读写
```python
# 从CSV文件读取
# df = pd.read_csv('data.csv')

# 从Excel文件读取  
# df = pd.read_excel('data.xlsx')

# 读取时指定参数
# df = pd.read_csv('data.csv', encoding='utf-8', sep=',', index_col=0)
```
```python
# 保存为CSV
# df.to_csv('output.csv', index=False)

# 保存为Excel
# df.to_excel('output.xlsx', sheet_name='员工数据')

# 保存为JSON
# df.to_json('output.json', orient='records')
```

### 12.时间序列处理
```python
# 创建时间序列数据
dates = pd.date_range('20230101', periods=6)
time_series_df = pd.DataFrame({
    '日期': dates,
    '销售额': np.random.randint(1000, 5000, 6),
    '产品': ['A', 'B', 'A', 'B', 'A', 'B']
})

print("时间序列数据:\n", time_series_df)

# 设置日期索引
time_series_df.set_index('日期', inplace=True)
print("设置日期索引后:\n", time_series_df)

# 时间重采样
monthly_sales = time_series_df['销售额'].resample('M').sum()
print("月度销售额:\n", monthly_sales)
```





# list、set的用法及联合操作

## 1. 集合 (Set) 的用法和操作

### 创建集合

python

```python
# 创建空集合
empty_set = set()

# 创建有元素的集合
my_set = {1, 2, 3, 4, 5}
mixed_set = {1, "hello", 3.14, True}

# 从列表创建集合
list_to_set = set([1, 2, 2, 3, 4, 4, 5])  # {1, 2, 3, 4, 5}
```



### 集合基本操作

python

```python
my_set = {1, 2, 3, 4, 5}

# 添加元素
my_set.add(6)           # {1, 2, 3, 4, 5, 6}
my_set.update([7, 8])   # {1, 2, 3, 4, 5, 6, 7, 8}

# 删除元素
my_set.remove(3)        # 删除元素3，如果不存在会报错
my_set.discard(10)      # 删除元素10，如果不存在不会报错
popped = my_set.pop()   # 随机删除并返回一个元素

# 检查元素是否存在
if 2 in my_set:
    print("2在集合中")

# 集合长度
length = len(my_set)
```



### 集合运算

python

```python
set_a = {1, 2, 3, 4, 5}
set_b = {4, 5, 6, 7, 8}

# 并集
union_set = set_a | set_b           # {1, 2, 3, 4, 5, 6, 7, 8}
union_set = set_a.union(set_b)      # 同上

# 交集
intersection_set = set_a & set_b    # {4, 5}
intersection_set = set_a.intersection(set_b)  # 同上

# 差集
difference_set = set_a - set_b      # {1, 2, 3}
difference_set = set_a.difference(set_b)  # 同上

# 对称差集 (只在其中一个集合中出现的元素)
symmetric_diff = set_a ^ set_b      # {1, 2, 3, 6, 7, 8}
symmetric_diff = set_a.symmetric_difference(set_b)  # 同上

# 子集和超集检查
set_c = {1, 2}
print(set_c.issubset(set_a))        # True
print(set_a.issuperset(set_c))      # True
```



## 2. 列表 (List) 的用法和操作

### 创建列表

python

```python
# 创建空列表
empty_list = []
empty_list = list()

# 创建有元素的列表
my_list = [1, 2, 3, 4, 5]
mixed_list = [1, "hello", 3.14, True, [1, 2, 3]]

# 使用range创建列表
numbers = list(range(10))           # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```



### 列表基本操作

python

```python
my_list = [1, 2, 3, 4, 5]

# 访问元素
first = my_list[0]                  # 1
last = my_list[-1]                  # 5
sublist = my_list[1:4]              # [2, 3, 4]

# 修改元素
my_list[0] = 10                     # [10, 2, 3, 4, 5]

# 添加元素
my_list.append(6)                   # [10, 2, 3, 4, 5, 6]
my_list.insert(1, 1.5)              # [10, 1.5, 2, 3, 4, 5, 6]
my_list.extend([7, 8])              # [10, 1.5, 2, 3, 4, 5, 6, 7, 8]

# 删除元素
removed = my_list.pop()             # 删除并返回最后一个元素
removed = my_list.pop(1)            # 删除并返回索引1的元素
my_list.remove(3)                   # 删除第一个出现的3
del my_list[0]                      # 删除索引0的元素

# 查找元素
index = my_list.index(4)            # 返回4的索引
count = my_list.count(2)            # 返回2出现的次数

# 排序
my_list.sort()                      # 升序排序
my_list.sort(reverse=True)          # 降序排序
sorted_list = sorted(my_list)       # 返回新排序列表，原列表不变

# 反转
my_list.reverse()                   # 反转列表
reversed_list = list(reversed(my_list))  # 返回新反转列表
```



### 列表推导式

python

```python
# 创建平方数列表
squares = [x**2 for x in range(10)]

# 带条件的列表推导式
even_squares = [x**2 for x in range(10) if x % 2 == 0]

# 嵌套列表推导式
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [num for row in matrix for num in row]  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```



## 3. 列表和集合的联合操作

### 相互转换

python

```python
# 列表转集合 (去重)
my_list = [1, 2, 2, 3, 4, 4, 5]
my_set = set(my_list)               # {1, 2, 3, 4, 5}

# 集合转列表
new_list = list(my_set)             # [1, 2, 3, 4, 5] (顺序可能不同)

# 保持顺序的去重
ordered_list = list(dict.fromkeys(my_list))  # [1, 2, 3, 4, 5] (保持顺序)
```



### 常用联合操作

python

```python
# 1. 列表去重
duplicate_list = [1, 2, 2, 3, 4, 4, 4, 5]
unique_list = list(set(duplicate_list))  # [1, 2, 3, 4, 5]

# 2. 找出两个列表的共同元素
list1 = [1, 2, 3, 4, 5]
list2 = [4, 5, 6, 7, 8]
common_elements = list(set(list1) & set(list2))  # [4, 5]

# 3. 找出只在其中一个列表中的元素
unique_to_list1 = list(set(list1) - set(list2))  # [1, 2, 3]
unique_to_list2 = list(set(list2) - set(list1))  # [6, 7, 8]

# 4. 合并列表并去重
combined_unique = list(set(list1 + list2))  # [1, 2, 3, 4, 5, 6, 7, 8]

# 5. 检查列表是否有重复元素
def has_duplicates(lst):
    return len(lst) != len(set(lst))

print(has_duplicates([1, 2, 3, 4, 5]))     # False
print(has_duplicates([1, 2, 3, 3, 4]))     # True

# 6. 找出列表中的唯一元素（只出现一次的元素）
from collections import Counter
def find_unique_elements(lst):
    counter = Counter(lst)
    return [item for item, count in counter.items() if count == 1]

print(find_unique_elements([1, 2, 2, 3, 4, 4, 5]))  # [1, 3, 5]
```



### 实际应用示例

python

```python
# 示例1: 处理用户标签
user1_tags = ["python", "programming", "coding", "tech"]
user2_tags = ["python", "data", "analysis", "tech"]

# 共同兴趣
common_interests = list(set(user1_tags) & set(user2_tags))
print("共同兴趣:", common_interests)  # ['python', 'tech']

# 所有不重复的兴趣
all_interests = list(set(user1_tags + user2_tags))
print("所有兴趣:", all_interests)  # ['python', 'programming', 'coding', 'tech', 'data', 'analysis']

# 示例2: 数据清洗
raw_data = ["apple", "banana", "Apple", "BANANA", "apple", "orange"]
cleaned_data = list(set(item.lower() for item in raw_data))
print("清洗后数据:", cleaned_data)  # ['banana', 'orange', 'apple']

# 示例3: 权限管理
user_roles = ["admin", "editor", "viewer", "editor", "admin"]
available_roles = ["admin", "editor", "viewer", "guest"]

# 验证用户角色是否有效
user_unique_roles = set(user_roles)
valid_roles = set(available_roles)
invalid_roles = user_unique_roles - valid_roles

if invalid_roles:
    print(f"无效角色: {list(invalid_roles)}")
else:
    print("所有角色都有效")
```



### 性能考虑

- **集合**：查找、添加、删除操作的平均时间复杂度为 O(1)
- **列表**：查找操作的平均时间复杂度为 O(n)，添加/删除操作取决于位置

python

```python
# 当需要频繁检查元素是否存在时，使用集合更高效
large_list = list(range(1000000))
large_set = set(large_list)

# 检查元素是否存在 - 集合更快
%timeit 999999 in large_set    # 约 O(1)
%timeit 999999 in large_list   # 约 O(n)
```



# 生成器generator

生成器（Generator）是Python中一种特殊的迭代器，它允许你按需生成值，而不是一次性生成所有值。这样可以节省内存，并且可以表示无限序列。

生成器有两种创建方式：

1. 生成器表达式：类似于列表推导式，但是使用圆括号。
2. 生成器函数：使用`yield`语句的函数。

### 1. 生成器表达式

生成器表达式的语法和列表推导式类似，但是使用圆括号而不是方括号。

python

```python
# 列表推导式
list_comp = [x**2 for x in range(5)]  # [0, 1, 4, 9, 16]

# 生成器表达式
gen_exp = (x**2 for x in range(5))
print(gen_exp)  # <generator object <genexpr> at 0x...>

# 可以通过迭代来获取值
for value in gen_exp:
    print(value)
```



### 2. 生成器函数

生成器函数使用`yield`语句来返回一个值，并暂停函数的执行，直到下一次请求值。当函数再次被调用时，它会从上次暂停的地方继续执行。

python

```python
def my_generator():
    yield 1
    yield 2
    yield 3

# 调用生成器函数返回一个生成器对象
gen = my_generator()

# 可以使用next()函数获取下一个值
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3
# print(next(gen))  # 抛出StopIteration异常

# 或者使用for循环迭代
for value in my_generator():
    print(value)
```



### 生成器的优势

- **内存效率**：生成器一次只产生一个值，不会一次性将整个序列加载到内存中。这对于处理大量数据或无限序列非常有用。
- **惰性求值**：值只有在被请求时才会生成，这允许表示无限序列。

### 生成器的方法

生成器对象有一些方法，如`send()`、`throw()`和`close()`，这些方法允许与生成器交互。

- `send(value)`：向生成器发送一个值，该值将成为当前yield表达式的结果，并返回生成器产生的下一个值。
- `throw(type[, value[, traceback]])`：在生成器暂停的地方抛出异常。
- `close()`：关闭生成器。

### 示例：使用send方法

python

```python
def generator_with_send():
    value = yield "First yield"
    yield value

gen = generator_with_send()
print(next(gen))        # 输出: First yield
print(gen.send("Hello")) # 输出: Hello
```



在这个例子中，首先调用`next(gen)`启动生成器，直到第一个yield。然后使用`send("Hello")`将"Hello"发送给生成器，这个值被赋值给变量`value`，然后生成器继续执行到下一个yield，返回`value`。

生成器是Python中强大且常用的特性，特别是在处理大数据流、异步编程和协程中。



# 内置函数

## map

`map()` 是一个内置函数，用于对一个或多个可迭代对象（如列表、元组等）中的每个元素应用一个指定的函数，并返回一个 **map 对象**（一种惰性求值的迭代器）。

### 基本语法：

```python
map(function, iterable, ...)
```

- `function`：要应用于每个元素的函数。
- `iterable`：一个或多个可迭代对象（如 list、tuple 等）。
- 如果传入多个可迭代对象，`function` 必须能接受相应数量的参数。

---

### 示例 1：单个可迭代对象

将列表中每个数字平方：

```python
numbers = [1, 2, 3, 4]
squared = map(lambda x: x ** 2, numbers)
print(list(squared))  # 输出: [1, 4, 9, 16]
```

> 注意：`map()` 返回的是一个 **迭代器**，所以通常需要用 `list()` 转换为列表才能看到全部结果。

---

### 示例 2：多个可迭代对象

对两个列表对应位置的元素求和：

```python
a = [1, 2, 3]
b = [10, 20, 30]
result = map(lambda x, y: x + y, a, b)
print(list(result))  # 输出: [11, 22, 33]
```

> 两个列表长度不同时，`map()` 会在最短的那个结束时停止（类似于 `zip` 的行为）。

---

### 示例 3：使用普通函数代替 lambda

```python
def to_upper(s):
    return s.upper()

words = ['hello', 'world']
upper_words = map(to_upper, words)
print(list(upper_words))  # 输出: ['HELLO', 'WORLD']
```

---

### 特点与注意事项：

- **惰性求值**：`map()` 不会立即计算所有结果，而是在你遍历（如用 `for` 循环或转成 `list`）时才逐个生成。这节省内存。
- **一次消费**：map 对象只能被遍历一次。再次遍历时为空。
- **性能**：对于简单操作，`map()` 通常比显式 `for` 循环快（尤其配合内置函数时），但可读性有时不如列表推导式。
- **替代写法**：很多情况下可以用 **列表推导式** 替代，例如：
  ```python
  [x**2 for x in numbers]  # 等价于 map(lambda x: x**2, numbers)
  ```

---

### 总结

`map()` 是函数式编程风格的工具，适合“对每个元素做相同操作”的场景。合理使用可以让代码更简洁高效。

---



## filter

在 Python 中，`filter()` 是一个内置函数，用于**过滤可迭代对象中的元素**。它根据一个函数的返回值（布尔类型）来决定保留或丢弃元素：

- 如果函数返回 `True`，则保留该元素；
- 如果返回 `False`，则丢弃该元素。

---

### 基本语法

```python
filter(function, iterable)
```

- `function`：一个接受单个参数并返回布尔值的函数（`True` 或 `False`）。
- `iterable`：一个可迭代对象（如列表、元组、字符串等）。
- 返回一个 **filter 对象**（惰性求值的迭代器）。

---

### 示例 1：使用 lambda 过滤奇数

```python
numbers = [1, 2, 3, 4, 5, 6]
odds = filter(lambda x: x % 2 == 1, numbers)
print(list(odds))  # 输出: [1, 3, 5]
```

> 只保留满足 `x % 2 == 1`（即奇数）的元素。

---

### 示例 2：使用普通函数过滤字符串长度

```python
words = ['cat', 'elephant', 'dog', 'hippopotamus']
long_words = filter(lambda w: len(w) > 3, words)
print(list(long_words))  # 输出: ['elephant', 'hippopotamus']
```

---

### 示例 3：过滤非空字符串

```python
items = ['hello', '', 'world', '', 'python']
non_empty = filter(lambda x: x != '', items)
print(list(non_empty))  # ['hello', 'world', 'python']
```

> 空字符串在布尔上下文中为 `False`，所以也可以写成：

```python
non_empty = filter(None, items)  # 相当于 filter(bool, items)
```

---

### 示例 4：过滤 None 值

```python
data = [1, None, 3, None, 5]
cleaned = filter(None, data)  # 过滤掉 false 值（包括 None、0、''、[] 等）
print(list(cleaned))  # [1, 3, 5]
```

> 当 `function` 为 `None` 时，`filter()` 会保留所有“真值”元素（相当于 `bool(item)` 为 `True`）。

---

### 特点与注意事项

- **惰性求值**：和 `map()`、`zip()` 一样，`filter()` 返回迭代器，节省内存。
- **一次消费**：filter 对象只能遍历一次。
- **性能**：对于简单过滤逻辑，`filter()` 比 `for` 循环更快，但**列表推导式**通常更具可读性：

  ```python
  # 等价写法
  odds = [x for x in numbers if x % 2 == 1]  # 列表推导式
  odds = list(filter(lambda x: x % 2 == 1, numbers))  # filter
  ```

---

### 常见用途总结

| 场景                 | 示例                                                   |
| -------------------- | ------------------------------------------------------ |
| 过滤满足条件的元素   | `filter(lambda x: x > 0, numbers)`                     |
| 移除空值或 None      | `filter(None, data)`                                   |
| 数据清洗             | `filter(lambda item: item.startswith('valid'), items)` |
| 与自定义判断函数结合 | `filter(is_valid, data)`                               |

---

### 与其他函数对比

| 函数       | 功能                           | 示例                            |
| ---------- | ------------------------------ | ------------------------------- |
| `map()`    | 对每个元素执行函数             | `map(str.upper, words)`         |
| `filter()` | 保留满足条件的元素             | `filter(lambda x: x > 0, nums)` |
| `reduce()` | 累积计算（需导入 `functools`） | `reduce(operator.add, nums)`    |
| 列表推导式 | 更 Pythonic 的替代方案         | `[x for x in nums if x > 0]`    |

---

### 总结

- `filter()` 适合“**筛选符合条件的元素**”的场景。
- 通常配合 `lambda` 或自定义函数使用。
- 与列表推导式相比，`filter()` 在处理大型数据集时更省内存，但可读性略低。
- 记住：返回的是迭代器，需用 `list()` 或循环来查看结果。

---



## zip

`zip()` 是一个内置函数，用于**将多个可迭代对象（如列表、元组等）“打包”成一个由元组组成的迭代器**。它会按位置将每个可迭代对象的元素组合在一起，直到最短的那个可迭代对象耗尽为止。

**基本语法**

```python
zip(iterable1, iterable2, ..., iterableN)
```

- 接受任意数量的可迭代对象。
- 返回一个 **zip 对象**（惰性求值的迭代器）。
- 每次从每个可迭代对象中取一个元素，组成一个元组。

**示例 1：基本用法**

```python
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30, 35]

zipped = zip(names, ages)
print(list(zipped))
# 输出: [('Alice', 25), ('Bob', 30), ('Charlie', 35)]
```

------

 **示例 2：多个可迭代对象**

```python
a = [1, 2, 3]
b = ['x', 'y', 'z']
c = [10.0, 20.0, 30.0]

result = zip(a, b, c)
print(list(result))
# 输出: [(1, 'x', 10.0), (2, 'y', 20.0), (3, 'z', 30.0)]
```

------

 **示例 3：长度不一致时（以最短为准**）

```python
list1 = [1, 2, 3, 4]
list2 = ['a', 'b']

print(list(zip(list1, list2)))
# 输出: [(1, 'a'), (2, 'b')]
# 注意：4 和 3 被忽略，因为 list2 只有两个元素
```

> 如果你希望以最长的为准，并用填充值补齐，可以使用 `itertools.zip_longest()`。

 **示例 4：解压（“反 zip”）**

`zip()` 的逆操作可以用 `*` 解包实现：

```python
pairs = [('Alice', 25), ('Bob', 30), ('Charlie', 35)]

names, ages = zip(*pairs)
print(names)  # ('Alice', 'Bob', 'Charlie')
print(ages)   # (25, 30, 35)
```

这在数据处理中非常实用！

------

 **特点与注意事项**

- **惰性求值**：`zip()` 返回的是迭代器，不会立即生成所有结果，节省内存。

- **只能遍历一次**：和 `map()`、`filter()` 一样，zip 对象消费一次后就为空。

- **适用于并行遍历**：

  ```python
  for name, age in zip(names, ages):
      print(f"{name} is {age} years old")
  ```

 **常见用途**

| 场景                 | 说明                                  |
| -------------------- | ------------------------------------- |
| 并行遍历多个列表     | 避免用索引 `for i in range(len(...))` |
| 构造字典             | `dict(zip(keys, values))`             |
| 数据对齐             | 将来自不同来源但顺序对应的数据组合    |
| 矩阵转置（简单情况） | `list(zip(*matrix))`                  |

 **示例：构造字典**

```python
keys = ['name', 'age', 'city']
values = ['Alice', 25, 'New York']
person = dict(zip(keys, values))
print(person)
# {'name': 'Alice', 'age': 25, 'city': 'New York'}
```

 **示例：矩阵转置**

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6]
]
transposed = list(zip(*matrix))
print(transposed)
# [(1, 4), (2, 5), (3, 6)]
```



---





## lambda表达式

`lambda` 表达式是 Python 中用于创建**匿名函数**（即没有名字的函数）的一种简洁语法。它通常用于需要一个简单函数但又不想正式用 `def` 定义函数的场合，尤其常与 `map()`、`filter()`、`sorted()`、`reduce()` 等高阶函数配合使用。

------

### 基本语法

```python
lambda 参数: 表达式
```

- **参数**：可以有零个或多个（用逗号分隔）。
- **表达式**：只能是一个**单行表达式**（不能是语句，如 `print`、`return`、`if` 块等）。
- 整个 `lambda` 表达式会**自动返回表达式的计算结果**。

------

### 示例 1：最简单的 lambda

```python
# 定义一个加法匿名函数
add = lambda x, y: x + y
print(add(3, 5))  # 输出: 8
```

这等价于：

```python
def add(x, y):
    return x + y
```

------

### 示例 2：与 `map()` 配合使用

将列表中每个数平方：

```python
numbers = [1, 2, 3, 4]
squared = list(map(lambda x: x ** 2, numbers))
print(squared)  # [1, 4, 9, 16]
```

------

### 示例 3：与 `filter()` 配合使用

筛选出偶数：

```python
nums = [1, 2, 3, 4, 5, 6]
evens = list(filter(lambda x: x % 2 == 0, nums))
print(evens)  # [2, 4, 6]
```

------

### 示例 4：与 `sorted()` 配合使用（自定义排序）

按元组第二个元素排序：

```python
pairs = [('apple', 3), ('banana', 1), ('cherry', 2)]
sorted_pairs = sorted(pairs, key=lambda x: x[1])
print(sorted_pairs)  # [('banana', 1), ('cherry', 2), ('apple', 3)]
```

------

### 注意事项

1. **只能包含表达式，不能包含语句**
    ❌ 错误示例：

   ```python
   f = lambda x: print(x)  # 虽然能运行，但不推荐（print 是语句，但在 Python 中是函数）
   f = lambda x: if x > 0: return x  # 语法错误！不能用 if/else 语句块
   ```

   ✅ 正确写法（使用条件表达式）：

   ```python
   f = lambda x: x if x > 0 else -x
   ```

2. **可读性优先**
    如果逻辑复杂（比如多行、嵌套条件），建议使用普通 `def` 函数，而不是强行塞进 `lambda`。

3. **lambda 不是必须的**
    很多时候可以用**列表推导式**或命名函数替代，代码更清晰。

---







## *解包

在 Python 中，`*`（星号）被称为**解包操作符**（unpacking operator），它可以将**可迭代对象**（如列表、元组、字符串等）“展开”成独立的元素。它常用于函数调用、变量赋值、合并序列等多种场景。

---

### 一、在函数调用中解包参数

最常见的用途是将一个序列（如列表或元组）作为**多个独立参数**传递给函数。

 **示例 1：传递多个参数**

```python
def greet(name, age, city):
    print(f"Hello {name}, you are {age} years old and live in {city}")

info = ["Alice", 25, "Paris"]
greet(*info)  # 等价于 greet("Alice", 25, "Paris")
```

> `*info` 将列表 `["Alice", 25, "Paris"]` 展开为三个独立参数。

 **示例 2：与 zip 结合（前面提到过）**

```python
pairs = [('Alice', 25), ('Bob', 30)]
names, ages = zip(*pairs)  # 解包 pairs
print(names)  # ('Alice', 'Bob')
print(ages)   # (25, 30)
```

---

### 二、在变量赋值中解包

可以用来快速分离序列中的元素。

 **示例 3：解包首尾元素**

```python
numbers = [1, 2, 3, 4, 5]
first, *middle, last = numbers
print(first)   # 1
print(middle)  # [2, 3, 4]
print(last)    # 5
```

> `*middle` 收集中间的所有元素（即使为空）。

 **示例 4：交换变量（常见技巧）**

```python
a, b = 10, 20
a, b = b, a  # 等价于 a, b = *(20, 10)
print(a, b)  # 20 10
```

---

### 三、合并序列

`*` 可以用来合并列表、元组等。

 **示例 5：合并列表**

```python
list1 = [1, 2, 3]
list2 = [4, 5, 6]
merged = [*list1, *list2]
print(merged)  # [1, 2, 3, 4, 5, 6]
```

 **示例 6：合并字符串（转换为字符列表）**

```python
chars = [*"abc", *"xyz"]
print(chars)  # ['a', 'b', 'c', 'x', 'y', 'z']
```

---

### 四、在函数定义中收集参数（`*args`）

`*` 也可用于函数定义中，将多余的参数收集为元组。

```python
def my_sum(*args):
    return sum(args)

print(my_sum(1, 2, 3, 4))  # 10
```

> `args` 是一个元组：`(1, 2, 3, 4)`

---

### 五、`**` 解包字典（扩展知识）

还有一个类似的 `**` 操作符，用于解包字典键值对作为关键字参数：

```python
def introduce(name, age):
    print(f"I'm {name}, {age} years old")

person = {"name": "Alice", "age": 25}
introduce(**person)  # 等价于 introduce(name="Alice", age=25)
```

> `**person` 将字典解包为 `name="Alice", age=25`。

---

### 注意事项

1. **只能解包可迭代对象**：`*5` 会报错，因为整数不可迭代。
2. **解包顺序**：解包时按顺序展开，例如 `func(1, *list, 2)` 会先传 `1`，然后是 `list` 中的每个元素，最后是 `2`。
3. **与 `zip`、`map` 等结合**：常见用法，如 `zip(*matrix)` 实现矩阵转置。

---

### 常见用途总结

| 场景                 | 示例                   |
| -------------------- | ---------------------- |
| 函数调用时展开参数   | `func(*args)`          |
| 合并序列             | `[*list1, *list2]`     |
| 分离首尾元素         | `first, *rest = seq`   |
| 矩阵转置             | `zip(*matrix)`         |
| 收集多余参数         | `def func(*args): ...` |
| 解包字典为关键字参数 | `func(**kwargs)`       |

---

### 总结

- `*` 解包操作符让 Python 代码更灵活、简洁。
- 它是实现“动态参数传递”和“序列操作”的强大工具。
- 记住：`*` 用于序列（列表、元组），`**` 用于字典。



# 高级概念



## 1.闭包

**闭包（Closure）** 是 Python（以及许多其他编程语言）中一个非常重要的概念，它指的是：

> **一个内部函数“记住”并能够访问其外部作用域中的变量，即使外部函数已经执行完毕。**

**1. 示例：**

```python
def outer(x):
    def inner(y):
        return x + y      # inner 使用了外部函数 outer 的变量 x
    return inner          # 返回 inner 函数本身（不是调用）

# 创建闭包
add_10 = outer(10)   # outer 执行完毕，x=10 已“消失”
print(add_10(5))     # 输出 15！inner 仍然能访问 x=10
```

这里：

- `inner` 是一个**内部函数**
- 它引用了**非局部变量 `x`**（来自 `outer`）
- 即使 `outer` 已经返回，`inner` 依然“绑定了” `x=10`
- 这个组合（`inner` 函数 + 被捕获的变量 `x`）就叫 **闭包**



**2.闭包的必要条件：**

- **存在嵌套函数**（函数 inside 函数）
- **内部函数引用了外部函数的变量**（自由变量）
- **外部函数返回了内部函数**（或以某种方式让内部函数在外部被调用）



**3.查看闭包内容：**

你可以通过 `__closure__` 属性查看闭包捕获的变量：

```python
print(add_10.__closure__)        # (<cell at 0x...>,)
print(add_10.__closure__[0].cell_contents)  # 10
```



**4. lambda与闭包**:

✅ lambda作为闭包：

```python
def make_adder(x):
    return lambda y: x + y   # lambda 捕获了 x

add_5 = make_adder(5)
print(add_5(3))  # 输出 8
```

这里：

- `lambda y: x + y` 是一个匿名函数
- 它引用了外层 `make_adder` 的参数 `x`
- 即使 `make_adder` 返回后，`x=5` 仍被保留
- **这就是一个由 `lambda` 构成的闭包！**

❌ 错误示例：
```python
funcs = []
for i in range(3):
    funcs.append(lambda: i)   # 所有 lambda 共享同一个 i

for f in funcs:
    print(f())  # 输出：2, 2, 2 （不是 0,1,2！）
```

**原因**：
 所有 `lambda` 都引用了**同一个变量 `i`**，而 `i` 在循环结束后值为 `2`。
 这不是“每个 lambda 记住当时的 i”，而是“都指向同一个 i”。

✅ 正确做法：用默认参数“固化”当前值:
```python
funcs = []
for i in range(3):
    funcs.append(lambda x=i: x)   # 利用默认参数捕获当前 i

for f in funcs:
    print(f())  # 输出：0, 1, 2 ✅
```


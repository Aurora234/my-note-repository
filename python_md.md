## numpy

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
## pandas
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


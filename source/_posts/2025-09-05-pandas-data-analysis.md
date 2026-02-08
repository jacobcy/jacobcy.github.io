---
title: 用Python做数据分析：Pandas实战指南
date: 2025-09-05 15:00:00
updated: 2025-09-05 15:00:00
description: 'Pandas数据分析完全指南，从数据清洗、处理到可视化的完整实战，包含量化投资、商业分析等真实案例'
keywords: 'Python,Pandas,数据分析,数据清洗,量化分析,数据可视化,DataFrame,实战教程'
tags: 
  - Python
  - Pandas
  - 数据分析
  - 量化分析
categories:
  - 技术笔记
author: Yi Chen
toc: true
comments: true
---

> 🐍 "Pandas是Python数据科学的基石。"

## 🎯 为什么学Pandas

作为一个程序员+投资者，我每天都在和数据打交道：
- 股票价格数据
- 投资组合表现
- 网站访问统计
- 咖啡冲泡记录

**Pandas让这一切变得简单**。

## 🚀 快速入门

### 安装

```bash
pip install pandas numpy matplotlib seaborn
```

### 核心数据结构

```python
import pandas as pd
import numpy as np

# 1. Series - 一维数据
s = pd.Series([1, 3, 5, np.nan, 6, 8])
print(s)

# 2. DataFrame - 二维数据
df = pd.DataFrame({
    'A': [1, 2, 3, 4],
    'B': ['a', 'b', 'c', 'd'],
    'C': [1.0, 2.0, 3.0, 4.0]
})
print(df)
```

## 💡 实战案例1：股票数据分析

### 获取数据

```python
import akshare as ak
import pandas as pd

# 获取贵州茅台历史数据
df = ak.stock_zh_a_hist(
    symbol="600519",
    period="daily",
    start_date="20230101",
    end_date="20241231"
)

# 查看数据结构
print(df.head())
print(df.info())
```

### 数据清洗

```python
# 重命名列（英文，方便后续处理）
df = df.rename(columns={
    '日期': 'date',
    '开盘': 'open',
    '收盘': 'close',
    '最高': 'high',
    '最低': 'low',
    '成交量': 'volume'
})

# 转换日期格式
df['date'] = pd.to_datetime(df['date'])
df.set_index('date', inplace=True)

# 检查缺失值
print(df.isnull().sum())

# 删除缺失值
df = df.dropna()

# 数据类型转换
df['volume'] = df['volume'].astype(int)
```

### 计算技术指标

```python
# 移动平均线
df['MA5'] = df['close'].rolling(window=5).mean()
df['MA20'] = df['close'].rolling(window=20).mean()
df['MA60'] = df['close'].rolling(window=60).mean()

# 涨跌幅
df['returns'] = df['close'].pct_change()

# 波动率（20日）
df['volatility'] = df['returns'].rolling(window=20).std() * np.sqrt(252)

# RSI指标
def calculate_rsi(prices, window=14):
    delta = prices.diff()
    gain = (delta.where(delta > 0, 0)).rolling(window=window).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(window=window).mean()
    rs = gain / loss
    rsi = 100 - (100 / (1 + rs))
    return rsi

df['RSI'] = calculate_rsi(df['close'])

print(df.tail())
```

### 数据分析

```python
# 基础统计
print("基础统计：")
print(df['returns'].describe())

# 年化收益
total_return = (df['close'].iloc[-1] / df['close'].iloc[0]) - 1
years = (df.index[-1] - df.index[0]).days / 365.25
annual_return = (1 + total_return) ** (1/years) - 1
print(f"\n年化收益率: {annual_return:.2%}")

# 最大回撤
cumulative = (1 + df['returns']).cumprod()
rolling_max = cumulative.expanding().max()
drawdown = (cumulative - rolling_max) / rolling_max
max_drawdown = drawdown.min()
print(f"最大回撤: {max_drawdown:.2%}")

# 夏普比率（假设无风险利率2%）
excess_return = df['returns'].mean() * 252 - 0.02
sharpe = excess_return / (df['returns'].std() * np.sqrt(252))
print(f"夏普比率: {sharpe:.2f}")
```

### 可视化

```python
import matplotlib.pyplot as plt
import seaborn as sns

# 设置样式
plt.style.use('seaborn-v0_8')
sns.set_palette("husl")

# 创建子图
fig, axes = plt.subplots(3, 1, figsize=(12, 10))

# 图1：价格与均线
axes[0].plot(df.index, df['close'], label='Close', linewidth=1)
axes[0].plot(df.index, df['MA5'], label='MA5', alpha=0.7)
axes[0].plot(df.index, df['MA20'], label='MA20', alpha=0.7)
axes[0].set_title('Stock Price with Moving Averages')
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# 图2：收益率分布
axes[1].hist(df['returns'].dropna(), bins=50, alpha=0.7, edgecolor='black')
axes[1].set_title('Returns Distribution')
axes[1].set_xlabel('Daily Returns')
axes[1].axvline(df['returns'].mean(), color='red', linestyle='--', label='Mean')
axes[1].legend()

# 图3：累计收益
cumulative_returns = (1 + df['returns']).cumprod()
axes[2].plot(df.index, cumulative_returns)
axes[2].set_title('Cumulative Returns')
axes[2].grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

## 💡 实战案例2：投资组合分析

### 多资产数据获取

```python
# 获取多只股票数据
stocks = ['600519', '000858', '000333', '000002']  # 茅台、五粮液、美的、万科
stock_names = ['茅台', '五粮液', '美的', '万科']

data = {}
for symbol, name in zip(stocks, stock_names):
    df = ak.stock_zh_a_hist(symbol=symbol, period="daily", 
                           start_date="20230101", end_date="20241231")
    df['date'] = pd.to_datetime(df['日期'])
    df.set_index('date', inplace=True)
    data[name] = df['收盘']

# 合并成DataFrame
prices = pd.DataFrame(data)
prices = prices.dropna()
print(prices.head())
```

### 收益与风险分析

```python
# 计算收益率
returns = prices.pct_change().dropna()

# 年化收益
annual_returns = returns.mean() * 252
print("年化收益：")
print(annual_returns)

# 年化波动率
annual_volatility = returns.std() * np.sqrt(252)
print("\n年化波动率：")
print(annual_volatility)

# 相关性矩阵
correlation = returns.corr()
print("\n相关性矩阵：")
print(correlation)

# 可视化相关性
plt.figure(figsize=(8, 6))
sns.heatmap(correlation, annot=True, cmap='coolwarm', center=0,
            square=True, fmt='.2f')
plt.title('Stock Returns Correlation')
plt.show()
```

### 投资组合优化

```python
# 计算协方差矩阵
cov_matrix = returns.cov() * 252

# 等权重组合
weights = np.array([0.25, 0.25, 0.25, 0.25])
portfolio_return = np.sum(annual_returns * weights)
portfolio_volatility = np.sqrt(np.dot(weights.T, np.dot(cov_matrix, weights)))
sharpe_ratio = (portfolio_return - 0.02) / portfolio_volatility

print(f"等权重组合：")
print(f"预期年化收益: {portfolio_return:.2%}")
print(f"年化波动率: {portfolio_volatility:.2%}")
print(f"夏普比率: {sharpe_ratio:.2f}")

# 绘制有效前沿
from scipy.optimize import minimize

def portfolio_performance(weights, returns, cov_matrix):
    port_return = np.sum(returns * weights)
    port_volatility = np.sqrt(np.dot(weights.T, np.dot(cov_matrix, weights)))
    return port_return, port_volatility

def minimize_volatility(weights, returns, cov_matrix):
    return portfolio_performance(weights, returns, cov_matrix)[1]

# 生成随机组合
num_portfolios = 1000
results = np.zeros((3, num_portfolios))
weight_array = []

for i in range(num_portfolios):
    weights = np.random.random(4)
    weights /= np.sum(weights)
    weight_array.append(weights)
    
    port_return, port_volatility = portfolio_performance(weights, annual_returns, cov_matrix)
    results[0, i] = port_volatility
    results[1, i] = port_return
    results[2, i] = (port_return - 0.02) / port_volatility

# 绘制
plt.figure(figsize=(10, 6))
plt.scatter(results[0, :], results[1, :], c=results[2, :], 
           cmap='YlOrRd', marker='o', s=10, alpha=0.5)
plt.colorbar(label='Sharpe Ratio')
plt.xlabel('Volatility')
plt.ylabel('Expected Return')
plt.title('Efficient Frontier')
plt.grid(True, alpha=0.3)
plt.show()
```

## 💡 实战案例3：数据清洗技巧

### 处理缺失值

```python
import pandas as pd
import numpy as np

# 创建示例数据
df = pd.DataFrame({
    'A': [1, 2, np.nan, 4, 5],
    'B': ['a', 'b', 'c', np.nan, 'e'],
    'C': [1.0, 2.0, 3.0, 4.0, 5.0]
})

# 查看缺失值
print(df.isnull().sum())

# 填充缺失值
df['A'].fillna(df['A'].mean(), inplace=True)  # 用均值填充
df['B'].fillna('unknown', inplace=True)  # 用特定值填充

# 删除缺失值
df_clean = df.dropna()  # 删除包含缺失值的行
df_clean = df.dropna(subset=['A', 'B'])  # 只在特定列检查
```

### 数据类型转换

```python
# 字符串转日期
df['date'] = pd.to_datetime(df['date_string'])

# 字符串转数值
df['numeric'] = pd.to_numeric(df['string_number'], errors='coerce')

# 分类变量
df['category'] = df['category'].astype('category')

# 提取日期特征
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day_of_week'] = df['date'].dt.dayofweek
df['quarter'] = df['date'].dt.quarter
```

### 数据合并

```python
# 创建示例数据
df1 = pd.DataFrame({'key': ['A', 'B', 'C'], 'value1': [1, 2, 3]})
df2 = pd.DataFrame({'key': ['B', 'C', 'D'], 'value2': [4, 5, 6]})

# 不同合并方式
inner = pd.merge(df1, df2, on='key', how='inner')  # 交集
outer = pd.merge(df1, df2, on='key', how='outer')  # 并集
left = pd.merge(df1, df2, on='key', how='left')   # 保留左表
right = pd.merge(df1, df2, on='key', how='right')  # 保留右表

# 按索引合并
df_merged = pd.concat([df1, df2], axis=1)  # 横向合并
df_stacked = pd.concat([df1, df2], axis=0)  # 纵向合并
```

## 🎯 性能优化技巧

### 1. 使用向量化操作

```python
# 慢：循环
result = []
for i in range(len(df)):
    result.append(df.iloc[i]['A'] + df.iloc[i]['B'])

# 快：向量化
df['result'] = df['A'] + df['B']
```

### 2. 使用Categorical类型

```python
# 如果有大量重复字符串，转换为category
df['category'] = df['category'].astype('category')

# 内存使用减少80%+
```

### 3. 分块处理大文件

```python
# 处理大CSV文件
chunk_size = 10000
chunks = []

for chunk in pd.read_csv('large_file.csv', chunksize=chunk_size):
    # 处理每个chunk
    processed = chunk.process()
    chunks.append(processed)

# 合并结果
df = pd.concat(chunks)
```

### 4. 使用query和eval

```python
# 比标准索引快
df.query('A > 0 and B < 100')

# 复杂计算
df.eval('C = A + B')
```

## 📊 常用代码模板

### 模板1：数据探索

```python
def explore_data(df):
    """快速探索数据集"""
    print("数据集形状:", df.shape)
    print("\n数据类型:")
    print(df.dtypes)
    print("\n基础统计:")
    print(df.describe())
    print("\n缺失值:")
    print(df.isnull().sum())
    print("\n重复行数:", df.duplicated().sum())
```

### 模板2：时间序列处理

```python
def resample_data(df, freq='M'):
    """重采样时间序列数据"""
    return df.resample(freq).agg({
        'open': 'first',
        'high': 'max',
        'low': 'min',
        'close': 'last',
        'volume': 'sum'
    })
```

### 模板3：分组统计

```python
def group_analysis(df, group_col, value_col):
    """分组统计分析"""
    return df.groupby(group_col)[value_col].agg([
        'count', 'mean', 'std', 'min', 'max'
    ]).round(2)
```

---

## 💭 写在最后

Pandas是数据分析师的"瑞士军刀"。

掌握它，你可以：
- 处理任何结构化数据
- 快速进行数据清洗和转换
- 生成专业级的分析报告

**学习建议**：
1. 先掌握基础（DataFrame、Series）
2. 通过实战项目练习
3. 遇到问题查文档，不要死记API
4. 关注性能优化（大数据场景）

> "数据是新时代的石油，Pandas是炼油厂。"

希望这篇指南对你有帮助！有任何问题，欢迎交流。🐼

---

*写于 2025年9月5日 | Pandas实战总结*

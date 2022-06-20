# jupyter note

## 查看基本信息

```python
!head data/sales_data.csv # basic csv info

sales = pd.read_csv('data/sales_data.csv', parse_dates=['Date'])
sales.head() # head of frames

sales.shape() # row number and column number
sales.info() # data type of columns
sales.describe() # baseic data info like count mean, std, min. max

sales['Unit_Cost'].describe() # analyze the `Unit_Cost` column

# 查看各 column 之间的显著关系
corr = sales.corr()

corr

## 绘关系图
fig = plt.figure(figsize=(8,8))
plt.matshow(corr, cmap='RdBu', fignum=fig.number)
plt.xticks(range(len(corr.columns)), corr.columns, rotation='vertical');
plt.yticks(range(len(corr.columns)), corr.columns);
```


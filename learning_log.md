# 学习日志 - 电商用户行为分析项目

## 2026年4月20日（第一天）：SQL 数据提取

### 今日任务
- 注册 Kaggle、GitHub、Tableau Public 账号
- 下载 Online Retail II 数据集
- 在 sqliteonline.com 中导入 CSV 文件
- 编写并运行每日销售汇总 SQL
- 导出 daily_sales.csv

### 遇到的问题及解决方法

**1. 导入 CSV 后列名变成 c1~c8，数字列无法计算**  
- 原因：在线 SQL 工具自动命名列，且所有字段默认识别为文本。  
- 解决：查询时使用 c1~c8 代替原列名，用 `CAST(c4 AS INTEGER)` 和 `CAST(c6 AS REAL)` 转换数据类型后再相乘。

**2. 日期列提取时包含了小时**  
- 现象：`substr(c5, 1, 10)` 得到 `1/10/11 10`（带小时）。  
- 原因：日期长度不固定（如 `1/10/11 10:26` 和 `2012/1/10 8:26`），固定截取10个字符会切到小时。  
- 解决：改用 `substr(c5, 1, instr(c5, ' ') - 1)`，先找到空格位置，截取空格前的所有字符。

**3. 销售额出现很多小数位（如 32660.17000000002）**  
- 原因：SQLite 的 REAL 浮点数计算精度问题。  
- 解决：用 `ROUND(SUM(...), 2)` 保留两位小数。

**4. 发现 c4 列有负数**  
- 理解：负数表示退货记录。每日销售汇总中保留负数可反映净销售额（真实收入）。

### 需要记住的知识点
- SQL 核心：`SELECT`, `FROM`, `WHERE`, `GROUP BY`, `ORDER BY`
- 聚合函数：`COUNT(DISTINCT ...)`, `SUM`, `MAX`, `ROUND`
- 字符串处理：`substr`, `instr` 用于提取日期部分
- 数据类型转换：`CAST(column AS INTEGER/REAL)`
- 过滤空值：`WHERE c7 IS NOT NULL AND c7 != ''`（NULL 和空字符串不同）

### 今日产出
- `daily_sales.csv`：每日销售汇总（sale_date, order_count, total_revenue）

---

## 2026年4月21日（第二天）：SQL RFM 基础表 + Python 分析

### 上午：SQL 用户 RFM 基础表

**任务**：计算每个客户的最近购买时间、购买次数、总金额。

**最终 SQL 代码**：
```sql
SELECT 
    c7 AS customer_id,
    MAX(c5) AS last_purchase,
    COUNT(DISTINCT c1) AS frequency,
    ROUND(SUM(CAST(c4 AS INTEGER) * CAST(c6 AS REAL)), 2) AS monetary
FROM online_retail_II
WHERE c7 IS NOT NULL AND c7 != ''
  AND CAST(c4 AS INTEGER) > 0
GROUP BY c7;
```

**学到的新知识点**：
- `MAX(c5)` 获取每个客户最晚的日期时间
- 过滤退货：`Quantity > 0`
- 导出为 `user_rfm.csv`

### 下午：Python 数据分析（和鲸社区）

**任务**：读取 CSV，清洗数据，转换日期，画趋势图，RFM 分层。

#### 遇到的问题及解决方法

**1. 文件找不到（FileNotFoundError）**  
- 原因：CSV 文件上传到了 `/home/mw/` 目录，而当前工作目录是 `/home/mw/project`。  
- 解决：使用 `os.listdir('/home/mw/')` 找到文件，然后用绝对路径读取，或将文件复制到当前目录。

**2. 日期转换报错：time data '1/18/11 10:01' does not match format '%m/%d/%Y %H:%M'**  
- 原因：数据中年份是两位（`11`），而格式用了四位年份 `%Y`。  
- 解决：将格式改为 `'%m/%d/%y %H:%M'`，并用 `errors='coerce'` 处理异常值。

**3. 画图时横轴显示数字而不是日期**  
- 原因：日期列还是字符串类型，没有转换为 datetime。  
- 解决：先执行 `pd.to_datetime()` 转换，再 `sort_values()` 排序，最后画图。

**4. RFM 打分时 qcut 报错“Bin edges must be unique”**  
- 原因：数据中有大量相同的 frequency 或 monetary 值，无法生成唯一的分位数边界。  
- 解决：使用 `rank(method='first')` 先排名，再分档。

#### Python 核心代码片段

```python
# 读取数据
daily = pd.read_csv('daily_sales.csv')
daily = daily.dropna(subset=['sale_date'])
daily['sale_date'] = pd.to_datetime(daily['sale_date'], format='%m/%d/%y')
daily = daily.sort_values('sale_date')

# 画图
import matplotlib.pyplot as plt
plt.figure(figsize=(12,6))
plt.plot(daily['sale_date'], daily['total_revenue'])
plt.title('Daily Total Revenue')
plt.show()

# RFM 打分
def rfm_score_r(x):
    return pd.qcut(x.rank(method='first'), 3, labels=[3,2,1])
def rfm_score_fm(x):
    return pd.qcut(x.rank(method='first', ascending=False), 3, labels=[3,2,1])

rfm['R'] = rfm_score_r(rfm['recency'])
rfm['F'] = rfm_score_fm(rfm['frequency'])
rfm['M'] = rfm_score_fm(rfm['monetary'])
rfm['RFM_Score'] = rfm['R'].astype(str) + rfm['F'].astype(str) + rfm['M'].astype(str)
```

#### 需要记住的 Python 知识点
- pandas 读取 CSV：`pd.read_csv()`
- 删除缺失行：`dropna(subset=['列名'])`
- 日期转换：`pd.to_datetime(..., format='...')`
- 排序：`sort_values()`
- 画图：`matplotlib.pyplot` 的基本用法
- RFM 分层：使用 `rank()` + `qcut()` 解决重复值问题
- 导出 CSV：`to_csv()`

### 项目结论与业务理解
- **高价值用户（R=3,F=3,M=3）**：共 367 人，贡献了 23.89% 的销售额。  
- **业务建议**：  
  - 针对高价值用户提供专属折扣或 VIP 服务，提升忠诚度。  
  - 对近期未购买但历史消费高的用户（R低，F/M高）发送唤醒优惠券。  
  - 销售趋势图中的峰值日期可进一步分析对应营销活动。

### 今日产出
- `user_rfm.csv`（RFM 基础表）
- `rfm_with_scores.csv`（带 RFM 分数的完整结果）
- 销售额趋势图截图
- 本学习日志

---

## 项目整体反思

### 我通过这个项目掌握了什么？

1. **SQL 能力**  
   - 从 CSV 导入数据库，使用聚合函数、字符串处理、类型转换提取业务指标。  
   - 理解了日期格式混乱时的处理方法（`substr` + `instr`）。

2. **Python 数据分析**  
   - 使用 pandas 进行数据清洗、日期转换、排序。  
   - 使用 matplotlib 画趋势图。  
   - 实现 RFM 用户分层，解决分位数重复问题。

3. **业务思维**  
   - 知道 RFM 模型的业务含义：Recency 越近越好，Frequency 越高越好，Monetary 越高越好。  
   - 能根据分析结果提出可落地的运营建议。

4. **工具协作**  
   - SQL 负责数据提取和聚合，Python 负责复杂计算和可视化。  
   - 能够独立完成从原始数据到业务洞察的全流程。

总结：
我做了一个电商用户行为分析项目。先用 SQL 从 54 万条订单中提取了每日销售趋势和用户 RFM 基础表，然后用 Python 进行数据清洗、日期转换、可视化和 RFM 分层。发现高价值用户虽然只占 367 人，却贡献了近 24% 的销售额。基于这个发现，我建议运营团队对这类用户发放专属优惠券，同时针对流失的高价值用户设计唤醒活动。这个项目锻炼了我的 SQL 和 Python 实战能力，也让我理解了数据分析如何驱动业务决策。

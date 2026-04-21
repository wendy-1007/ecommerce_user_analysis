# 电商用户行为分析项目

> 基于 Online Retail II 数据集，使用 SQL 进行数据提取与聚合，使用 Python (pandas, matplotlib) 进行用户行为分析与 RFM 分层，识别高价值用户并给出业务建议。

## 项目背景

本项目模拟电商平台数据分析师的工作，对真实的零售交易数据（54 万条订单记录）进行清洗、分析和可视化，旨在：

- 掌握每日销售趋势
- 对用户进行价值分层（RFM）
- 提出可落地的运营策略

## 工具与技能

| 工具 | 用途 |
|------|------|
| SQLiteOnline | 编写 SQL 查询，提取每日销售汇总、用户 RFM 基础表 |
| Python (pandas, matplotlib) | 数据清洗、日期处理、RFM 打分、可视化 |
| GitHub | 版本控制与作品集展示 |

## 分析流程

### 1. SQL 数据提取

使用 SQL 从原始订单表中计算两个核心表：

- **每日销售汇总**：按天统计订单数和销售额  
  [查看 SQL 代码](sql/daily_sales.sql)

- **用户 RFM 基础表**：每个客户的购买次数、总金额、最近购买时间  
  [查看 SQL 代码](sql/user_rfm.sql)

> 由于原始日期格式不统一（`1/4/11 10:26` 和 `2012/1/10 8:26` 混合），SQL 中使用 `substr` + `instr` 提取空格前的日期部分，确保准确性。

### 2. Python 数据清洗与可视化

- 读取 CSV 文件，删除无效行（日期为空）
- 将文本日期转换为标准 `datetime` 格式，并按时间排序
- 绘制每日销售额趋势图

**销售额趋势图（示例）**  
<img width="856" height="424" alt="image" src="https://github.com/user-attachments/assets/3f3d0769-d9c3-4561-9124-bb550b5931ed" />

*说明：X 轴为日期（2011‑01 至 2011‑12），Y 轴为每日总销售额（英镑）。从图中可以看出 11 月份销售额明显上升，可能存在促销活动或节日效应。*

### 3. RFM 用户分层

- **R (Recency)**：最近一次购买距离数据集最后一天（2011‑12‑09）的天数
- **F (Frequency)**：购买次数（订单数）
- **M (Monetary)**：总消费金额

使用分位数（`qcut`）将每个指标分为 3 档：
- 3 分 = 高价值
- 2 分 = 中价值
- 1 分 = 低价值

最终组合成 `RFM_Score`（例如 `333` 表示最高价值用户）。

## 关键结论

| 指标 | 数值 |
|------|------|
| 高价值用户数量（R=3,F=3,M=3） | 367 人 |
| 高价值用户贡献的销售额占比 | 23.89% |

**业务建议**：
- 针对高价值用户（RFM_Score = 333）提供专属折扣或 VIP 服务，提升忠诚度。
- 对于近期未购买但历史消费高的用户（R 低，F/M 高），可发送唤醒优惠券。
- 销售趋势图中的峰值日期可进一步分析对应营销活动，作为成功经验复制。

## 文件结构
ecommerce_user_analysis/
├── README.md
├── learning_log.md
├── data/
│   ├── daily_sales.csv
│   ├── user_rfm.csv
│   └── rfm_with_scores.csv
├── sql/
│   ├── daily_sales.sql
│   └── user_rfm.sql
├── notebooks/
│   └── ecommerce_analysis.ipynb
└── images/
    └── daily_revenue_trend.png

## 如何复现

1. 从 [UCI](https://archive.ics.uci.edu/static/public/502/online+retail+ii.zip) 下载数据集并转换为 CSV。
2. 使用 SQLiteOnline 或本地 MySQL 导入 CSV，运行 `sql/` 下的两个 SQL 脚本，导出 CSV 文件。
3. 将 CSV 文件放入 `data/` 目录，使用 Jupyter Notebook（或和鲸社区）打开 `notebooks/ecommerce_analysis.ipynb`，按顺序执行所有单元格。
4. 运行后即可得到 RFM 分层结果和可视化图表。

## 学习日志

详细的项目踩坑记录和每日进展请查看 [`learning_log.md`](learning_log.md)。

## 作者

[胡悦]  
[wendy-1007](https://github.com/wendy-1007)| 2675868084@qq.com

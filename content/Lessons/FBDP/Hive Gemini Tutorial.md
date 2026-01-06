> [!Question] Gemini 老师出题：
> 既然你在复习，那我们来模拟一个**真实的电商考试场景**。这道题涵盖了 Hive 中最核心的三个考点：**聚合计算、空值处理和 Top N 排名**。
### 🚀 题目：用户消费排行榜

背景描述：

你有一张用户订单表 user_orders，包含以下字段：

- `user_id` (String): 用户唯一标识
    
- `order_amount` (Double): 订单金额（注意：有些金额可能是 `NULL`）
    
- `category` (String): 商品分类（如 'Electronics', 'Clothing'）
    

你的任务：

编写一段 HiveQL 语句，统计每个分类下，消费总额最高的前 3 名用户。

**具体要求：**

1. 如果 `order_amount` 为空，请将其视为 `0` 处理。
    
2. 输出字段包括：`category`（分类）、`user_id`（用户ID）、`total_spend`（该用户在该分类下的总消费额）。
    
3. 结果需要按分类分组，并按消费总额降序排列。
    

---

### 💡 复习小贴士（做题提示）：

- **处理 NULL**：记得用 `NVL()` 或 `COALESCE()`。
    
- **第一步**：你需要先用 `GROUP BY` 算出每个用户在每个分类下的总消费额。
    
- **第二步**：使用**窗口函数** `row_number()` 或 `rank()` 来进行组内排名。
    
- **嵌套**：通常需要用到子查询。
    

---

**你可以先把你的 SQL 代码写在这里发给我，或者如果你卡住了，随时问我某一部分怎么写！需要我直接揭晓答案和解析吗？**


```sql
CREATE EXTERNAL IF NOT EXISTS user_orders(
	user_id STRING,
	order_amount DOUBLE,
	category STRING
)

SELECT 
	user_id,
	category,
	SUM(NVL(order_amount, 0)) AS totel_spend
FROM user_orders
GROUP BY user_id

```
如何过滤？


```sql
SELECT 
    user_id, 
    AVG(order_amount) AS avg_amount
FROM orders
WHERE dt >= '2023-01-01' AND dt <= '2023-12-31'  -- 1. 分区过滤（性能最关键！）
  AND order_amount > 500                         -- 2. 普通字段过滤
  AND user_id <> 'test_user'                     -- 3. 排除特定值
GROUP BY user_id
ORDER BY avg_amount DESC;
```

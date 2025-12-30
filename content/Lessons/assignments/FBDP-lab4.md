## Environment Setup

- Use docker to setup `spark:latest`, with R, python, scala,  java support.
 ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215154548.png)
 - Develop Environment: VS Code with docker plugin
 - Use Python Spark (`pyspark`)


- Prepare a `spark-submit` script:
```shell
bin/spark-submit \
  --master local[*] \
  --name "CouponStatisticsJob" \
  /opt/spark/work-dir/lab4/task1-1.py\
  /opt/spark/work-dir/lab4/data/ccf_online_stage1_train.csv
```

## Directory Tree
```
├── data			
│   ├── ccf_offline_stage1_test_revised.csv		# test set
│   ├── ccf_offline_stage1_train.csv			# offline train set
│   ├── ccf_online_stage1_train.csv				# online train set
│   └── sample_submission.csv
├── output
│   ├── submission
│   │   ├── _SUCCESS
│   │   └── part-00000-302f19c5-04ab-4a2d-b658-d992509ed5bb-c000.csv	# task 3
│   ├── task1-1
│   │   ├── _SUCCESS
│   │   ├── part-00000			# output of task 1 stage 1
│   │   └── part-00001
│   ├── task1-2-CouponCheck
│   │   ├── _SUCCESS
│   │   ├── part-00000		# output of task 1 stage 2
│   │   └── part-00001
│   ├── task2-1
│   │   ├── _SUCCESS
│   │   └── part-00000-a3023914-9395-482a-b149-95cab6c0d864-c000.csv # task 2-1
│   └── task2-2
│       ├── _SUCCESS
│       └── part-00000-642dc3f5-85db-43c9-9442-31cbea30252a-c000.csv # task 2-2
├── README.md		# the report- you are here!
├── tainchi.csv		# the copy file of ./output/submission/part-....
├── tainchi2.csv	# another try of test
├── task1-1.py		# the python script done task 1-1
├── task1-2.py		# the python script done task 1-2
├── task2-1.py		# the python script done task 2-1
├── task2-2.py		# the python script done task 2-2	
└── task3-main.py	# the python script done task 3 tianchi predict
```


## Task1 Spark RDD Programming

### Sub task 1
1. 统计优惠券发放数量： 使⽤ ` ccf_online_stage1_train` 统计每种优惠券的被使⽤次数，并
按数量降序排列输出，完整结果以附件形式给出，实验报告中给出前⼗名优惠券的结果。
输出格式：
```
<Coupon_id> <总使⽤次数>
```

#### Idea 
仍然是一个 `word_count` 类似的问题
用 RDD 的思想来说，我们每次处理 ` ccf_online_stage1_train` 的一行，就是对这个单元 RDD 做操作。
我们可以先 `map` 其做 `split`，将 csv 的一行处理成不同的 token
然后做 `filter` 筛选出符合使用了优惠券 (That is `Action = 2`) 的数据，写入新的 RDD
然后，在新的 RDD 中 `map` 成 `key, 1`, 然后 `reduceByKey` 方法是 `add`（累加出现次数）

最后再排序即可

#### Code
```python
def coupon_count(spark, input_path):
    """
        Use RDD to count each coupon times being used
    """

    sc = spark.sparkContext

    # read the data
    try:
        raw_rdd = sc.textFile(input_path)
    except Exceptoin as e:
        print(f"ERROR: fail to read the data path{input_path}")
        return

    header = raw_rdd.first()
    print(f"header is {header}")
    data_rdd = raw_rdd.filter(lambda row: row!=header)

    # count

    coupon_counts_rdd = (
        data_rdd 
        .map(lambda line : line.split(','))
        # oupon_id != null && Date != null
        .filter(lambda fields:
            # action: 2 means buying
            fields[2].strip() == '2' and
            # Coupon_id
            fields[3].strip() != 'null')
        # start mapping
        .map(lambda fields : 
            (fields[3], 1))
        .reduceByKey(lambda a,b : a+b)
    )

    sorted_rdd = (
        # here we only need map the elements apart
        coupon_counts_rdd
        # map: (count, key)
        # now we use count as key, so it can be sort by key easily
        .map(lambda x : (x[1], x[0]))
        .sortByKey(ascending=False)
    )

    top10 = sorted_rdd.take(10)
```

#### Result
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251216212104.png)

完整请看附件 

### Sub task 2
查询指定商家优惠券使⽤情况： 使⽤ ccf_online_stage1_train 统计每个商家的优惠券使⽤情况，分为负样本、普通消费和正样本三种，按照 Mechant_id 升序排序并将结果存储在新表 online_consumption_table 中，实验报告中给出前⼗⾏结果。

*注：如果Date=null & Coupon_id != null，该记录表示领取优惠券但没有使⽤，即负样本；如果Date!=null &Coupon_id = null，则表示普通消费⽇期；如果Date!=null & Coupon_id != null，则表示⽤优惠券消费⽇期，即正样本。*

输出格式：
```
<Mechant_id> <负样本数量> <普通消费数量> <正样本数量>
```

#### Idea
类似的，我们这次仍然对 RDD 做操作
不同的是，**这次我们有三种样本要标记**。如何做 filter 呢？
我们引出一个函数，做标记，然后返回一个 `<Mechant_id> <negtive> <normal> <positive>` 的元组即可了


```shell
bin/spark-submit \
  --master local[*] \
  --name "CouponCheckJob" \
  /opt/spark/work-dir/lab4/task1-2.py\
  /opt/spark/work-dir/lab4/data/ccf_online_stage1_train.csv
```


#### Code
**仅展示核心逻辑**，详细代码  please read the src
```python

def coupon_check(spark, input_path):
	
	# some code...
	
	coupon_check_rdd = (
    	data_rdd 
    	.map(lambda line : line.split(','))
    	# start mapping
    	.map(lambda fields : 
    	    map_consume_type(fields))
    	# reduce: add up each type, return a tuple
    	.reduceByKey(lambda a,b : (a[0]+b[0], a[1]+b[1], a[2]+b[2]))
	)


def map_consume_type(fields):
    '''
    *注: 如果Date=null & Coupon_id != null,该记录表示领取优惠券但没有使⽤,即负样本;
    如果Date!=null &Coupon_id = null,则表示普通消费⽇期;
    如果Date!=null & Coupon_id != null,则表示⽤优惠券消费⽇期,即正样本。
    *
    '''
    merchant_id = fields[1]
    date = fields[6]
    coupon_id = fields[3]

    count = (0, 0, 0)
    # -1 0 1
    
    # negtive
    if date is "null" and coupon_id is not "null":
        count = (1, 0, 0)
    # normal
    elif date is not "null" and coupon_id is "null":
        count = (0, 1, 0)
    # positive
    elif date is not "null" and coupon_id is not "null":
        count = (0, 0, 1)

    return (merchant_id, count)
```



#### Result
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251216221652.png)

For detailed result please see output directory


## Task 2 Spark SQL Programming
In this task, we need to use Spark SQL to do the job.

[参考文档](https://spark.apache.org/docs/latest/sql-getting-started.html)

### Sub task1
优惠券使⽤时间分布统计： 根据 ccf_offline_stage1_train 表中数据，统计每⼀种优惠券被使⽤时间位于⼀个⽉的上中下旬。给出每⼀种优惠券被使⽤时间的分布。
输出格式：
```
<Coupon_id> <上旬被使⽤概率> <中旬被使⽤概率> <下旬被使⽤概率>
```

#### Idea

提交代码如下

```
bin/spark-submit \
  --master local[*] \
  --name "CouponMonthStatJob" \
  /opt/spark/work-dir/lab4/task2-1.py\
  /opt/spark/work-dir/lab4/data/ccf_offline_stage1_train.csv
```

首先 init the table:
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251218121553.png)
可以看到，if `Date is not 'null'`, then last 2 bit is month
```
[1,10] 上旬
[11,20] 中旬
[21, ] 下旬
```

构造 SQL 查询指令即可



我们用 `sql` 实际返回了一个 `dataframe`, 由于惰性计算机制，只有调用 save 或者 show 的时候才开始计算。

#### Code
此处展示 SQL 查询关键代码
```python
def coupon_month(spark, input_path):
    sc = spark.sparkContext
    df = spark.read.csv(input_path, header=True)

    df.show()


    df.createOrReplaceTempView("offline_train")
    # prepare a SQL query

    sql_qurey = """
    WITH used_coupons AS (
        SELECT 
            Coupon_id, 
            CAST(SUBSTRING(Date,7,2) AS INT) as day
        FROM 
            offline_train
        WHERE Date IS NOT NULL AND Date != 'null' AND Coupon_id != 'null'
    ),
    count_table AS (
        SELECT
            Coupon_id,
            COUNT(*) as total,
            SUM(
                CASE 
                    WHEN day BETWEEN 1 AND 10 THEN 1 ELSE 0
                END
            ) as early_count,
            SUM(
                CASE
                    WHEN day BETWEEN 11 AND 20 THEN 1 ELSE 0
                END
            ) as mid_count,
            SUM(
                CASE
                    WHEN day BETWEEN 21 AND 31 THEN 1 ELSE 0
                END
            ) as late_count
        FROM used_coupons
        GROUP BY Coupon_id
    )

    SELECT
        Coupon_id,
        ROUND(early_count / total, 2) as early_prob,
        ROUND(mid_count / total, 2) as mid_prob,
        ROUND(late_count / total, 2) as late_prob
    FROM count_table
    """

    res = spark.sql(sql_qurey)

    res.show(10)
```


#### Result

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251218131643.png)



### Sub task2
> [!Question] 商家正样本比例统计
  根据 online_consumption_table 表中数据，按正样本⽐例对商家排序，给出正样本⽐例最⾼的前⼗个商家。
输出格式：
> ```
> <Merchant_id> <正样本⽐例> <正样本数量> <总样本数量>
> ```
#### Idea
仍然用 SQL 处理。
先构建表，然后做一个子表统计正比例，最后排序输出即可。

- init the table
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251218133035.png)


- 如果Date!=null & Coupon_id != null，则表示⽤优惠券消费⽇期，即正样本。

启动命令：
```shell
bin/spark-submit \
  --master local[*] \
  --name "CouponPosiStat" \
  /opt/spark/work-dir/lab4/task2-2.py\
  /opt/spark/work-dir/lab4/data/ccf_online_stage1_train.csv
```
#### Code
框架完全类似，这里附上 SQL 查询代码

```sql
WITH count_table AS(
	SELECT 
		Merchant_id,
		COUNT(*) as total,
		SUM(
			CASE 
				WHEN Date != 'null' AND Coupon_id != 'null' THEN 1 ELSE 0
			END
		) as posi_count
	FROM online_train
	GROUP BY Merchant_id
)

--- caclulate the prob

SELECT
	Merchant_id,
	ROUND(posi_count/total,2) as posi_prob
FROM
	count_table
ORDER BY posi_prob|DESC
```
其中 `DESC` 是排序降序关键字

#### Result
取十行
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251218134753.png)


## Task 3 Spark MLlib Programming

本任务是[天池新人实战赛o2o优惠券使用预测任务](https://tianchi.aliyun.com/competition/entrance/231593/information)
可以尝试决策树模型或者Logistic回归



### Idea
```shell
bin/spark-submit \
  --master local[*] \
  --name "CouponPredict" \
  /opt/spark/work-dir/lab4/task3-main.py
```

提交指令如上。

我使用随机森林 (RandomForest) 模型来完成这个分类预测任务。

使用 Spark ML, 搭建 Pipeline 来完成预处理、训练和预测任务。

我选取的特征有：

- **`discount_rate_val` (折扣率)**：
    - 通过 `parse_discount` 函数解析。将满减格式（如 `100:10`）转换为折扣比例（$0.9$），将折扣格式（如 `0.8`）保持原样。
    - *不过这样可能损失了一些对满减这一特殊消费券形式的刻画*.
- **`Distance` (距离)**：
    - 用户距离商家的地理位置。将其转换为整数，并对缺失值填充了 `-1`。
- **`day_of_week` (领券星期几)**：
    - 根据领券日期 `Date_received` 提取。反映了用户在工作日与周末的不同消费习惯。
- **`on_received_cnt` (线上领券数)**：
    - 用户在 `ccf_online_stage1_train.csv` 数据集中领取的优惠券总数。
- **`on_use_rate` (线上核销率)**：
    - 用户在线上领券后实际使用的比例（$\frac{已使用数}{领券总数}$）。
### Code
这里展示核心代码
```python
import os
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, when, datediff, to_date, udf, dayofweek, count,expr, sum as _sum
from pyspark.sql.types import DoubleType, FloatType, IntegerType
from pyspark.ml.feature import VectorAssembler
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml import Pipeline

# DATA_DIR is the absolute path in my docker sparker.
DATA_DIR = "/opt/spark/work-dir/lab4/data"
OUTPUT_DIR = "/opt/spark/work-dir/lab4/output"
spark = SparkSession.builder \
    .appName("O2O_Advanced_Model") \
    .config("spark.sql.shuffle.partitions", "10") \
    .getOrCreate()

# Wash and Featuring

def clean_null_strings(df):
    """清理字符串中的 'null' 异常值"""
    for col_name in df.columns:
        df = df.withColumn(col_name, 
            # wash the "null" col
            when(col(col_name).cast("string") == "null", None)
            .otherwise(col(col_name))
        )
    return df

@udf(returnType=DoubleType())
def parse_discount(d):
    """解析折扣率 UDF"""
    if not d: return 1.0
    s = str(d).strip()
    if ':' in s:
        try:
            x, y = s.split(':')
            return (float(x) - float(y)) / float(x)
        except: return 1.0
    try:
        return float(s)
    except: return 1.0


def get_online_user_features():
    print(">>> 正在挖掘 Online 用户行为特征...")
    online_raw = spark.read.csv(f"{DATA_DIR}/ccf_online_stage1_train.csv", header=True, inferSchema=True)
    online_clean = clean_null_strings(online_raw)
    
    user_on_features = online_clean.groupBy("User_id").agg(
        count("Coupon_id").alias("on_received_cnt"),
        _sum(when(col("Date").isNotNull(), 1).otherwise(0)).alias("on_used_cnt")
    # here we use Spark SQL expr to do
    ).withColumn("on_use_rate", expr("CASE WHEN on_received_cnt !=0 THEN on_used_cnt / on_received_cnt ELSE 0 END"))
    
    return user_on_features.na.fill(0.0)


def preprocess_with_online(df, online_features, is_train=False):
    # 日期与基础特征
    df = df.withColumn("rec_dt", to_date(col("Date_received").cast("string"), "yyyyMMdd"))
    
    if is_train:
        df = df.filter(col("Coupon_id").isNotNull())
        df = df.withColumn("con_dt", to_date(col("Date").cast("string"), "yyyyMMdd"))
        df = df.withColumn("label", when(
            (col("con_dt").isNotNull()) & (datediff(col("con_dt"), col("rec_dt")) <= 15), 1
        ).otherwise(0))
    
    df = df.withColumn("discount_rate_val", parse_discount(col("Discount_rate"))) \
           .withColumn("Distance", col("Distance").cast(IntegerType())) \
           .withColumn("day_of_week", dayofweek(col("rec_dt")))
    
    df = df.join(online_features, on="User_id", how="left")
    
    # set some default value
    return df.na.fill({
        "Distance": -1, "day_of_week": 1, "discount_rate_val": 1.0,
        "on_received_cnt": 0, "on_used_cnt": 0, "on_use_rate": 0.0
    })

# main
online_feats = get_online_user_features()

print(">>> 处理 Offline 训练数据...")
train_raw = spark.read.csv(f"{DATA_DIR}/ccf_offline_stage1_train.csv", header=True, inferSchema=True)
train_data = preprocess_with_online(clean_null_strings(train_raw), online_feats, is_train=True)

# train
feature_cols = ["discount_rate_val", "Distance", "day_of_week", "on_received_cnt", "on_use_rate"]
assembler = VectorAssembler(inputCols=feature_cols, outputCol="features")
rf = RandomForestClassifier(labelCol="label", featuresCol="features", numTrees=50, seed=42)
# here we use the pipeline.
pipeline = Pipeline(stages=[assembler, rf])

print(">>> 正在训练加强版随机森林模型...")
# fit as train function
model = pipeline.fit(train_data)

# predict
print(">>> 预测测试集...")
test_raw = spark.read.csv(f"{DATA_DIR}/ccf_offline_stage1_test_revised.csv", header=True, inferSchema=True)
test_data = preprocess_with_online(clean_null_strings(test_raw), online_feats, is_train=False)

# fit return a transformer model, we use that to do so
predictions = model.transform(test_data)
extract_prob_udf = udf(lambda v: float(v[1]), FloatType())
final_output = predictions.withColumn("Probability", extract_prob_udf("probability")) \
    .select("User_id", "Coupon_id", "Date_received", "Probability")

# output
output_path = f"{OUTPUT_DIR}/submission"
final_output.repartition(1).write.mode("overwrite").csv(output_path, header=False)

print(f">>> 任务完成！结果已存入: {output_path}")
```

### Result
下图是天池官网给出的 AUC 值。
`0.5417` 效果并不理想，可以考虑后续挖掘更多特征，并在数据清理阶段调整细节

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251218230648.png)


详细预测表格，请看文件中 `output/tianchi.csv`


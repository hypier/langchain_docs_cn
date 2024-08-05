---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/spark_sql.ipynb
---

# Spark SQL

本笔记本展示了如何使用代理与 `Spark SQL` 进行交互。与 [SQL Database Agent](/docs/integrations/toolkits/sql_database) 类似，它旨在处理有关 `Spark SQL` 的一般查询并促进错误恢复。

**注意：请注意，由于该代理正在积极开发中，所有答案可能并不正确。此外，不能保证代理在某些问题下不会对您的 Spark 集群执行 DML 语句。请在敏感数据上谨慎运行！**

## 初始化


```python
from langchain_community.agent_toolkits import SparkSQLToolkit, create_spark_sql_agent
from langchain_community.utilities.spark_sql import SparkSQL
from langchain_openai import ChatOpenAI
```


```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()
schema = "langchain_example"
spark.sql(f"CREATE DATABASE IF NOT EXISTS {schema}")
spark.sql(f"USE {schema}")
csv_file_path = "titanic.csv"
table = "titanic"
spark.read.csv(csv_file_path, header=True, inferSchema=True).write.saveAsTable(table)
spark.table(table).show()
```
```output
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
23/05/18 16:03:10 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
``````output
+-----------+--------+------+--------------------+------+----+-----+-----+----------------+-------+-----+--------+
|PassengerId|Survived|Pclass|                Name|   Sex| Age|SibSp|Parch|          Ticket|   Fare|Cabin|Embarked|
+-----------+--------+------+--------------------+------+----+-----+-----+----------------+-------+-----+--------+
|          1|       0|     3|Braund, Mr. Owen ...|  male|22.0|    1|    0|       A/5 21171|   7.25| null|       S|
|          2|       1|     1|Cumings, Mrs. Joh...|female|38.0|    1|    0|        PC 17599|71.2833|  C85|       C|
|          3|       1|     3|Heikkinen, Miss. ...|female|26.0|    0|    0|STON/O2. 3101282|  7.925| null|       S|
|          4|       1|     1|Futrelle, Mrs. Ja...|female|35.0|    1|    0|          113803|   53.1| C123|       S|
|          5|       0|     3|Allen, Mr. Willia...|  male|35.0|    0|    0|          373450|   8.05| null|       S|
|          6|       0|     3|    Moran, Mr. James|  male|null|    0|    0|          330877| 8.4583| null|       Q|
|          7|       0|     1|McCarthy, Mr. Tim...|  male|54.0|    0|    0|           17463|51.8625|  E46|       S|
|          8|       0|     3|Palsson, Master. ...|  male| 2.0|    3|    1|          349909| 21.075| null|       S|
|          9|       1|     3|Johnson, Mrs. Osc...|female|27.0|    0|    2|          347742|11.1333| null|       S|
|         10|       1|     2|Nasser, Mrs. Nich...|female|14.0|    1|    0|          237736|30.0708| null|       C|
|         11|       1|     3|Sandstrom, Miss. ...|female| 4.0|    1|    1|         PP 9549|   16.7|   G6|       S|
|         12|       1|     1|Bonnell, Miss. El...|female|58.0|    0|    0|          113783|  26.55| C103|       S|
|         13|       0|     3|Saundercock, Mr. ...|  male|20.0|    0|    0|       A/5. 2151|   8.05| null|       S|
|         14|       0|     3|Andersson, Mr. An...|  male|39.0|    1|    5|          347082| 31.275| null|       S|
|         15|       0|     3|Vestrom, Miss. Hu...|female|14.0|    0|    0|          350406| 7.8542| null|       S|
|         16|       1|     2|Hewlett, Mrs. (Ma...|female|55.0|    0|    0|          248706|   16.0| null|       S|
|         17|       0|     3|Rice, Master. Eugene|  male| 2.0|    4|    1|          382652| 29.125| null|       Q|
|         18|       1|     2|Williams, Mr. Cha...|  male|null|    0|    0|          244373|   13.0| null|       S|
|         19|       0|     3|Vander Planke, Mr...|female|31.0|    1|    0|          345763|   18.0| null|       S|
|         20|       1|     3|Masselmani, Mrs. ...|female|null|    0|    0|            2649|  7.225| null|       C|
+-----------+--------+------+--------------------+------+----+-----+-----+----------------+-------+-----+--------+
only showing top 20 rows
```

```python
# 注意，您还可以通过 Spark 连接到 Spark。例如：
# db = SparkSQL.from_uri("sc://localhost:15002", schema=schema)
spark_sql = SparkSQL(schema=schema)
llm = ChatOpenAI(temperature=0)
toolkit = SparkSQLToolkit(db=spark_sql, llm=llm)
agent_executor = create_spark_sql_agent(llm=llm, toolkit=toolkit, verbose=True)
```

## 示例：描述一个表格


```python
agent_executor.run("Describe the titanic table")
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m操作：list_tables_sql_db
操作输入：[0m
观察： [38;5;200m[1;3mtitanic[0m
思考：[32;1m[1;3m我找到了 titanic 表。现在我需要获取 titanic 表的模式和示例行。
操作：schema_sql_db
操作输入：titanic[0m
观察： [33;1m[1;3mCREATE TABLE langchain_example.titanic (
  PassengerId INT,
  Survived INT,
  Pclass INT,
  Name STRING,
  Sex STRING,
  Age DOUBLE,
  SibSp INT,
  Parch INT,
  Ticket STRING,
  Fare DOUBLE,
  Cabin STRING,
  Embarked STRING)
;

/*
titanic 表的 3 行数据：
PassengerId	Survived	Pclass	Name	Sex	Age	SibSp	Parch	Ticket	Fare	Cabin	Embarked
1	0	3	Braund, Mr. Owen Harris	男性	22.0	1	0	A/5 21171	7.25	无	S
2	1	1	Cumings, Mrs. John Bradley (Florence Briggs Thayer)	女性	38.0	1	0	PC 17599	71.2833	C85	C
3	1	3	Heikkinen, Miss. Laina	女性	26.0	0	0	STON/O2. 3101282	7.925	无	S
*/[0m
思考：[32;1m[1;3m我现在知道了 titanic 表的模式和示例行。
最终答案：titanic 表具有以下列：PassengerId (INT)，Survived (INT)，Pclass (INT)，Name (STRING)，Sex (STRING)，Age (DOUBLE)，SibSp (INT)，Parch (INT)，Ticket (STRING)，Fare (DOUBLE)，Cabin (STRING)，和 Embarked (STRING)。以下是表中的一些示例行： 

1. PassengerId: 1, Survived: 0, Pclass: 3, Name: Braund, Mr. Owen Harris, Sex: male, Age: 22.0, SibSp: 1, Parch: 0, Ticket: A/5 21171, Fare: 7.25, Cabin: None, Embarked: S
2. PassengerId: 2, Survived: 1, Pclass: 1, Name: Cumings, Mrs. John Bradley (Florence Briggs Thayer), Sex: female, Age: 38.0, SibSp: 1, Parch: 0, Ticket: PC 17599, Fare: 71.2833, Cabin: C85, Embarked: C
3. PassengerId: 3, Survived: 1, Pclass: 3, Name: Heikkinen, Miss. Laina, Sex: female, Age: 26.0, SibSp: 0, Parch: 0, Ticket: STON/O2. 3101282, Fare: 7.925, Cabin: None, Embarked: S[0m

[1m> 完成链。[0m
```


```output
'titanic 表具有以下列：PassengerId (INT)，Survived (INT)，Pclass (INT)，Name (STRING)，Sex (STRING)，Age (DOUBLE)，SibSp (INT)，Parch (INT)，Ticket (STRING)，Fare (DOUBLE)，Cabin (STRING)，和 Embarked (STRING)。以下是表中的一些示例行： \n\n1. PassengerId: 1, Survived: 0, Pclass: 3, Name: Braund, Mr. Owen Harris, Sex: male, Age: 22.0, SibSp: 1, Parch: 0, Ticket: A/5 21171, Fare: 7.25, Cabin: None, Embarked: S\n2. PassengerId: 2, Survived: 1, Pclass: 1, Name: Cumings, Mrs. John Bradley (Florence Briggs Thayer), Sex: female, Age: 38.0, SibSp: 1, Parch: 0, Ticket: PC 17599, Fare: 71.2833, Cabin: C85, Embarked: C\n3. PassengerId: 3, Survived: 1, Pclass: 3, Name: Heikkinen, Miss. Laina, Sex: female, Age: 26.0, SibSp: 0, Parch: 0, Ticket: STON/O2. 3101282, Fare: 7.925, Cabin: None, Embarked: S'
```

## 示例：运行查询


```python
agent_executor.run("whats the square root of the average age?")
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m动作：list_tables_sql_db
动作输入：[0m
观察：[38;5;200m[1;3mtitanic[0m
思考：[32;1m[1;3m我应该检查 titanic 表的结构，看看是否有年龄列。
动作：schema_sql_db
动作输入：titanic[0m
观察：[33;1m[1;3mCREATE TABLE langchain_example.titanic (
  PassengerId INT,
  Survived INT,
  Pclass INT,
  Name STRING,
  Sex STRING,
  Age DOUBLE,
  SibSp INT,
  Parch INT,
  Ticket STRING,
  Fare DOUBLE,
  Cabin STRING,
  Embarked STRING)
;

/*
titanic 表的 3 行数据：
PassengerId	Survived	Pclass	Name	Sex	Age	SibSp	Parch	Ticket	Fare	Cabin	Embarked
1	0	3	Braund, Mr. Owen Harris	male	22.0	1	0	A/5 21171	7.25	None	S
2	1	1	Cumings, Mrs. John Bradley (Florence Briggs Thayer)	female	38.0	1	0	PC 17599	71.2833	C85	C
3	1	3	Heikkinen, Miss. Laina	female	26.0	0	0	STON/O2. 3101282	7.925	None	S
*/[0m
思考：[32;1m[1;3m在 titanic 表中有一个年龄列。我应该写一个查询来计算平均年龄，然后找到结果的平方根。
动作：query_checker_sql_db
动作输入：SELECT SQRT(AVG(Age)) as square_root_of_avg_age FROM titanic[0m
观察：[31;1m[1;3m原始查询似乎是正确的。再看一遍：

SELECT SQRT(AVG(Age)) as square_root_of_avg_age FROM titanic[0m
思考：[32;1m[1;3m查询是正确的，所以我可以执行它来找到平均年龄的平方根。
动作：query_sql_db
动作输入：SELECT SQRT(AVG(Age)) as square_root_of_avg_age FROM titanic[0m
观察：[36;1m[1;3m[('5.449689683556195',)][0m
思考：[32;1m[1;3m我现在知道最终答案
最终答案：平均年龄的平方根大约是 5.45。[0m

[1m> 完成链。[0m
```


```output
'平均年龄的平方根大约是 5.45。'
```



```python
agent_executor.run("What's the name of the oldest survived passenger?")
```
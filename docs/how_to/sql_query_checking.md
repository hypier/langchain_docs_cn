---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/sql_query_checking.ipynb
---

# 如何在 SQL 问答中进行查询验证

在任何 SQL 链或代理中，编写有效且安全的 SQL 查询可能是最容易出错的部分。在本指南中，我们将讨论一些验证查询和处理无效查询的策略。

我们将涵盖：

1. 在查询生成中添加“查询验证器”步骤；
2. 提示工程以减少错误发生的频率。

## 设置

首先，获取所需的包并设置环境变量：

```python
%pip install --upgrade --quiet  langchain langchain-community langchain-openai
```

```python
# 取消注释以下内容以使用 LangSmith。不是必需的。
# import os
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
```

以下示例将使用与 Chinook 数据库的 SQLite 连接。请按照 [这些安装步骤](https://database.guide/2-sample-databases-sqlite/) 在与此笔记本相同的目录中创建 `Chinook.db`：

* 将 [此文件](https://raw.githubusercontent.com/lerocha/chinook-database/master/ChinookDatabase/DataSources/Chinook_Sqlite.sql) 保存为 `Chinook_Sqlite.sql`
* 运行 `sqlite3 Chinook.db`
* 运行 `.read Chinook_Sqlite.sql`
* 测试 `SELECT * FROM Artist LIMIT 10;`

现在，`Chinook.db` 在我们的目录中，我们可以使用 SQLAlchemy 驱动的 `SQLDatabase` 类与之接口：

```python
from langchain_community.utilities import SQLDatabase

db = SQLDatabase.from_uri("sqlite:///Chinook.db")
print(db.dialect)
print(db.get_usable_table_names())
print(db.run("SELECT * FROM Artist LIMIT 10;"))
```
```output
sqlite
['Album', 'Artist', 'Customer', 'Employee', 'Genre', 'Invoice', 'InvoiceLine', 'MediaType', 'Playlist', 'PlaylistTrack', 'Track']
[(1, 'AC/DC'), (2, 'Accept'), (3, 'Aerosmith'), (4, 'Alanis Morissette'), (5, 'Alice In Chains'), (6, 'Antônio Carlos Jobim'), (7, 'Apocalyptica'), (8, 'Audioslave'), (9, 'BackBeat'), (10, 'Billy Cobham')]
```

## 查询检查器

或许最简单的策略是让模型本身检查原始查询中的常见错误。假设我们有以下 SQL 查询链：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />

```python
from langchain.chains import create_sql_query_chain

chain = create_sql_query_chain(llm, db)
```

我们想要验证其输出。我们可以通过扩展链并添加第二个提示和模型调用来实现：

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate

system = """仔细检查用户的 {dialect} 查询是否存在常见错误，包括：
- 在 NULL 值上使用 NOT IN
- 在应该使用 UNION ALL 时使用 UNION
- 对于排他范围使用 BETWEEN
- 谓词中的数据类型不匹配
- 正确引用标识符
- 函数的参数数量正确
- 转换为正确的数据类型
- 使用正确的列进行连接

如果存在上述错误，请重写查询。
如果没有错误，请仅复制原始查询，不做进一步评论。

仅输出最终的 SQL 查询。"""
prompt = ChatPromptTemplate.from_messages(
    [("system", system), ("human", "{query}")]
).partial(dialect=db.dialect)
validation_chain = prompt | llm | StrOutputParser()

full_chain = {"query": chain} | validation_chain
```

```python
query = full_chain.invoke(
    {
        "question": "来自美国客户的平均发票是多少，且自2003年以来缺少传真，但在2010年之前"
    }
)
print(query)
```
```output
SELECT AVG(i.Total) AS AverageInvoice
FROM Invoice i
JOIN Customer c ON i.CustomerId = c.CustomerId
WHERE c.Country = 'USA'
AND c.Fax IS NULL
AND i.InvoiceDate >= '2003-01-01' 
AND i.InvoiceDate < '2010-01-01'
```
注意我们可以在 [Langsmith trace](https://smith.langchain.com/public/8a743295-a57c-4e4c-8625-bc7e36af9d74/r) 中看到链的两个步骤。

```python
db.run(query)
```

```output
'[(6.632999999999998,)]'
```

这种方法明显的缺点是我们需要进行两次模型调用而不是一次来生成查询。为了避免这种情况，我们可以尝试在一次模型调用中进行查询生成和查询检查：

```python
system = """你是一个 {dialect} 专家。给定一个输入问题，创建一个语法正确的 {dialect} 查询来执行。
除非用户在问题中指定要获得的示例数量，否则使用 LIMIT 子句查询最多 {top_k} 个结果，按照 {dialect} 的要求。你可以对结果进行排序，以返回数据库中最有信息的数据。
永远不要查询表中的所有列。你必须只查询回答问题所需的列。将每个列名用双引号 (") 括起来，以表示它们是限定标识符。
注意只使用你在下面的表中看到的列名。小心不要查询不存在的列。同时，注意哪个列在哪个表中。
如果问题涉及“今天”，请注意使用 date('now') 函数获取当前日期。

仅使用以下表：
{table_info}

写一个查询的初始草稿。然后仔细检查 {dialect} 查询是否存在常见错误，包括：
- 在 NULL 值上使用 NOT IN
- 在应该使用 UNION ALL 时使用 UNION
- 对于排他范围使用 BETWEEN
- 谓词中的数据类型不匹配
- 正确引用标识符
- 函数的参数数量正确
- 转换为正确的数据类型
- 使用正确的列进行连接

使用格式：

初稿: <<FIRST_DRAFT_QUERY>>
最终答案: <<FINAL_ANSWER_QUERY>>
"""
prompt = ChatPromptTemplate.from_messages(
    [("system", system), ("human", "{input}")]
).partial(dialect=db.dialect)


def parse_final_answer(output: str) -> str:
    return output.split("最终答案: ")[1]


chain = create_sql_query_chain(llm, db, prompt=prompt) | parse_final_answer
prompt.pretty_print()
```
```output
================================[1m 系统消息 [0m================================

你是一个 [33;1m[1;3m{dialect}[0m 专家。给定一个输入问题，创建一个语法正确的 [33;1m[1;3m{dialect}[0m 查询来执行。
除非用户在问题中指定要获得的示例数量，否则使用 LIMIT 子句查询最多 [33;1m[1;3m{top_k}[0m 个结果，按照 [33;1m[1;3m{dialect}[0m 的要求。你可以对结果进行排序，以返回数据库中最有信息的数据。
永远不要查询表中的所有列。你必须只查询回答问题所需的列。将每个列名用双引号 (") 括起来，以表示它们是限定标识符。
注意只使用你在下面的表中看到的列名。小心不要查询不存在的列。同时，注意哪个列在哪个表中。
如果问题涉及“今天”，请注意使用 date('now') 函数获取当前日期。

仅使用以下表：
[33;1m[1;3m{table_info}[0m

写一个查询的初始草稿。然后仔细检查 [33;1m[1;3m{dialect}[0m 查询是否存在常见错误，包括：
- 在 NULL 值上使用 NOT IN
- 在应该使用 UNION ALL 时使用 UNION
- 对于排他范围使用 BETWEEN
- 谓词中的数据类型不匹配
- 正确引用标识符
- 函数的参数数量正确
- 转换为正确的数据类型
- 使用正确的列进行连接

使用格式：

初稿: <<FIRST_DRAFT_QUERY>>
最终答案: <<FINAL_ANSWER_QUERY>>


================================[1m 人类消息 [0m=================================

[33;1m[1;3m{input}[0m
```

```python
query = chain.invoke(
    {
        "question": "来自美国客户的平均发票是多少，且自2003年以来缺少传真，但在2010年之前"
    }
)
print(query)
```
```output
SELECT AVG(i."Total") AS "AverageInvoice"
FROM "Invoice" i
JOIN "Customer" c ON i."CustomerId" = c."CustomerId"
WHERE c."Country" = 'USA'
AND c."Fax" IS NULL
AND i."InvoiceDate" BETWEEN '2003-01-01' AND '2010-01-01';
```

```python
db.run(query)
```

```output
'[(6.632999999999998,)]'
```

## 人工参与

在某些情况下，我们的数据敏感到不希望在没有人类先行批准的情况下执行 SQL 查询。请前往 [工具使用：人工参与](/docs/how_to/tools_human) 页面，了解如何为任何工具、链或代理添加人工参与。

## 错误处理

在某些情况下，模型可能会出错并生成无效的 SQL 查询。或者我们的数据库可能会出现问题。或者模型 API 可能会宕机。我们希望在这些情况下为我们的链和代理添加一些错误处理行为，以便优雅地失败，甚至可能自动恢复。要了解有关工具的错误处理，请访问 [工具使用：错误处理](/docs/how_to/tools_error) 页面。
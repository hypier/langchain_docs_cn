---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/tutorials/sql_qa.ipynb
---

# 在 SQL 数据上构建问答系统

:::info 先决条件

本指南假设您对以下概念有一定的了解：

- [链接可运行的任务](/docs/how_to/sequence/)
- [聊天模型](/docs/concepts/#chat-models)
- [工具](/docs/concepts/#tools)
- [代理](/docs/concepts/#agents)

:::

使 LLM 系统能够查询结构化数据与查询非结构化文本数据在性质上可能有很大不同。在后者中，生成可以与向量数据库搜索的文本是很常见的，而对于结构化数据，LLM 通常需要编写并执行 DSL 查询，例如 SQL。在本指南中，我们将介绍如何在数据库的表格数据上创建问答系统的基本方法。我们将涵盖使用链和代理的实现。这些系统将允许我们就数据库中的数据提出问题，并获得自然语言的回答。这两者之间的主要区别在于，我们的代理可以在循环中查询数据库多次，以满足回答问题的需要。

## ⚠️ 安全提示 ⚠️

构建 SQL 数据库的问答系统需要执行模型生成的 SQL 查询。这其中存在固有的风险。确保您的数据库连接权限始终根据您的链/代理的需求尽可能狭窄地范围化。这将减轻但不能消除构建模型驱动系统的风险。有关一般安全最佳实践的更多信息，请[查看这里](/docs/security)。

## 架构

从高层次来看，这些系统的步骤是：

1. **将问题转换为 DSL 查询**：模型将用户输入转换为 SQL 查询。
2. **执行 SQL 查询**：执行查询。
3. **回答问题**：模型使用查询结果响应用户输入。

请注意，查询 CSV 中的数据可以遵循类似的方法。有关 CSV 数据问答的更多细节，请参阅我们的 [使用指南](/docs/how_to/sql_csv)。

![sql_usecase.png](../../static/img/sql_usecase.png)

## 设置

首先，获取所需的包并设置环境变量：

```python
%%capture --no-stderr
%pip install --upgrade --quiet langchain langchain-community langchain-openai faiss-cpu
```

在本指南中，我们将使用 OpenAI 模型和一个 [FAISS 支持的向量存储](/docs/integrations/vectorstores/faiss/)。

```python
import getpass
import os

if not os.environ.get("OPENAI_API_KEY"):
    os.environ["OPENAI_API_KEY"] = getpass.getpass()

# 注释掉下面的内容以选择不在此笔记本中使用 LangSmith。不是必需的。
if not os.environ.get("LANGCHAIN_API_KEY"):
    os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
    os.environ["LANGCHAIN_TRACING_V2"] = "true"
```

下面的示例将使用与 Chinook 数据库的 SQLite 连接。按照 [这些安装步骤](https://database.guide/2-sample-databases-sqlite/) 在与此笔记本相同的目录中创建 `Chinook.db`：

* 将 [此文件](https://raw.githubusercontent.com/lerocha/chinook-database/master/ChinookDatabase/DataSources/Chinook_Sqlite.sql) 保存为 `Chinook.sql`
* 运行 `sqlite3 Chinook.db`
* 运行 `.read Chinook.sql`
* 测试 `SELECT * FROM Artist LIMIT 10;`

现在，`Chinook.db` 在我们的目录中，我们可以使用 SQLAlchemy 驱动的 `SQLDatabase` 类与它进行交互：

```python
from langchain_community.utilities import SQLDatabase

db = SQLDatabase.from_uri("sqlite:///Chinook.db")
print(db.dialect)
print(db.get_usable_table_names())
db.run("SELECT * FROM Artist LIMIT 10;")
```
```output
sqlite
['Album', 'Artist', 'Customer', 'Employee', 'Genre', 'Invoice', 'InvoiceLine', 'MediaType', 'Playlist', 'PlaylistTrack', 'Track']
```


```output
"[(1, 'AC/DC'), (2, 'Accept'), (3, 'Aerosmith'), (4, 'Alanis Morissette'), (5, 'Alice In Chains'), (6, 'Antônio Carlos Jobim'), (7, 'Apocalyptica'), (8, 'Audioslave'), (9, 'BackBeat'), (10, 'Billy Cobham')]"
```


太好了！我们有一个可以查询的 SQL 数据库。现在让我们尝试将其连接到一个 LLM。

## Chains {#chains}

链（即 LangChain [Runnables](/docs/concepts#langchain-expression-language-lcel) 的组合）支持步骤可预测的应用程序。我们可以创建一个简单的链，它接受一个问题并执行以下操作：
- 将问题转换为 SQL 查询；
- 执行查询；
- 使用结果来回答原始问题。

这种安排不支持某些场景。例如，该系统将对任何用户输入执行 SQL 查询——甚至是“你好”。重要的是，正如我们在下面将看到的，有些问题需要多个查询才能回答。我们将在代理部分讨论这些场景。

### 将问题转换为SQL查询

SQL链或代理的第一步是接受用户输入并将其转换为SQL查询。LangChain提供了一个内置链来实现这一点：[create_sql_query_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.sql_database.query.create_sql_query_chain.html)。

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />

```python
from langchain.chains import create_sql_query_chain

chain = create_sql_query_chain(llm, db)
response = chain.invoke({"question": "How many employees are there"})
response
```

```output
'SELECT COUNT("EmployeeId") AS "TotalEmployees" FROM "Employee"\nLIMIT 1;'
```

我们可以执行查询以确保它是有效的：

```python
db.run(response)
```

```output
'[(8,)]'
```

我们可以查看[LangSmith跟踪](https://smith.langchain.com/public/c8fa52ea-be46-4829-bde2-52894970b830/r)，以更好地理解这个链的功能。我们还可以直接检查链的提示。查看提示（如下），我们可以看到它是：

* 特定于方言的。在这种情况下，它明确引用了SQLite。
* 对所有可用表有定义。
* 每个表有三个示例行。

这项技术受到像[这篇论文](https://arxiv.org/pdf/2204.00498.pdf)的启发，建议展示示例行并明确表格可以提高性能。我们还可以像这样检查完整的提示：

```python
chain.get_prompts()[0].pretty_print()
```
```output
You are a SQLite expert. Given an input question, first create a syntactically correct SQLite query to run, then look at the results of the query and return the answer to the input question.
Unless the user specifies in the question a specific number of examples to obtain, query for at most 5 results using the LIMIT clause as per SQLite. You can order the results to return the most informative data in the database.
Never query for all columns from a table. You must query only the columns that are needed to answer the question. Wrap each column name in double quotes (") to denote them as delimited identifiers.
Pay attention to use only the column names you can see in the tables below. Be careful to not query for columns that do not exist. Also, pay attention to which column is in which table.
Pay attention to use date('now') function to get the current date, if the question involves "today".

Use the following format:

Question: Question here
SQLQuery: SQL Query to run
SQLResult: Result of the SQLQuery
Answer: Final answer here

Only use the following tables:
[33;1m[1;3m{table_info}[0m

Question: [33;1m[1;3m{input}[0m
```

### 执行 SQL 查询

现在我们已经生成了一个 SQL 查询，我们需要执行它。**这是创建 SQL 链中最危险的部分。** 请仔细考虑是否可以对您的数据运行自动化查询。尽可能最小化数据库连接权限。在查询执行之前，考虑在您的链中添加一个人工审批步骤（见下文）。

我们可以使用 `QuerySQLDatabaseTool` 来轻松地将查询执行添加到我们的链中：


```python
from langchain_community.tools.sql_database.tool import QuerySQLDataBaseTool

execute_query = QuerySQLDataBaseTool(db=db)
write_query = create_sql_query_chain(llm, db)
chain = write_query | execute_query
chain.invoke({"question": "How many employees are there"})
```



```output
'[(8,)]'
```

### 回答问题

现在我们已经找到了一种自动生成和执行查询的方法，我们只需要将原始问题和 SQL 查询结果结合起来，生成最终答案。我们可以通过将问题和结果再次传递给 LLM 来实现这一点：

```python
from operator import itemgetter

from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnablePassthrough

answer_prompt = PromptTemplate.from_template(
    """Given the following user question, corresponding SQL query, and SQL result, answer the user question.

Question: {question}
SQL Query: {query}
SQL Result: {result}
Answer: """
)

chain = (
    RunnablePassthrough.assign(query=write_query).assign(
        result=itemgetter("query") | execute_query
    )
    | answer_prompt
    | llm
    | StrOutputParser()
)

chain.invoke({"question": "How many employees are there"})
```

```output
'There are a total of 8 employees.'
```

让我们回顾一下上述 LCEL 中发生的事情。假设这个链被调用。
- 在第一次 `RunnablePassthrough.assign` 之后，我们得到了一个包含两个元素的可运行体：  
  `{"question": question, "query": write_query.invoke(question)}`  
  其中 `write_query` 将生成一个 SQL 查询，以便回答问题。
- 在第二次 `RunnablePassthrough.assign` 之后，我们添加了一个第三个元素 `"result"`，其中包含 `execute_query.invoke(query)`，而 `query` 是在前一步计算得出的。
- 这三个输入被格式化到提示中，并传递给 LLM。
- `StrOutputParser()` 从输出消息中提取字符串内容。

请注意，我们正在将 LLM、工具、提示和其他链组合在一起，但由于每个都实现了 Runnable 接口，因此它们的输入和输出可以以合理的方式连接在一起。

### 下一步

对于更复杂的查询生成，我们可能需要创建少量示例提示或添加查询检查步骤。有关此类高级技术和更多信息，请查看：

* [提示策略](/docs/how_to/sql_prompting)：高级提示工程技术。
* [查询检查](/docs/how_to/sql_query_checking)：添加查询验证和错误处理。
* [大型数据库](/docs/how_to/sql_large_db)：处理大型数据库的技术。

## Agents {#agents}

LangChain 具有一个 SQL Agent，它提供了比链式方式与 SQL 数据库交互更灵活的方法。使用 SQL Agent 的主要优势包括：

- 它可以基于数据库的模式和数据库的内容回答问题（例如描述特定的表）。
- 它可以通过运行生成的查询来恢复错误，捕获回溯并正确重新生成。
- 它可以根据需要多次查询数据库以回答用户问题。
- 它将通过仅从相关表中检索模式来节省令牌。

要初始化代理，我们将使用 `SQLDatabaseToolkit` 创建一组工具：

* 创建和执行查询
* 检查查询语法
* 检索表描述
* ... 以及更多


```python
from langchain_community.agent_toolkits import SQLDatabaseToolkit

toolkit = SQLDatabaseToolkit(db=db, llm=llm)

tools = toolkit.get_tools()

tools
```



```output
[QuerySQLDataBaseTool(description="Input to this tool is a detailed and correct SQL query, output is a result from the database. If the query is not correct, an error message will be returned. If an error is returned, rewrite the query, check the query, and try again. If you encounter an issue with Unknown column 'xxxx' in 'field list', use sql_db_schema to query the correct table fields.", db=<langchain_community.utilities.sql_database.SQLDatabase object at 0x113403b50>),
 InfoSQLDatabaseTool(description='Input to this tool is a comma-separated list of tables, output is the schema and sample rows for those tables. Be sure that the tables actually exist by calling sql_db_list_tables first! Example Input: table1, table2, table3', db=<langchain_community.utilities.sql_database.SQLDatabase object at 0x113403b50>),
 ListSQLDatabaseTool(db=<langchain_community.utilities.sql_database.SQLDatabase object at 0x113403b50>),
 QuerySQLCheckerTool(description='Use this tool to double check if your query is correct before executing it. Always use this tool before executing a query with sql_db_query!', db=<langchain_community.utilities.sql_database.SQLDatabase object at 0x113403b50>, llm=ChatOpenAI(client=<openai.resources.chat.completions.Completions object at 0x115b7e890>, async_client=<openai.resources.chat.completions.AsyncCompletions object at 0x115457e10>, temperature=0.0, openai_api_key=SecretStr('**********'), openai_proxy=''), llm_chain=LLMChain(prompt=PromptTemplate(input_variables=['dialect', 'query'], template='\n{query}\nDouble check the {dialect} query above for common mistakes, including:\n- Using NOT IN with NULL values\n- Using UNION when UNION ALL should have been used\n- Using BETWEEN for exclusive ranges\n- Data type mismatch in predicates\n- Properly quoting identifiers\n- Using the correct number of arguments for functions\n- Casting to the correct data type\n- Using the proper columns for joins\n\nIf there are any of the above mistakes, rewrite the query. If there are no mistakes, just reproduce the original query.\n\nOutput the final SQL query only.\n\nSQL Query: '), llm=ChatOpenAI(client=<openai.resources.chat.completions.Completions object at 0x115b7e890>, async_client=<openai.resources.chat.completions.AsyncCompletions object at 0x115457e10>, temperature=0.0, openai_api_key=SecretStr('**********'), openai_proxy='')))]
```

### 系统提示

我们还想为我们的代理创建一个系统提示。这将包括行为指令。

```python
from langchain_core.messages import SystemMessage

SQL_PREFIX = """你是一个旨在与 SQL 数据库交互的代理。
根据输入的问题，创建一个语法正确的 SQLite 查询，然后查看查询结果并返回答案。
除非用户指定希望获得的示例数量，否则始终将查询限制为最多 5 个结果。
你可以根据相关列对结果进行排序，以返回数据库中最有趣的示例。
永远不要查询特定表中的所有列，只要求与问题相关的列。
你可以使用与数据库交互的工具。
仅使用以下工具。仅使用以下工具返回的信息来构建你的最终答案。
在执行查询之前，你必须仔细检查你的查询。如果在执行查询时遇到错误，请重写查询并重试。

请勿对数据库进行任何 DML 语句（INSERT、UPDATE、DELETE、DROP 等）。

开始时，你应始终查看数据库中的表，以了解可以查询的内容。
请勿跳过此步骤。
然后，你应该查询最相关表的模式。"""

system_message = SystemMessage(content=SQL_PREFIX)
```

### 初始化代理
首先，获取所需的包 **LangGraph**

```python
%%capture --no-stderr
%pip install --upgrade --quiet langgraph
```

我们将使用预构建的 [LangGraph](/docs/concepts/#langgraph) 代理来构建我们的代理

```python
from langchain_core.messages import HumanMessage
from langgraph.prebuilt import create_react_agent

agent_executor = create_react_agent(llm, tools, messages_modifier=system_message)
```

考虑代理如何回答以下问题：

```python
for s in agent_executor.stream(
    {"messages": [HumanMessage(content="哪个国家的客户消费最多？")]}
):
    print(s)
    print("----")

注意，代理会执行多个查询，直到获取到所需的信息：
1. 列出可用的表；
2. 检索三个表的架构；
3. 通过连接操作查询多个表。

然后，代理能够使用最终查询的结果生成对原始问题的回答。

代理同样可以处理定性问题：

```python
for s in agent_executor.stream(
    {"messages": [HumanMessage(content="描述 playlisttrack 表")]}
):
    print(s)
    print("----")
```

### 处理高基数列

为了过滤包含专有名词（如地址、歌曲名称或艺术家）的列，我们首先需要仔细检查拼写，以便正确过滤数据。

我们可以通过创建一个包含数据库中所有不同专有名词的向量存储来实现这一点。然后，每当用户在问题中包含专有名词时，代理可以查询该向量存储，以找到该词的正确拼写。通过这种方式，代理可以确保在构建目标查询之前理解用户所指的实体。

首先，我们需要获取每个实体的唯一值，为此我们定义一个函数，将结果解析为元素列表：

```python
import ast
import re


def query_as_list(db, query):
    res = db.run(query)
    res = [el for sub in ast.literal_eval(res) for el in sub if el]
    res = [re.sub(r"\b\d+\b", "", string).strip() for string in res]
    return list(set(res))


artists = query_as_list(db, "SELECT Name FROM Artist")
albums = query_as_list(db, "SELECT Title FROM Album")
albums[:5]
```

```output
['Big Ones',
 'Cidade Negra - Hits',
 'In Step',
 'Use Your Illusion I',
 'Voodoo Lounge']
```

使用这个函数，我们可以创建一个 **检索工具**，代理可以根据需要执行。

```python
from langchain.agents.agent_toolkits import create_retriever_tool
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings

vector_db = FAISS.from_texts(artists + albums, OpenAIEmbeddings())
retriever = vector_db.as_retriever(search_kwargs={"k": 5})
description = """用于查找要过滤的值。输入是专有名词的近似拼写，输出是有效的专有名词。使用与搜索最相似的名词。"""
retriever_tool = create_retriever_tool(
    retriever,
    name="search_proper_nouns",
    description=description,
)
```

让我们试试看：

```python
print(retriever_tool.invoke("Alice Chains"))
```
```output
Alice In Chains

Alanis Morissette

Pearl Jam

Pearl Jam

Audioslave
```
这样，如果代理确定需要根据艺术家写一个过滤器，例如 "Alice Chains"，它可以首先使用检索工具来观察列的相关值。

将这些结合起来：

```python
system = """您是一个与 SQL 数据库交互的代理。
给定一个输入问题，创建一个语法正确的 SQLite 查询来运行，然后查看查询的结果并返回答案。
除非用户指定他们希望获得的示例数量，否则始终将查询限制为最多 5 个结果。
您可以按相关列对结果进行排序，以返回数据库中最有趣的示例。
绝不要查询特定表的所有列，只请求与问题相关的列。
您可以使用与数据库交互的工具。
仅使用给定的工具。仅使用工具返回的信息来构建您的最终答案。
在执行查询之前，您必须仔细检查您的查询。如果在执行查询时遇到错误，请重写查询并重试。

请勿对数据库进行任何 DML 语句（INSERT、UPDATE、DELETE、DROP 等）。

您可以访问以下表：{table_names}

如果您需要过滤专有名词，您必须始终首先使用“search_proper_nouns”工具查找过滤值！
不要试图猜测专有名词 - 使用此函数找到相似的。""".format(
    table_names=db.get_usable_table_names()
)

system_message = SystemMessage(content=system)

tools.append(retriever_tool)

agent = create_react_agent(llm, tools, messages_modifier=system_message)
```

```python
for s in agent.stream(
    {"messages": [HumanMessage(content="Alice In Chains 有多少张专辑？")]}
):
    print(s)
    print("----")
```
```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_r5UlSwHKQcWDHx6LrttnqE56', 'function': {'arguments': '{"query":"SELECT COUNT(*) AS album_count FROM Album WHERE ArtistId IN (SELECT ArtistId FROM Artist WHERE Name = \'Alice In Chains\')"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 40, 'prompt_tokens': 612, 'total_tokens': 652}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-548353fd-b06c-45bf-beab-46f81eb434df-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': "SELECT COUNT(*) AS album_count FROM Album WHERE ArtistId IN (SELECT ArtistId FROM Artist WHERE Name = 'Alice In Chains')"}, 'id': 'call_r5UlSwHKQcWDHx6LrttnqE56'}])]}}
----
{'action': {'messages': [ToolMessage(content='[(1,)]', name='sql_db_query', id='093058a9-f013-4be1-8e7a-ed839b0c90cd', tool_call_id='call_r5UlSwHKQcWDHx6LrttnqE56')]}}
----
{'agent': {'messages': [AIMessage(content='Alice In Chains 有 11 张专辑。', response_metadata={'token_usage': {'completion_tokens': 9, 'prompt_tokens': 665, 'total_tokens': 674}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'stop', 'logprobs': None}, id='run-f804eaab-9812-4fb3-ae8b-280af8594ac6-0')]}}
----
```
如我们所见，代理使用 `search_proper_nouns` 工具来检查如何正确查询数据库以获取该特定艺术家。
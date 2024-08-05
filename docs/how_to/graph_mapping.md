---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/graph_mapping.ipynb
sidebar_position: 1
---

# 如何将值映射到图数据库

在本指南中，我们将讨论通过将用户输入的值映射到数据库来改善图数据库查询生成的策略。当使用内置的图链时，LLM 知道图模式，但对存储在数据库中的属性值没有任何信息。因此，我们可以在图数据库 QA 系统中引入一个新步骤，以准确映射值。

## 设置

首先，获取所需的包并设置环境变量：

```python
%pip install --upgrade --quiet  langchain langchain-community langchain-openai neo4j
```

在本指南中，我们默认使用 OpenAI 模型，但您可以将其替换为您选择的模型提供商。

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()

# 取消注释以使用 LangSmith。不是必需的。
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
```
```output
 ········
```
接下来，我们需要定义 Neo4j 凭证。
按照 [这些安装步骤](https://neo4j.com/docs/operations-manual/current/installation/) 设置 Neo4j 数据库。

```python
os.environ["NEO4J_URI"] = "bolt://localhost:7687"
os.environ["NEO4J_USERNAME"] = "neo4j"
os.environ["NEO4J_PASSWORD"] = "password"
```

下面的示例将与 Neo4j 数据库建立连接，并用有关电影及其演员的示例数据填充它。

```python
from langchain_community.graphs import Neo4jGraph

graph = Neo4jGraph()

# 导入电影信息

movies_query = """
LOAD CSV WITH HEADERS FROM 
'https://raw.githubusercontent.com/tomasonjo/blog-datasets/main/movies/movies_small.csv'
AS row
MERGE (m:Movie {id:row.movieId})
SET m.released = date(row.released),
    m.title = row.title,
    m.imdbRating = toFloat(row.imdbRating)
FOREACH (director in split(row.director, '|') | 
    MERGE (p:Person {name:trim(director)})
    MERGE (p)-[:DIRECTED]->(m))
FOREACH (actor in split(row.actors, '|') | 
    MERGE (p:Person {name:trim(actor)})
    MERGE (p)-[:ACTED_IN]->(m))
FOREACH (genre in split(row.genres, '|') | 
    MERGE (g:Genre {name:trim(genre)})
    MERGE (m)-[:IN_GENRE]->(g))
"""

graph.query(movies_query)
```

```output
[]
```

## 检测用户输入中的实体
我们需要提取我们想要映射到图形数据库的实体/值的类型。在这个例子中，我们处理的是电影图，因此我们可以将电影和人物映射到数据库中。


```python
from typing import List, Optional

from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)


class Entities(BaseModel):
    """有关实体的识别信息。"""

    names: List[str] = Field(
        ...,
        description="文本中出现的所有人物或电影",
    )


prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "您正在从文本中提取人物和电影。",
        ),
        (
            "human",
            "使用给定的格式从以下输入中提取信息：{question}",
        ),
    ]
)


entity_chain = prompt | llm.with_structured_output(Entities)
```

我们可以测试实体提取链。


```python
entities = entity_chain.invoke({"question": "谁在《赌场》电影中出演？"})
entities
```



```output
Entities(names=['赌场'])
```


我们将利用一个简单的 `CONTAINS` 子句将实体匹配到数据库。在实际操作中，您可能希望使用模糊搜索或全文索引以允许轻微的拼写错误。


```python
match_query = """MATCH (p:Person|Movie)
WHERE p.name CONTAINS $value OR p.title CONTAINS $value
RETURN coalesce(p.name, p.title) AS result, labels(p)[0] AS type
LIMIT 1
"""


def map_to_database(entities: Entities) -> Optional[str]:
    result = ""
    for entity in entities.names:
        response = graph.query(match_query, {"value": entity})
        try:
            result += f"{entity} maps to {response[0]['result']} {response[0]['type']} in database\n"
        except IndexError:
            pass
    return result


map_to_database(entities)
```



```output
'赌场 maps to 赌场 Movie in database\n'
```

## 自定义 Cypher 生成链

我们需要定义一个自定义的 Cypher 提示，结合实体映射信息、模式和用户问题来构建 Cypher 语句。我们将使用 LangChain 表达式语言来实现这一点。

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# 根据自然语言输入生成 Cypher 语句
cypher_template = """根据下面的 Neo4j 图模式，编写一个 Cypher 查询以回答用户的问题：
{schema}
问题中的实体映射到以下数据库值：
{entities_list}
问题：{question}
Cypher 查询："""

cypher_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "给定一个输入问题，将其转换为 Cypher 查询。没有前言。",
        ),
        ("human", cypher_template),
    ]
)

cypher_response = (
    RunnablePassthrough.assign(names=entity_chain)
    | RunnablePassthrough.assign(
        entities_list=lambda x: map_to_database(x["names"]),
        schema=lambda _: graph.get_schema,
    )
    | cypher_prompt
    | llm.bind(stop=["\nCypherResult:"])
    | StrOutputParser()
)
```

```python
cypher = cypher_response.invoke({"question": "谁在电影《赌场》中演出？"})
cypher
```

```output
'MATCH (:Movie {title: "Casino"})<-[:ACTED_IN]-(actor)\nRETURN actor.name'
```

## 基于数据库结果生成答案

现在我们有了一个生成 Cypher 语句的链，我们需要针对数据库执行该 Cypher 语句，并将数据库结果发送回 LLM 以生成最终答案。
同样，我们将使用 LCEL。


```python
from langchain.chains.graph_qa.cypher_utils import CypherQueryCorrector, Schema

# Cypher 验证工具，用于关系方向
corrector_schema = [
    Schema(el["start"], el["type"], el["end"])
    for el in graph.structured_schema.get("relationships")
]
cypher_validation = CypherQueryCorrector(corrector_schema)

# 根据数据库结果生成自然语言响应
response_template = """根据问题、Cypher 查询和 Cypher 响应，写出自然语言响应：
问题：{question}
Cypher 查询：{query}
Cypher 响应：{response}"""

response_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "给定一个输入问题和 Cypher 响应，将其转换为自然语言答案。无前言。",
        ),
        ("human", response_template),
    ]
)

chain = (
    RunnablePassthrough.assign(query=cypher_response)
    | RunnablePassthrough.assign(
        response=lambda x: graph.query(cypher_validation(x["query"])),
    )
    | response_prompt
    | llm
    | StrOutputParser()
)
```


```python
chain.invoke({"question": "谁在电影《赌场》中出演？"})
```



```output
'罗伯特·德尼罗、詹姆斯·伍兹、乔·佩西和香农·斯通在电影《赌场》中出演。'
```
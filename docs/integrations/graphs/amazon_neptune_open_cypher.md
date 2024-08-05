---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/graphs/amazon_neptune_open_cypher.ipynb
---

# Amazon Neptune与Cypher

>[Amazon Neptune](https://aws.amazon.com/neptune/) 是一个高性能图形分析和无服务器数据库，具有卓越的可扩展性和可用性。
>
>此示例展示了一个使用 `openCypher` 查询 `Neptune` 图形数据库并返回人类可读响应的QA链。
>
>[Cypher](https://en.wikipedia.org/wiki/Cypher_(query_language)) 是一种声明式图形查询语言，允许在属性图中进行富有表现力和高效的数据查询。
>
>[openCypher](https://opencypher.org/) 是Cypher的开源实现。# Neptune Open Cypher QA Chain
此QA链使用openCypher查询Amazon Neptune并返回人类可读响应

LangChain支持 [Neptune Database](https://docs.aws.amazon.com/neptune/latest/userguide/intro.html) 和 [Neptune Analytics](https://docs.aws.amazon.com/neptune-analytics/latest/userguide/what-is-neptune-analytics.html) ，使用 `NeptuneOpenCypherQAChain` 

Neptune Database 是一个无服务器图形数据库，旨在实现最佳的可扩展性和可用性。它为需要扩展到每秒100,000个查询的图形数据库工作负载提供解决方案，支持多可用区高可用性和多区域部署。您可以将Neptune Database用于社交网络、欺诈警报和客户360应用。

Neptune Analytics 是一个分析数据库引擎，可以快速分析内存中的大量图形数据，以获取洞察和发现趋势。Neptune Analytics 是快速分析现有图形数据库或存储在数据湖中的图形数据集的解决方案。它使用流行的图形分析算法和低延迟分析查询。

## 使用 Neptune 数据库


```python
from langchain_community.graphs import NeptuneGraph

host = "<neptune-host>"
port = 8182
use_https = True

graph = NeptuneGraph(host=host, port=port, use_https=use_https)
```

### 使用 Neptune Analytics


```python
from langchain_community.graphs import NeptuneAnalyticsGraph

graph = NeptuneAnalyticsGraph(graph_identifier="<neptune-analytics-graph-id>")
```

## 使用 NeptuneOpenCypherQAChain

该 QA 链使用 openCypher 查询 Neptune 图数据库并返回人类可读的响应。


```python
from langchain.chains import NeptuneOpenCypherQAChain
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(temperature=0, model="gpt-4")

chain = NeptuneOpenCypherQAChain.from_llm(llm=llm, graph=graph)

chain.invoke("how many outgoing routes does the Austin airport have?")
```



```output
'The Austin airport has 98 outgoing routes.'
```
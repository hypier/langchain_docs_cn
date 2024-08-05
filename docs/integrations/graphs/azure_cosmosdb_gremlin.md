---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/graphs/azure_cosmosdb_gremlin.ipynb
---

# Azure Cosmos DB for Apache Gremlin

>[Azure Cosmos DB for Apache Gremlin](https://learn.microsoft.com/en-us/azure/cosmos-db/gremlin/introduction) 是一种图形数据库服务，可用于存储数十亿个顶点和边的大型图形。您可以以毫秒级延迟查询图形，并轻松演变图形结构。
>
>[Gremlin](https://en.wikipedia.org/wiki/Gremlin_(query_language)) 是由 `Apache TinkerPop` 的 `Apache Software Foundation` 开发的图形遍历语言和虚拟机。

此笔记本展示了如何使用 LLM 提供自然语言接口，以便使用 `Gremlin` 查询语言查询图形数据库。

## 设置

安装一个库：

```python
!pip3 install gremlinpython
```

您需要一个 Azure CosmosDB 图形数据库实例。一个选项是在 Azure 中创建一个 [免费的 CosmosDB 图形数据库实例](https://learn.microsoft.com/en-us/azure/cosmos-db/free-tier)。

当您创建 Cosmos DB 账户和图形时，请使用 `/type` 作为分区键。

```python
cosmosdb_name = "mycosmosdb"
cosmosdb_db_id = "graphtesting"
cosmosdb_db_graph_id = "mygraph"
cosmosdb_access_Key = "longstring=="
```

```python
import nest_asyncio
from langchain.chains.graph_qa.gremlin import GremlinQAChain
from langchain_community.graphs import GremlinGraph
from langchain_community.graphs.graph_document import GraphDocument, Node, Relationship
from langchain_core.documents import Document
from langchain_openai import AzureChatOpenAI
```

```python
graph = GremlinGraph(
    url=f"=wss://{cosmosdb_name}.gremlin.cosmos.azure.com:443/",
    username=f"/dbs/{cosmosdb_db_id}/colls/{cosmosdb_db_graph_id}",
    password=cosmosdb_access_Key,
)
```

## 填充数据库

假设您的数据库是空的，您可以使用 GraphDocuments 来填充它。

对于 Gremlin，始终为每个节点添加名为 'label' 的属性。如果未设置标签，则使用 Node.type 作为标签。对于 Cosmos，使用自然 ID 是有意义的，因为它们在图形浏览器中可见。

```python
source_doc = Document(
    page_content="Matrix is a movie where Keanu Reeves, Laurence Fishburne and Carrie-Anne Moss acted."
)
movie = Node(id="The Matrix", properties={"label": "movie", "title": "The Matrix"})
actor1 = Node(id="Keanu Reeves", properties={"label": "actor", "name": "Keanu Reeves"})
actor2 = Node(
    id="Laurence Fishburne", properties={"label": "actor", "name": "Laurence Fishburne"}
)
actor3 = Node(
    id="Carrie-Anne Moss", properties={"label": "actor", "name": "Carrie-Anne Moss"}
)
rel1 = Relationship(
    id=5, type="ActedIn", source=actor1, target=movie, properties={"label": "ActedIn"}
)
rel2 = Relationship(
    id=6, type="ActedIn", source=actor2, target=movie, properties={"label": "ActedIn"}
)
rel3 = Relationship(
    id=7, type="ActedIn", source=actor3, target=movie, properties={"label": "ActedIn"}
)
rel4 = Relationship(
    id=8,
    type="Starring",
    source=movie,
    target=actor1,
    properties={"label": "Strarring"},
)
rel5 = Relationship(
    id=9,
    type="Starring",
    source=movie,
    target=actor2,
    properties={"label": "Strarring"},
)
rel6 = Relationship(
    id=10,
    type="Straring",
    source=movie,
    target=actor3,
    properties={"label": "Strarring"},
)
graph_doc = GraphDocument(
    nodes=[movie, actor1, actor2, actor3],
    relationships=[rel1, rel2, rel3, rel4, rel5, rel6],
    source=source_doc,
)
```

```python
# The underlying python-gremlin has a problem when running in notebook
# The following line is a workaround to fix the problem
nest_asyncio.apply()

# Add the document to the CosmosDB graph.
graph.add_graph_documents([graph_doc])
```

## 刷新图形架构信息
如果数据库的架构发生变化（在更新之后），您可以刷新架构信息。



```python
graph.refresh_schema()
```


```python
print(graph.schema)
```

## 查询图形

我们现在可以使用 gremlin QA 链来询问图形

```python
chain = GremlinQAChain.from_llm(
    AzureChatOpenAI(
        temperature=0,
        azure_deployment="gpt-4-turbo",
    ),
    graph=graph,
    verbose=True,
)
```

```python
chain.invoke("Who played in The Matrix?")
```

```python
chain.run("How many people played in The Matrix?")
```
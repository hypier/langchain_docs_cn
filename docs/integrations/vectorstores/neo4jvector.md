---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/neo4jvector.ipynb
---

# Neo4j 向量索引

>[Neo4j](https://neo4j.com/) 是一个开源图数据库，集成了向量相似性搜索的支持

它支持：

- 近似最近邻搜索
- 欧几里得相似性和余弦相似性
- 结合向量和关键字搜索的混合搜索

本笔记本展示了如何使用 Neo4j 向量索引 (`Neo4jVector`)。

请参阅 [安装说明](https://neo4j.com/docs/operations-manual/current/installation/)。


```python
# Pip install necessary package
%pip install --upgrade --quiet  neo4j
%pip install --upgrade --quiet  langchain-openai langchain-community
%pip install --upgrade --quiet  tiktoken
```

我们想使用 `OpenAIEmbeddings`，因此我们必须获取 OpenAI API 密钥。


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```
```output
OpenAI API Key: ········
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Neo4jVector
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```


```python
loader = TextLoader("../../how_to/state_of_the_union.txt")

documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```


```python
# Neo4jVector requires the Neo4j database credentials

url = "bolt://localhost:7687"
username = "neo4j"
password = "password"

# You can also use environment variables instead of directly passing named parameters
# os.environ["NEO4J_URI"] = "bolt://localhost:7687"
# os.environ["NEO4J_USERNAME"] = "neo4j"
# os.environ["NEO4J_PASSWORD"] = "pleaseletmein"
```

## 基于余弦距离的相似性搜索（默认）

```python
# Neo4jVector 模块将连接到 Neo4j，并在需要时创建一个向量索引。

db = Neo4jVector.from_documents(
    docs, OpenAIEmbeddings(), url=url, username=username, password=password
)
```

```python
query = "总统对 Ketanji Brown Jackson 说了什么"
docs_with_score = db.similarity_search_with_score(query, k=2)
```

```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```
```output
--------------------------------------------------------------------------------
Score:  0.9076391458511353
今晚。我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯投票权法案》。同时，请通过《披露法案》，让美国人知道谁在资助我们的选举。

今晚，我想向一位为这个国家奉献一生的人致敬：大法官斯蒂芬·布雷耶——一位退伍军人、宪法学者和即将退休的美国最高法院大法官。布雷耶法官，感谢您的服务。

总统最严肃的宪法责任之一就是提名某人担任美国最高法院法官。

四天前，我提名了上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律人才之一，将继续布雷耶法官卓越的遗产。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.8912242650985718
一位曾在私人执业中担任顶级诉讼律师的人。一位前联邦公设辩护人。来自一个公共学校教育工作者和警察的家庭。一个共识建立者。自从她被提名以来，她获得了广泛的支持——从兄弟警察组织到由民主党和共和党任命的前法官。

如果我们要推动自由和正义，我们需要保卫边境并修复移民系统。

我们可以同时做到这两点。在我们的边境，我们安装了新技术，如尖端扫描仪，以更好地检测毒品走私。

我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人贩子。

我们正在设立专门的移民法官，以便逃离迫害和暴力的家庭可以更快地听取他们的案件。

我们正在确保承诺，并支持南美和中美洲的合作伙伴接纳更多难民，并保卫他们自己的边境。
--------------------------------------------------------------------------------
```

## 使用向量存储

上面，我们从头创建了一个向量存储。然而，通常我们希望使用现有的向量存储。为了做到这一点，我们可以直接初始化它。

```python
index_name = "vector"  # 默认索引名称

store = Neo4jVector.from_existing_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name=index_name,
)
```

我们还可以使用 `from_existing_graph` 方法从现有图形初始化一个向量存储。此方法从数据库中提取相关文本信息，并计算并将文本嵌入存储回数据库。

```python
# 首先我们在图中创建示例数据
store.query(
    "CREATE (p:Person {name: 'Tomaz', location:'Slovenia', hobby:'Bicycle', age: 33})"
)
```

```output
[]
```

```python
# 现在我们从现有图形初始化
existing_graph = Neo4jVector.from_existing_graph(
    embedding=OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name="person_index",
    node_label="Person",
    text_node_properties=["name", "location"],
    embedding_node_property="embedding",
)
result = existing_graph.similarity_search("Slovenia", k=1)
```

```python
result[0]
```

```output
Document(page_content='\nname: Tomaz\nlocation: Slovenia', metadata={'age': 33, 'hobby': 'Bicycle'})
```

Neo4j 还支持关系向量索引，其中嵌入作为关系属性存储并被索引。关系向量索引不能通过 LangChain 填充，但您可以将其连接到现有的关系向量索引。

```python
# 首先我们在图中创建示例数据和索引
store.query(
    "MERGE (p:Person {name: 'Tomaz'}) "
    "MERGE (p1:Person {name:'Leann'}) "
    "MERGE (p1)-[:FRIEND {text:'example text', embedding:$embedding}]->(p2)",
    params={"embedding": OpenAIEmbeddings().embed_query("example text")},
)
# 创建一个向量索引
relationship_index = "relationship_vector"
store.query(
    """
CREATE VECTOR INDEX $relationship_index
IF NOT EXISTS
FOR ()-[r:FRIEND]-() ON (r.embedding)
OPTIONS {indexConfig: {
 `vector.dimensions`: 1536,
 `vector.similarity_function`: 'cosine'
}}
""",
    params={"relationship_index": relationship_index},
)
```

```output
[]
```

```python
relationship_vector = Neo4jVector.from_existing_relationship_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name=relationship_index,
    text_node_property="text",
)
relationship_vector.similarity_search("Example")
```

```output
[Document(page_content='example text')]
```

### 元数据过滤

Neo4j 向量存储还支持通过结合并行运行时和精确最近邻搜索进行元数据过滤。  
_需要 Neo4j 5.18 或更高版本。_

相等过滤的语法如下。


```python
existing_graph.similarity_search(
    "Slovenia",
    filter={"hobby": "Bicycle", "name": "Tomaz"},
)
```



```output
[Document(page_content='\nname: Tomaz\nlocation: Slovenia', metadata={'age': 33, 'hobby': 'Bicycle'})]
```


元数据过滤还支持以下运算符：

* `$eq: 等于`
* `$ne: 不等于`
* `$lt: 小于`
* `$lte: 小于或等于`
* `$gt: 大于`
* `$gte: 大于或等于`
* `$in: 在值列表中`
* `$nin: 不在值列表中`
* `$between: 在两个值之间`
* `$like: 文本包含值`
* `$ilike: 小写文本包含值`


```python
existing_graph.similarity_search(
    "Slovenia",
    filter={"hobby": {"$eq": "Bicycle"}, "age": {"$gt": 15}},
)
```



```output
[Document(page_content='\nname: Tomaz\nlocation: Slovenia', metadata={'age': 33, 'hobby': 'Bicycle'})]
```


您还可以在过滤器之间使用 `OR` 运算符


```python
existing_graph.similarity_search(
    "Slovenia",
    filter={"$or": [{"hobby": {"$eq": "Bicycle"}}, {"age": {"$gt": 15}}]},
)
```



```output
[Document(page_content='\nname: Tomaz\nlocation: Slovenia', metadata={'age': 33, 'hobby': 'Bicycle'})]
```

### 添加文档
我们可以向现有的 vectorstore 添加文档。

```python
store.add_documents([Document(page_content="foo")])
```

```output
['acbd18db4cc2f85cedef654fccc4a4d8']
```

```python
docs_with_score = store.similarity_search_with_score("foo")
```

```python
docs_with_score[0]
```

```output
(Document(page_content='foo'), 0.9999997615814209)
```

## 自定义响应与检索查询

您还可以通过使用自定义的 Cypher 代码片段来定制响应，该代码片段可以从图中获取其他信息。
在后台，最终的 Cypher 语句是这样构建的：

```
read_query = (
  "CALL db.index.vector.queryNodes($index, $k, $embedding) "
  "YIELD node, score "
) + retrieval_query
```

检索查询必须返回以下三列：

* `text`: Union[str, Dict] = 用于填充文档的 `page_content` 的值
* `score`: Float = 相似度分数
* `metadata`: Dict = 文档的附加元数据

在这篇 [博客文章](https://medium.com/neo4j/implementing-rag-how-to-write-a-graph-retrieval-query-in-langchain-74abf13044f2) 中了解更多信息。


```python
retrieval_query = """
RETURN "Name:" + node.name AS text, score, {foo:"bar"} AS metadata
"""
retrieval_example = Neo4jVector.from_existing_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name="person_index",
    retrieval_query=retrieval_query,
)
retrieval_example.similarity_search("Foo", k=1)
```



```output
[Document(page_content='Name:Tomaz', metadata={'foo': 'bar'})]
```


这里是一个将除 `embedding` 以外的所有节点属性作为字典传递给 `text` 列的示例，


```python
retrieval_query = """
RETURN node {.name, .age, .hobby} AS text, score, {foo:"bar"} AS metadata
"""
retrieval_example = Neo4jVector.from_existing_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name="person_index",
    retrieval_query=retrieval_query,
)
retrieval_example.similarity_search("Foo", k=1)
```



```output
[Document(page_content='name: Tomaz\nage: 33\nhobby: Bicycle\n', metadata={'foo': 'bar'})]
```


您还可以将 Cypher 参数传递给检索查询。
参数可以用于额外的过滤、遍历等...


```python
retrieval_query = """
RETURN node {.*, embedding:Null, extra: $extra} AS text, score, {foo:"bar"} AS metadata
"""
retrieval_example = Neo4jVector.from_existing_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name="person_index",
    retrieval_query=retrieval_query,
)
retrieval_example.similarity_search("Foo", k=1, params={"extra": "ParamInfo"})
```



```output
[Document(page_content='location: Slovenia\nextra: ParamInfo\nname: Tomaz\nage: 33\nhobby: Bicycle\nembedding: None\n', metadata={'foo': 'bar'})]
```

## 混合搜索（向量 + 关键字）

Neo4j 集成了向量和关键字索引，这使您能够使用混合搜索方法

```python
# The Neo4jVector Module will connect to Neo4j and create a vector and keyword indices if needed.
hybrid_db = Neo4jVector.from_documents(
    docs,
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    search_type="hybrid",
)
```

要从现有索引加载混合搜索，您必须提供向量和关键字索引

```python
index_name = "vector"  # default index name
keyword_index_name = "keyword"  # default keyword index name

store = Neo4jVector.from_existing_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name=index_name,
    keyword_index_name=keyword_index_name,
    search_type="hybrid",
)
```

## 检索器选项

本节展示如何使用 `Neo4jVector` 作为检索器。

```python
retriever = store.as_retriever()
retriever.invoke(query)[0]
```



```output
Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../how_to/state_of_the_union.txt'})
```

## 使用来源的问答

本节介绍如何通过索引进行带来源的问答。它使用 `RetrievalQAWithSourcesChain` 来从索引中查找文档。

```python
from langchain.chains import RetrievalQAWithSourcesChain
from langchain_openai import ChatOpenAI
```

```python
chain = RetrievalQAWithSourcesChain.from_chain_type(
    ChatOpenAI(temperature=0), chain_type="stuff", retriever=retriever
)
```

```python
chain.invoke(
    {"question": "What did the president say about Justice Breyer"},
    return_only_outputs=True,
)
```

```output
{'answer': 'The president honored Justice Stephen Breyer for his service to the country and mentioned his retirement from the United States Supreme Court.\n',
 'sources': '../../how_to/state_of_the_union.txt'}
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
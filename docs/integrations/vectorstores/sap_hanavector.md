---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/sap_hanavector.ipynb
---

# SAP HANA Cloud Vector Engine

>[SAP HANA Cloud Vector Engine](https://www.sap.com/events/teched/news-guide/ai.html#article8) 是一个完全集成到 `SAP HANA Cloud` 数据库中的向量存储。

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 以使用此集成。

## 设置

安装 HANA 数据库驱动程序。


```python
# Pip install necessary package
%pip install --upgrade --quiet  hdbcli
```

对于 `OpenAIEmbeddings`，我们使用来自环境的 OpenAI API 密钥。


```python
import os
# Use OPENAI_API_KEY env variable
# os.environ["OPENAI_API_KEY"] = "Your OpenAI API key"
```

创建到 HANA Cloud 实例的数据库连接。


```python
from hdbcli import dbapi

# Use connection settings from the environment
connection = dbapi.connect(
    address=os.environ.get("HANA_DB_ADDRESS"),
    port=os.environ.get("HANA_DB_PORT"),
    user=os.environ.get("HANA_DB_USER"),
    password=os.environ.get("HANA_DB_PASSWORD"),
    autocommit=True,
    sslValidateCertificate=False,
)
```

## 示例

加载示例文档 "state_of_the_union.txt" 并从中创建块。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.hanavector import HanaDB
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

text_documents = TextLoader("../../how_to/state_of_the_union.txt").load()
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=0)
text_chunks = text_splitter.split_documents(text_documents)
print(f"文档块的数量: {len(text_chunks)}")

embeddings = OpenAIEmbeddings()
```

为 HANA 数据库创建一个 LangChain VectorStore 接口，并指定用于访问向量嵌入的表（集合）。

```python
db = HanaDB(
    embedding=embeddings, connection=connection, table_name="STATE_OF_THE_UNION"
)
```

将加载的文档块添加到表中。对于此示例，我们删除表中可能存在的任何先前内容。

```python
# 删除表中已存在的文档
db.delete(filter={})

# 添加加载的文档块
db.add_documents(text_chunks)
```

执行查询以获取在上一步中添加的两个最佳匹配文档块。默认情况下，搜索使用 "余弦相似度"。

```python
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = db.similarity_search(query, k=2)

for doc in docs:
    print("-" * 80)
    print(doc.page_content)
```

使用 "欧几里得距离" 查询相同的内容。结果应与 "余弦相似度" 相同。

```python
from langchain_community.vectorstores.utils import DistanceStrategy

db = HanaDB(
    embedding=embeddings,
    connection=connection,
    distance_strategy=DistanceStrategy.EUCLIDEAN_DISTANCE,
    table_name="STATE_OF_THE_UNION",
)

query = "总统对 Ketanji Brown Jackson 说了什么"
docs = db.similarity_search(query, k=2)
for doc in docs:
    print("-" * 80)
    print(doc.page_content)
```

## 最大边际相关性搜索 (MMR)

`最大边际相关性` 在优化查询相似性和所选文档的多样性之间进行平衡。将从数据库中检索前 20 (fetch_k) 项。然后，MMR 算法将找到最佳的 2 (k) 个匹配项。

```python
docs = db.max_marginal_relevance_search(query, k=2, fetch_k=20)
for doc in docs:
    print("-" * 80)
    print(doc.page_content)
```

## 基本向量存储操作


```python
db = HanaDB(
    connection=connection, embedding=embeddings, table_name="LANGCHAIN_DEMO_BASIC"
)

# 从表中删除已经存在的文档
db.delete(filter={})
```

我们可以将简单的文本文档添加到现有表中。


```python
docs = [Document(page_content="Some text"), Document(page_content="Other docs")]
db.add_documents(docs)
```

添加带有元数据的文档。


```python
docs = [
    Document(
        page_content="foo",
        metadata={"start": 100, "end": 150, "doc_name": "foo.txt", "quality": "bad"},
    ),
    Document(
        page_content="bar",
        metadata={"start": 200, "end": 250, "doc_name": "bar.txt", "quality": "good"},
    ),
]
db.add_documents(docs)
```

查询具有特定元数据的文档。


```python
docs = db.similarity_search("foobar", k=2, filter={"quality": "bad"})
# 通过过滤“quality”==“bad”，应该只返回一个文档
for doc in docs:
    print("-" * 80)
    print(doc.page_content)
    print(doc.metadata)
```

删除具有特定元数据的文档。


```python
db.delete(filter={"quality": "bad"})

# 现在使用相同过滤条件的相似性搜索将返回零结果
docs = db.similarity_search("foobar", k=2, filter={"quality": "bad"})
print(len(docs))
```

## 高级过滤
除了基本的基于值的过滤功能外，还可以使用更高级的过滤。
下表显示了可用的过滤运算符。

| 运算符   | 语义                     |
|----------|-------------------------|
| `$eq`    | 等于 (==)               |
| `$ne`    | 不等于 (!=)             |
| `$lt`    | 小于 (<)                 |
| `$lte`   | 小于或等于 (<=)         |
| `$gt`    | 大于 (>)                |
| `$gte`   | 大于或等于 (>=)         |
| `$in`    | 包含在给定值的集合中 (in) |
| `$nin`   | 不包含在给定值的集合中 (not in) |
| `$between` | 在两个边界值的范围内   |
| `$like`  | 基于 SQL 中 "LIKE" 语义的文本相等 (使用 "%" 作为通配符) |
| `$and`   | 逻辑 "与"，支持 2 个或更多操作数 |
| `$or`    | 逻辑 "或"，支持 2 个或更多操作数 |


```python
# Prepare some test documents
docs = [
    Document(
        page_content="First",
        metadata={"name": "adam", "is_active": True, "id": 1, "height": 10.0},
    ),
    Document(
        page_content="Second",
        metadata={"name": "bob", "is_active": False, "id": 2, "height": 5.7},
    ),
    Document(
        page_content="Third",
        metadata={"name": "jane", "is_active": True, "id": 3, "height": 2.4},
    ),
]

db = HanaDB(
    connection=connection,
    embedding=embeddings,
    table_name="LANGCHAIN_DEMO_ADVANCED_FILTER",
)

# Delete already existing documents from the table
db.delete(filter={})
db.add_documents(docs)


# Helper function for printing filter results
def print_filter_result(result):
    if len(result) == 0:
        print("<empty result>")
    for doc in result:
        print(doc.metadata)
```

使用 `$ne`、`$gt`、`$gte`、`$lt`、`$lte` 进行过滤


```python
advanced_filter = {"id": {"$ne": 1}}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))

advanced_filter = {"id": {"$gt": 1}}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))

advanced_filter = {"id": {"$gte": 1}}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))

advanced_filter = {"id": {"$lt": 1}}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))

advanced_filter = {"id": {"$lte": 1}}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
```

使用 `$between`、`$in`、`$nin` 进行过滤


```python
advanced_filter = {"id": {"$between": (1, 2)}}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))

advanced_filter = {"name": {"$in": ["adam", "bob"]}}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))

advanced_filter = {"name": {"$nin": ["adam", "bob"]}}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
```

使用 `$like` 进行文本过滤


```python
advanced_filter = {"name": {"$like": "a%"}}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))

advanced_filter = {"name": {"$like": "%a%"}}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
```

使用 `$and`、`$or` 进行组合过滤


```python
advanced_filter = {"$or": [{"id": 1}, {"name": "bob"}]}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))

advanced_filter = {"$and": [{"id": 1}, {"id": 2}]}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))

advanced_filter = {"$or": [{"id": 1}, {"id": 2}, {"id": 3}]}
print(f"Filter: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
```

## 使用 VectorStore 作为链中的检索器进行检索增强生成 (RAG)

```python
from langchain.memory import ConversationBufferMemory
from langchain_openai import ChatOpenAI

# 通过新表访问向量数据库
db = HanaDB(
    connection=connection,
    embedding=embeddings,
    table_name="LANGCHAIN_DEMO_RETRIEVAL_CHAIN",
)

# 删除表中已存在的条目
db.delete(filter={})

# 从“国情咨文”文件中添加加载的文档片段
db.add_documents(text_chunks)

# 创建向量存储的检索器实例
retriever = db.as_retriever()
```

定义提示。

```python
from langchain_core.prompts import PromptTemplate

prompt_template = """
您是国情咨文主题的专家。您将获得多个与您必须回答的提示相关的上下文项目。
使用以下上下文内容回答最后的问题。

'''
{context}
'''

问题: {question}
"""

PROMPT = PromptTemplate(
    template=prompt_template, input_variables=["context", "question"]
)
chain_type_kwargs = {"prompt": PROMPT}
```

创建 ConversationalRetrievalChain，它处理聊天历史和检索类似的文档片段以添加到提示中。

```python
from langchain.chains import ConversationalRetrievalChain

llm = ChatOpenAI(model="gpt-3.5-turbo")
memory = ConversationBufferMemory(
    memory_key="chat_history", output_key="answer", return_messages=True
)
qa_chain = ConversationalRetrievalChain.from_llm(
    llm,
    db.as_retriever(search_kwargs={"k": 5}),
    return_source_documents=True,
    memory=memory,
    verbose=False,
    combine_docs_chain_kwargs={"prompt": PROMPT},
)
```

提出第一个问题（并验证使用了多少文本片段）。

```python
question = "关于墨西哥和危地马拉怎么样？"

result = qa_chain.invoke({"question": question})
print("来自 LLM 的回答:")
print("================")
print(result["answer"])

source_docs = result["source_documents"]
print("================")
print(f"使用的源文档片段数量: {len(source_docs)}")
```

详细检查链中使用的片段。检查排名最高的片段是否包含关于“墨西哥和危地马拉”的信息，如问题中提到的。

```python
for doc in source_docs:
    print("-" * 80)
    print(doc.page_content)
    print(doc.metadata)
```

在同一对话链上提出另一个问题。答案应与先前给出的答案相关。

```python
question = "其他国家怎么样？"

result = qa_chain.invoke({"question": question})
print("来自 LLM 的回答:")
print("================")
print(result["answer"])
```

## 标准表与带有向量数据的“自定义”表

默认情况下，嵌入的表创建了3列：

- 一列 `VEC_TEXT`，包含文档的文本
- 一列 `VEC_META`，包含文档的元数据
- 一列 `VEC_VECTOR`，包含文档文本的嵌入向量


```python
# Access the vector DB with a new table
db = HanaDB(
    connection=connection, embedding=embeddings, table_name="LANGCHAIN_DEMO_NEW_TABLE"
)

# Delete already existing entries from the table
db.delete(filter={})

# Add a simple document with some metadata
docs = [
    Document(
        page_content="A simple document",
        metadata={"start": 100, "end": 150, "doc_name": "simple.txt"},
    )
]
db.add_documents(docs)
```

显示表“LANGCHAIN_DEMO_NEW_TABLE”中的列


```python
cur = connection.cursor()
cur.execute(
    "SELECT COLUMN_NAME, DATA_TYPE_NAME FROM SYS.TABLE_COLUMNS WHERE SCHEMA_NAME = CURRENT_SCHEMA AND TABLE_NAME = 'LANGCHAIN_DEMO_NEW_TABLE'"
)
rows = cur.fetchall()
for row in rows:
    print(row)
cur.close()
```

显示插入文档在三列中的值


```python
cur = connection.cursor()
cur.execute(
    "SELECT VEC_TEXT, VEC_META, TO_NVARCHAR(VEC_VECTOR) FROM LANGCHAIN_DEMO_NEW_TABLE LIMIT 1"
)
rows = cur.fetchall()
print(rows[0][0])  # The text
print(rows[0][1])  # The metadata
print(rows[0][2])  # The vector
cur.close()
```

自定义表必须至少有三列，与标准表的语义匹配

- 一列类型为 `NCLOB` 或 `NVARCHAR` 的嵌入文本/上下文
- 一列类型为 `NCLOB` 或 `NVARCHAR` 的元数据
- 一列类型为 `REAL_VECTOR` 的嵌入向量

该表可以包含其他列。当新文档插入表中时，这些附加列必须允许 NULL 值。


```python
# Create a new table "MY_OWN_TABLE" with three "standard" columns and one additional column
my_own_table_name = "MY_OWN_TABLE"
cur = connection.cursor()
cur.execute(
    (
        f"CREATE TABLE {my_own_table_name} ("
        "SOME_OTHER_COLUMN NVARCHAR(42), "
        "MY_TEXT NVARCHAR(2048), "
        "MY_METADATA NVARCHAR(1024), "
        "MY_VECTOR REAL_VECTOR )"
    )
)

# Create a HanaDB instance with the own table
db = HanaDB(
    connection=connection,
    embedding=embeddings,
    table_name=my_own_table_name,
    content_column="MY_TEXT",
    metadata_column="MY_METADATA",
    vector_column="MY_VECTOR",
)

# Add a simple document with some metadata
docs = [
    Document(
        page_content="Some other text",
        metadata={"start": 400, "end": 450, "doc_name": "other.txt"},
    )
]
db.add_documents(docs)

# Check if data has been inserted into our own table
cur.execute(f"SELECT * FROM {my_own_table_name} LIMIT 1")
rows = cur.fetchall()
print(rows[0][0])  # Value of column "SOME_OTHER_DATA". Should be NULL/None
print(rows[0][1])  # The text
print(rows[0][2])  # The metadata
print(rows[0][3])  # The vector

cur.close()
```

添加另一个文档并在自定义表上执行相似性搜索。


```python
docs = [
    Document(
        page_content="Some more text",
        metadata={"start": 800, "end": 950, "doc_name": "more.txt"},
    )
]
db.add_documents(docs)

query = "What's up?"
docs = db.similarity_search(query, k=2)
for doc in docs:
    print("-" * 80)
    print(doc.page_content)
```

### 自定义列的过滤性能优化

为了允许灵活的元数据值，所有元数据默认存储为 JSON 格式在元数据列中。如果已知某些使用的元数据键和类型，可以通过创建目标表并将键名作为列名，将它们存储在额外的列中，并通过 specific_metadata_columns 列表将它们传递给 HanaDB 构造函数。在插入时，匹配这些值的元数据键会被复制到特殊列中。过滤器使用特殊列而不是元数据 JSON 列来获取 specific_metadata_columns 列表中的键。

```python
# Create a new table "PERFORMANT_CUSTOMTEXT_FILTER" with three "standard" columns and one additional column
my_own_table_name = "PERFORMANT_CUSTOMTEXT_FILTER"
cur = connection.cursor()
cur.execute(
    (
        f"CREATE TABLE {my_own_table_name} ("
        "CUSTOMTEXT NVARCHAR(500), "
        "MY_TEXT NVARCHAR(2048), "
        "MY_METADATA NVARCHAR(1024), "
        "MY_VECTOR REAL_VECTOR )"
    )
)

# Create a HanaDB instance with the own table
db = HanaDB(
    connection=connection,
    embedding=embeddings,
    table_name=my_own_table_name,
    content_column="MY_TEXT",
    metadata_column="MY_METADATA",
    vector_column="MY_VECTOR",
    specific_metadata_columns=["CUSTOMTEXT"],
)

# Add a simple document with some metadata
docs = [
    Document(
        page_content="Some other text",
        metadata={
            "start": 400,
            "end": 450,
            "doc_name": "other.txt",
            "CUSTOMTEXT": "Filters on this value are very performant",
        },
    )
]
db.add_documents(docs)

# Check if data has been inserted into our own table
cur.execute(f"SELECT * FROM {my_own_table_name} LIMIT 1")
rows = cur.fetchall()
print(
    rows[0][0]
)  # Value of column "CUSTOMTEXT". Should be "Filters on this value are very performant"
print(rows[0][1])  # The text
print(
    rows[0][2]
)  # The metadata without the "CUSTOMTEXT" data, as this is extracted into a sperate column
print(rows[0][3])  # The vector

cur.close()
```

特殊列对 langchain 接口的其余部分是完全透明的。一切都像以前一样工作，只是性能更高。

```python
docs = [
    Document(
        page_content="Some more text",
        metadata={
            "start": 800,
            "end": 950,
            "doc_name": "more.txt",
            "CUSTOMTEXT": "Another customtext value",
        },
    )
]
db.add_documents(docs)

advanced_filter = {"CUSTOMTEXT": {"$like": "%value%"}}
query = "What's up?"
docs = db.similarity_search(query, k=2, filter=advanced_filter)
for doc in docs:
    print("-" * 80)
    print(doc.page_content)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
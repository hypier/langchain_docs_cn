---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/timescalevector.ipynb
---

# Timescale Vector (Postgres)

>[Timescale Vector](https://www.timescale.com/ai?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 是用于 AI 应用的 `PostgreSQL++` 向量数据库。

本笔记本展示了如何使用 Postgres 向量数据库 `Timescale Vector`。您将学习如何使用 TimescaleVector 进行 (1) 语义搜索，(2) 基于时间的向量搜索，(3) 自我查询，以及 (4) 如何创建索引以加速查询。

## 什么是 Timescale Vector？

`Timescale Vector` 使您能够在 `PostgreSQL` 中高效存储和查询数百万个向量嵌入。
- 通过受 `DiskANN` 启发的索引算法，增强了 `pgvector`，在 100M+ 向量上实现更快和更准确的相似性搜索。
- 通过自动时间分区和索引，支持快速的基于时间的向量搜索。
- 提供熟悉的 SQL 接口，用于查询向量嵌入和关系数据。

`Timescale Vector` 是适用于 AI 的云 `PostgreSQL`，可以随着您从 POC 到生产的扩展而扩展：
- 通过使您能够在单个数据库中存储关系元数据、向量嵌入和时间序列数据，简化操作。
- 受益于坚如磐石的 PostgreSQL 基础，具有企业级功能，如流式备份和复制、高可用性和行级安全性。
- 提供无忧体验，具备企业级安全性和合规性。

## 如何访问 Timescale Vector

`Timescale Vector` 可在 [Timescale](https://www.timescale.com/ai?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 上使用，这是一个云 PostgreSQL 平台。（目前没有自托管版本。）

LangChain 用户可以获得 90 天的 Timescale Vector 免费试用。
- 要开始，请 [注册](https://console.cloud.timescale.com/signup?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) Timescale，创建一个新数据库并按照此笔记本操作！
- 有关更多详细信息和性能基准，请参阅 [Timescale Vector 解释博客](https://www.timescale.com/blog/how-we-made-postgresql-the-best-vector-database/?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral)。
- 有关在 Python 中使用 Timescale Vector 的更多详细信息，请参见 [安装说明](https://github.com/timescale/python-vector)。

## 设置

按照以下步骤准备好以跟随本教程。

```python
# Pip install necessary packages
%pip install --upgrade --quiet  timescale-vector
%pip install --upgrade --quiet  langchain-openai langchain-community
%pip install --upgrade --quiet  tiktoken
```

在此示例中，我们将使用 `OpenAIEmbeddings`，因此请加载您的 OpenAI API 密钥。

```python
import os

# Run export OPENAI_API_KEY=sk-YOUR_OPENAI_API_KEY...
# Get openAI api key by reading local .env file
from dotenv import find_dotenv, load_dotenv

_ = load_dotenv(find_dotenv())
OPENAI_API_KEY = os.environ["OPENAI_API_KEY"]
```

```python
# Get the API key and save it as an environment variable
# import os
# import getpass
# os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
from typing import Tuple
```

接下来，我们将导入所需的 Python 库和 LangChain 库。请注意，我们导入 `timescale-vector` 库以及 TimescaleVector LangChain 向量存储。

```python
from datetime import datetime, timedelta

from langchain_community.document_loaders import TextLoader
from langchain_community.document_loaders.json_loader import JSONLoader
from langchain_community.vectorstores.timescalevector import TimescaleVector
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

## 1. 使用欧几里得距离进行相似性搜索（默认）

首先，我们将查看一个在国情咨文演讲中进行相似性搜索查询的示例，以查找与给定查询句子最相似的句子。我们将使用 [欧几里得距离](https://en.wikipedia.org/wiki/Euclidean_distance) 作为我们的相似性度量标准。

```python
# Load the text and split it into chunks
loader = TextLoader("../../../extras/modules/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```

接下来，我们将加载 Timescale 数据库的服务 URL。

如果您还没有，请 [注册 Timescale](https://console.cloud.timescale.com/signup?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral)，并创建一个新数据库。

然后，要连接到您的 PostgreSQL 数据库，您需要服务 URI，该 URI 可以在创建新数据库后下载的备忘单或 `.env` 文件中找到。

URI 的格式如下：`postgres://tsdbadmin:<password>@<id>.tsdb.cloud.timescale.com:<port>/tsdb?sslmode=require`。

```python
# Timescale Vector needs the service url to your cloud database. You can see this as soon as you create the
# service in the cloud UI or in your credentials.sql file
SERVICE_URL = os.environ["TIMESCALE_SERVICE_URL"]

# Specify directly if testing
# SERVICE_URL = "postgres://tsdbadmin:<password>@<id>.tsdb.cloud.timescale.com:<port>/tsdb?sslmode=require"

# # You can get also it from an environment variables. We suggest using a .env file.
# import os
# SERVICE_URL = os.environ.get("TIMESCALE_SERVICE_URL", "")
```

接下来，我们创建一个 TimescaleVector 向量存储。我们指定一个集合名称，该名称将是我们数据存储的表名。

注意：在创建 TimescaleVector 的新实例时，TimescaleVector 模块将尝试创建一个与集合名称相同的表。因此，请确保集合名称是唯一的（即它不存在）。

```python
# The TimescaleVector Module will create a table with the name of the collection.
COLLECTION_NAME = "state_of_the_union_test"

# Create a Timescale Vector instance from the collection of documents
db = TimescaleVector.from_documents(
    embedding=embeddings,
    documents=docs,
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
)
```

现在我们已经加载了数据，可以进行相似性搜索。

```python
query = "What did the president say about Ketanji Brown Jackson"
docs_with_score = db.similarity_search_with_score(query)
```

```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```

### 使用 Timescale Vector 作为检索器
在初始化 TimescaleVector 存储后，您可以将其用作 [检索器](/docs/how_to#retrievers)。

```python
# Use TimescaleVector as a retriever
retriever = db.as_retriever()
```

```python
print(retriever)
```
```output
tags=['TimescaleVector', 'OpenAIEmbeddings'] metadata=None vectorstore=<langchain_community.vectorstores.timescalevector.TimescaleVector object at 0x10fc8d070> search_type='similarity' search_kwargs={}
```
让我们看一个使用 Timescale Vector 作为检索器的示例，结合 RetrievalQA 链和内容文档链。

在这个例子中，我们将提出与上面相同的问题，但这次我们会将从 Timescale Vector 返回的相关文档传递给 LLM，以用作回答我们问题的上下文。

首先，我们将创建我们的内容链：

```python
# Initialize GPT3.5 model
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(temperature=0.1, model="gpt-3.5-turbo-16k")

# Initialize a RetrievalQA class from a stuff chain
from langchain.chains import RetrievalQA

qa_stuff = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=retriever,
    verbose=True,
)
```

```python
query = "What did the president say about Ketanji Brown Jackson?"
response = qa_stuff.run(query)
```
```output


[1m> Entering new RetrievalQA chain...[0m

[1m> Finished chain.[0m
```

```python
print(response)
```
```output
总统表示，他提名了巡回上诉法院法官 Ketanji Brown Jackson，她是我们国家顶尖的法律人才之一，将继续布雷耶法官的卓越遗产。他还提到，自她被提名以来，她得到了包括警察兄弟会和由民主党和共和党任命的前法官在内的各个团体的广泛支持。
```

## 2. 基于时间过滤的相似性搜索

Timescale Vector 的一个关键用例是高效的基于时间的向量搜索。Timescale Vector 通过按时间自动对向量（及相关元数据）进行分区来实现这一点。这使您能够通过与查询向量的相似性和时间高效地查询向量。

基于时间的向量搜索功能对以下应用程序非常有帮助：
- 存储和检索 LLM 响应历史（例如聊天机器人）
- 查找与查询向量相似的最新嵌入（例如最近的新闻）
- 将相似性搜索限制在相关的时间范围内（例如对知识库提出基于时间的问题）

为了说明如何使用 TimescaleVector 的基于时间的向量搜索功能，我们将询问有关 TimescaleDB 的 git 日志历史的问题。我们将演示如何添加带有时间戳的 uuid 的文档，以及如何使用时间范围过滤器运行相似性搜索。

### 从 git log JSON 中提取内容和元数据
首先，让我们将 git log 数据加载到名为 `timescale_commits` 的 PostgreSQL 数据库中的新集合中。

我们将定义一个辅助函数，根据文档的时间戳为文档和相关的向量嵌入创建一个 uuid。我们将使用此函数为每个 git log 条目创建一个 uuid。

重要提示：如果您正在处理文档并希望将当前日期和时间与向量关联以进行基于时间的搜索，可以跳过此步骤。默认情况下，文档被摄取时将自动生成一个 uuid。

```python
from timescale_vector import client


# Function to take in a date string in the past and return a uuid v1
def create_uuid(date_string: str):
    if date_string is None:
        return None
    time_format = "%a %b %d %H:%M:%S %Y %z"
    datetime_obj = datetime.strptime(date_string, time_format)
    uuid = client.uuid_from_time(datetime_obj)
    return str(uuid)
```

接下来，我们将定义一个元数据函数，从 JSON 记录中提取相关的元数据。我们将把这个函数传递给 JSONLoader。有关更多详细信息，请参见 [JSON 文档加载器文档](/docs/how_to/document_loader_json)。

```python
# Helper function to split name and email given an author string consisting of Name Lastname <email>
def split_name(input_string: str) -> Tuple[str, str]:
    if input_string is None:
        return None, None
    start = input_string.find("<")
    end = input_string.find(">")
    name = input_string[:start].strip()
    email = input_string[start + 1 : end].strip()
    return name, email


# Helper function to transform a date string into a timestamp_tz string
def create_date(input_string: str) -> datetime:
    if input_string is None:
        return None
    # Define a dictionary to map month abbreviations to their numerical equivalents
    month_dict = {
        "Jan": "01",
        "Feb": "02",
        "Mar": "03",
        "Apr": "04",
        "May": "05",
        "Jun": "06",
        "Jul": "07",
        "Aug": "08",
        "Sep": "09",
        "Oct": "10",
        "Nov": "11",
        "Dec": "12",
    }

    # Split the input string into its components
    components = input_string.split()
    # Extract relevant information
    day = components[2]
    month = month_dict[components[1]]
    year = components[4]
    time = components[3]
    timezone_offset_minutes = int(components[5])  # Convert the offset to minutes
    timezone_hours = timezone_offset_minutes // 60  # Calculate the hours
    timezone_minutes = timezone_offset_minutes % 60  # Calculate the remaining minutes
    # Create a formatted string for the timestamptz in PostgreSQL format
    timestamp_tz_str = (
        f"{year}-{month}-{day} {time}+{timezone_hours:02}{timezone_minutes:02}"
    )
    return timestamp_tz_str


# Metadata extraction function to extract metadata from a JSON record
def extract_metadata(record: dict, metadata: dict) -> dict:
    record_name, record_email = split_name(record["author"])
    metadata["id"] = create_uuid(record["date"])
    metadata["date"] = create_date(record["date"])
    metadata["author_name"] = record_name
    metadata["author_email"] = record_email
    metadata["commit_hash"] = record["commit"]
    return metadata
```

接下来，您需要 [下载示例数据集](https://s3.amazonaws.com/assets.timescale.com/ai/ts_git_log.json) 并将其放在与此笔记本相同的目录中。

您可以使用以下命令：

```python
# Download the file using curl and save it as commit_history.csv
# Note: Execute this command in your terminal, in the same directory as the notebook
!curl -O https://s3.amazonaws.com/assets.timescale.com/ai/ts_git_log.json
```

最后，我们可以初始化 JSON 加载器以解析 JSON 记录。我们还删除空记录以简化操作。

```python
# Define path to the JSON file relative to this notebook
# Change this to the path to your JSON file
FILE_PATH = "../../../../../ts_git_log.json"

# Load data from JSON file and extract metadata
loader = JSONLoader(
    file_path=FILE_PATH,
    jq_schema=".commit_history[]",
    text_content=False,
    metadata_func=extract_metadata,
)
documents = loader.load()

# Remove documents with None dates
documents = [doc for doc in documents if doc.metadata["date"] is not None]
```

```python
print(documents[0])
```
```output
page_content='{"commit": "44e41c12ab25e36c202f58e068ced262eadc8d16", "author": "Lakshmi Narayanan Sreethar<lakshmi@timescale.com>", "date": "Tue Sep 5 21:03:21 2023 +0530", "change summary": "Fix segfault in set_integer_now_func", "change details": "When an invalid function oid is passed to set_integer_now_func, it finds out that the function oid is invalid but before throwing the error, it calls ReleaseSysCache on an invalid tuple causing a segfault. Fixed that by removing the invalid call to ReleaseSysCache.  Fixes #6037 "}' metadata={'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/ts_git_log.json', 'seq_num': 1, 'id': '8b407680-4c01-11ee-96a6-b82284ddccc6', 'date': '2023-09-5 21:03:21+0850', 'author_name': 'Lakshmi Narayanan Sreethar', 'author_email': 'lakshmi@timescale.com', 'commit_hash': '44e41c12ab25e36c202f58e068ced262eadc8d16'}
```

### 将文档和元数据加载到 TimescaleVector 向量存储中
现在我们已经准备好了文档，让我们处理它们并将它们及其向量嵌入表示加载到我们的 TimescaleVector 向量存储中。

由于这是一个演示，我们将只加载前 500 条记录。在实际操作中，您可以加载任意数量的记录。


```python
NUM_RECORDS = 500
documents = documents[:NUM_RECORDS]
```

然后，我们使用 CharacterTextSplitter 将文档拆分成更小的块，以便于嵌入。请注意，这个拆分过程保留了每个文档的元数据。


```python
# Split the documents into chunks for embedding
text_splitter = CharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
)
docs = text_splitter.split_documents(documents)
```

接下来，我们将从完成预处理的文档集合中创建一个 Timescale Vector 实例。

首先，我们将定义一个集合名称，这将是我们在 PostgreSQL 数据库中的表名。

我们还将定义一个时间增量，该增量将传递给 `time_partition_interval` 参数，用于按时间对数据进行分区。每个分区将包含指定时间长度的数据。为了简单起见，我们将使用 7 天，但您可以选择适合您用例的任何值——例如，如果您频繁查询最近的向量，您可能希望使用 1 天这样较小的时间增量，或者如果您查询跨越十年的向量，您可能希望使用 6 个月或 1 年这样较大的时间增量。

最后，我们将创建 TimescaleVector 实例。我们指定 `ids` 参数为我们在上述预处理步骤中创建的元数据中的 `uuid` 字段。我们这样做是因为我们希望 uuid 的时间部分反映过去的日期（即提交时的日期）。但是，如果我们希望将当前日期和时间与我们的文档相关联，我们可以删除 id 参数，uuid 将自动使用当前日期和时间创建。


```python
# Define collection name
COLLECTION_NAME = "timescale_commits"
embeddings = OpenAIEmbeddings()

# Create a Timescale Vector instance from the collection of documents
db = TimescaleVector.from_documents(
    embedding=embeddings,
    ids=[doc.metadata["id"] for doc in docs],
    documents=docs,
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
    time_partition_interval=timedelta(days=7),
)
```

### 按时间和相似性查询向量

现在我们已经将文档加载到 TimescaleVector 中，可以通过时间和相似性对其进行查询。

TimescaleVector 提供了多种方法，通过时间过滤进行相似性搜索。

让我们看看下面的每种方法：


```python
# Time filter variables
start_dt = datetime(2023, 8, 1, 22, 10, 35)  # Start date = 1 August 2023, 22:10:35
end_dt = datetime(2023, 8, 30, 22, 10, 35)  # End date = 30 August 2023, 22:10:35
td = timedelta(days=7)  # Time delta = 7 days

query = "What's new with TimescaleDB functions?"
```

方法 1：在提供的开始日期和结束日期之间进行过滤。



```python
# Method 1: Query for vectors between start_date and end_date
docs_with_score = db.similarity_search_with_score(
    query, start_date=start_dt, end_date=end_dt
)

for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print("Date: ", doc.metadata["date"])
    print(doc.page_content)
    print("-" * 80)
```

请注意，查询仅返回指定日期范围内的结果。

方法 2：在提供的开始日期和之后的时间差内进行过滤。


```python
# Method 2: Query for vectors between start_dt and a time delta td later
# Most relevant vectors between 1 August and 7 days later
docs_with_score = db.similarity_search_with_score(
    query, start_date=start_dt, time_delta=td
)

for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print("Date: ", doc.metadata["date"])
    print(doc.page_content)
    print("-" * 80)
```

再次注意，我们在指定的时间过滤内得到了结果，这与之前的查询不同。

方法 3：在提供的结束日期和之前的时间差内进行过滤。


```python
# Method 3: Query for vectors between end_dt and a time delta td earlier
# Most relevant vectors between 30 August and 7 days earlier
docs_with_score = db.similarity_search_with_score(query, end_date=end_dt, time_delta=td)

for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print("Date: ", doc.metadata["date"])
    print(doc.page_content)
    print("-" * 80)
```

方法 4：我们还可以通过在查询中仅指定开始日期来过滤所有给定日期之后的向量。

方法 5：类似地，我们可以通过在查询中仅指定结束日期来过滤所有给定日期之前的向量。


```python
# Method 4: Query all vectors after start_date
docs_with_score = db.similarity_search_with_score(query, start_date=start_dt)

for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print("Date: ", doc.metadata["date"])
    print(doc.page_content)
    print("-" * 80)
```


```python
# Method 5: Query all vectors before end_date
docs_with_score = db.similarity_search_with_score(query, end_date=end_dt)

for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print("Date: ", doc.metadata["date"])
    print(doc.page_content)
    print("-" * 80)
```

主要的结论是，在上述每个结果中，仅返回指定时间范围内的向量。这些查询非常高效，因为它们只需要搜索相关的分区。

我们还可以使用此功能进行问答，在这种情况下，我们希望找到指定时间范围内最相关的向量，以用作回答问题的上下文。让我们看看下面的示例，使用 Timescale Vector 作为检索器：


```python
# Set timescale vector as a retriever and specify start and end dates via kwargs
retriever = db.as_retriever(search_kwargs={"start_date": start_dt, "end_date": end_dt})
```


```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(temperature=0.1, model="gpt-3.5-turbo-16k")

from langchain.chains import RetrievalQA

qa_stuff = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=retriever,
    verbose=True,
)

query = (
    "What's new with the timescaledb functions? Tell me when these changes were made."
)
response = qa_stuff.run(query)
print(response)
```
```output


[1m> Entering new RetrievalQA chain...[0m

[1m> Finished chain.[0m
The following changes were made to the timescaledb functions:

1. "Add compatibility layer for _timescaledb_internal functions" - This change was made on Tue Aug 29 18:13:24 2023 +0200.
2. "Move functions to _timescaledb_functions schema" - This change was made on Sun Aug 20 22:47:10 2023 +0200.
3. "Move utility functions to _timescaledb_functions schema" - This change was made on Tue Aug 22 12:01:19 2023 +0200.
4. "Move partitioning functions to _timescaledb_functions schema" - This change was made on Tue Aug 29 10:49:47 2023 +0200.
```
请注意，LLM 用于构成答案的上下文仅来自于在指定日期范围内检索到的文档。

这展示了如何使用 Timescale Vector 通过检索与查询相关的时间范围内的文档来增强检索增强生成。

## 3. 使用 ANN 搜索索引加速查询

通过在嵌入列上创建索引，可以加速相似性查询。只有在摄取了大量数据后，才应执行此操作。

Timescale Vector 支持以下索引：
- timescale_vector 索引 (tsv)：一种受 disk-ann 启发的图形索引，用于快速相似性搜索（默认）。
- pgvector 的 HNSW 索引：一种层次可导航的小世界图索引，用于快速相似性搜索。
- pgvector 的 IVFFLAT 索引：一种倒排文件索引，用于快速相似性搜索。

重要提示：在 PostgreSQL 中，每个表只能在特定列上拥有一个索引。因此，如果您想测试不同索引类型的性能，可以通过以下方式进行：(1) 创建多个具有不同索引的表，(2) 在同一表中创建多个向量列并在每列上创建不同的索引，或者 (3) 删除并重新创建同一列上的索引并比较结果。

```python
# Initialize an existing TimescaleVector store
COLLECTION_NAME = "timescale_commits"
embeddings = OpenAIEmbeddings()
db = TimescaleVector(
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
    embedding_function=embeddings,
)
```

使用 `create_index()` 函数而不带额外参数将默认创建一个 timescale_vector_index，使用默认参数。

```python
# create an index
# by default this will create a Timescale Vector (DiskANN) index
db.create_index()
```

您还可以指定索引的参数。有关不同参数及其对性能影响的完整讨论，请参阅 Timescale Vector 文档。

注意：您不需要指定参数，因为我们设置了智能默认值。但如果您希望针对特定数据集进行实验以获得更好的性能，您始终可以指定自己的参数。

```python
# drop the old index
db.drop_index()

# create an index
# Note: You don't need to specify m and ef_construction parameters as we set smart defaults.
db.create_index(index_type="tsv", max_alpha=1.0, num_neighbors=50)
```

Timescale Vector 还支持 HNSW ANN 索引算法，以及 ivfflat ANN 索引算法。只需在 `index_type` 参数中指定要创建的索引，并可选地指定索引的参数。

```python
# drop the old index
db.drop_index()

# Create an HNSW index
# Note: You don't need to specify m and ef_construction parameters as we set smart defaults.
db.create_index(index_type="hnsw", m=16, ef_construction=64)
```

```python
# drop the old index
db.drop_index()

# Create an IVFFLAT index
# Note: You don't need to specify num_lists and num_records parameters as we set smart defaults.
db.create_index(index_type="ivfflat", num_lists=20, num_records=1000)
```

一般来说，我们建议使用默认的 timescale vector 索引或 HNSW 索引。

```python
# drop the old index
db.drop_index()
# Create a new timescale vector index
db.create_index()
```

## 4. 自查询检索器与 Timescale Vector

Timescale Vector 还支持自查询检索器功能，使其能够进行自我查询。给定一个包含查询语句和过滤器（单个或复合）的自然语言查询，检索器使用查询构造 LLM 链来编写 SQL 查询，然后将其应用于 Timescale Vector 向量存储中的底层 PostgreSQL 数据库。

有关自查询的更多信息，请 [查看文档](/docs/how_to/self_query)。

为了说明 Timescale Vector 的自查询，我们将使用第 3 部分中的相同 gitlog 数据集。

```python
COLLECTION_NAME = "timescale_commits"
vectorstore = TimescaleVector(
    embedding_function=OpenAIEmbeddings(),
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
)
```

接下来，我们将创建自查询检索器。为此，我们需要提前提供一些关于文档支持的元数据字段的信息以及文档内容的简短描述。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import OpenAI

# 向 LLM 提供有关元数据字段的信息
metadata_field_info = [
    AttributeInfo(
        name="id",
        description="从提交日期生成的 UUID v1",
        type="uuid",
    ),
    AttributeInfo(
        name="date",
        description="以 timestamptz 格式表示的提交日期",
        type="timestamptz",
    ),
    AttributeInfo(
        name="author_name",
        description="提交作者的姓名",
        type="string",
    ),
    AttributeInfo(
        name="author_email",
        description="提交作者的电子邮件地址",
        type="string",
    ),
]
document_content_description = "包含提交哈希、作者、提交日期、变更摘要和变更详情的 git 日志提交摘要"

# 从 LLM 实例化自查询检索器
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm,
    vectorstore,
    document_content_description,
    metadata_field_info,
    enable_limit=True,
    verbose=True,
)
```

现在让我们在 gitlog 数据集上测试自查询检索器。

运行以下查询，并注意如何在自然语言中指定查询、带过滤器的查询和带复合过滤器（带有 AND、OR 的过滤器），自查询检索器将把该查询转换为 SQL，并在 Timescale Vector PostgreSQL 向量存储上执行搜索。

这展示了自查询检索器的强大功能。您可以使用它对向量存储执行复杂搜索，而无需您或您的用户直接编写任何 SQL！

```python
# 此示例指定了一个相关查询
retriever.invoke("对连续聚合做了哪些改进？")
```
```output
/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/libs/langchain/langchain/chains/llm.py:275: UserWarning: The predict_and_parse method is deprecated, instead pass an output parser directly to LLMChain.
  warnings.warn(
``````output
query='improvements to continuous aggregates' filter=None limit=None
```






```python
# 此示例指定了一个过滤器
retriever.invoke("Sven Klemm 添加了哪些提交？")
```
```output
query=' ' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='author_name', value='Sven Klemm') limit=None
```


```python
# 此示例指定了一个查询和过滤器
retriever.invoke("Sven Klemm 添加了哪些关于 timescaledb_functions 的提交？")
```
```output
query='timescaledb_functions' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='author_name', value='Sven Klemm') limit=None
```



```python
# 此示例指定了一个基于时间的过滤器
retriever.invoke("在 2023 年 7 月添加了哪些提交？")
```
```output
query=' ' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='date', value='2023-07-01T00:00:00Z'), Comparison(comparator=<Comparator.LTE: 'lte'>, attribute='date', value='2023-07-31T23:59:59Z')]) limit=None
```




```python
# 此示例指定了一个查询和 LIMIT 值
retriever.invoke("关于分层连续聚合的两个提交是什么？")
```
```output
query='hierarchical continuous aggregates' filter=None limit=2
```

## 5. 使用现有的 TimescaleVector 向量存储

在上述示例中，我们从一组文档创建了一个向量存储。然而，我们通常希望向现有的向量存储中插入数据并查询数据。让我们看看如何初始化、向现有文档集合中添加文档以及查询 TimescaleVector 向量存储中的现有文档集合。

要使用现有的 Timescale Vector 存储，我们需要知道要查询的表的名称 (`COLLECTION_NAME`) 和云 PostgreSQL 数据库的 URL (`SERVICE_URL`)。

```python
# Initialize the existing
COLLECTION_NAME = "timescale_commits"
embeddings = OpenAIEmbeddings()
vectorstore = TimescaleVector(
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
    embedding_function=embeddings,
)
```

要将新数据加载到表中，我们使用 `add_document()` 函数。该函数接受文档列表和元数据列表。元数据必须包含每个文档的唯一 id。

如果您希望文档与当前日期和时间相关联，则无需创建 id 列表。每个文档将自动生成一个 uuid。

如果您希望文档与过去的日期和时间相关联，可以使用 `timecale-vector` Python 库中的 `uuid_from_time` 函数创建 id 列表，如上面第 2 节所示。该函数接受一个 datetime 对象，并返回一个带有日期和时间编码的 uuid。

```python
# Add documents to a collection in TimescaleVector
ids = vectorstore.add_documents([Document(page_content="foo")])
ids
```

```output
['a34f2b8a-53d7-11ee-8cc3-de1e4b2a0118']
```

```python
# Query the vectorstore for similar documents
docs_with_score = vectorstore.similarity_search_with_score("foo")
```

```python
docs_with_score[0]
```

```output
(Document(page_content='foo', metadata={}), 5.006789860928507e-06)
```

```python
docs_with_score[1]
```

```output
(Document(page_content='{"commit": " 00b566dfe478c11134bcf1e7bcf38943e7fafe8f", "author": "Fabr\\u00edzio de Royes Mello<fabriziomello@gmail.com>", "date": "Mon Mar 6 15:51:03 2023 -0300", "change summary": "Remove unused functions", "change details": "We don\'t use `ts_catalog_delete[_only]` functions anywhere and instead we rely on `ts_catalog_delete_tid[_only]` functions so removing it from our code base. "}', metadata={'id': 'd7f5c580-bc4f-11ed-9712-ffa0126a201a', 'date': '2023-03-6 15:51:03+-500', 'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/docs/docs/modules/ts_git_log.json', 'seq_num': 285, 'author_name': 'Fabrízio de Royes Mello', 'commit_hash': ' 00b566dfe478c11134bcf1e7bcf38943e7fafe8f', 'author_email': 'fabriziomello@gmail.com'}),
 0.23607668446580354)
```

### 删除数据

您可以通过 uuid 或者通过元数据的过滤器来删除数据。

```python
ids = vectorstore.add_documents([Document(page_content="Bar")])

vectorstore.delete(ids)
```

```output
True
```

通过元数据删除尤其有用，如果您想定期更新从特定来源、特定日期或其他元数据属性抓取的信息。

```python
vectorstore.add_documents(
    [Document(page_content="Hello World", metadata={"source": "www.example.com/hello"})]
)
vectorstore.add_documents(
    [Document(page_content="Adios", metadata={"source": "www.example.com/adios"})]
)

vectorstore.delete_by_metadata({"source": "www.example.com/adios"})

vectorstore.add_documents(
    [
        Document(
            page_content="Adios, but newer!",
            metadata={"source": "www.example.com/adios"},
        )
    ]
)
```

```output
['c6367004-53d7-11ee-8cc3-de1e4b2a0118']
```

### 覆盖向量存储

如果您有一个现有的集合，可以通过执行 `from_documents` 并将 `pre_delete_collection` 设置为 True 来覆盖它。

```python
db = TimescaleVector.from_documents(
    documents=docs,
    embedding=embeddings,
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
    pre_delete_collection=True,
)
```

```python
docs_with_score = db.similarity_search_with_score("foo")
```

```python
docs_with_score[0]
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/yellowbrick.ipynb
---

# Yellowbrick

[Yellowbrick](https://yellowbrick.com/yellowbrick-data-warehouse/) 是一个弹性、具备大规模并行处理 (MPP) 的 SQL 数据库，能够在云端和本地运行，利用 Kubernetes 实现扩展性、弹性和云端可移植性。Yellowbrick 的设计旨在解决最大和最复杂的业务关键数据仓库用例。Yellowbrick 提供的高效扩展性使其能够作为高性能和可扩展的向量数据库，使用 SQL 存储和搜索向量。

## 使用 Yellowbrick 作为 ChatGpt 的向量存储

本教程演示如何创建一个简单的聊天机器人，基于 ChatGpt，并使用 Yellowbrick 作为向量存储来支持检索增强生成（RAG）。您需要以下内容：

1. 一个 [Yellowbrick 沙盒](https://cloudlabs.yellowbrick.com/) 的账户
2. 来自 [OpenAI](https://platform.openai.com/) 的 API 密钥

本教程分为五个部分。首先，我们将使用 langchain 创建一个基线聊天机器人，以便在没有向量存储的情况下与 ChatGpt 互动。第二，我们将在 Yellowbrick 中创建一个表示向量存储的嵌入表。第三，我们将加载一系列文档（Yellowbrick 手册的管理章节）。第四，我们将创建这些文档的向量表示并存储在 Yellowbrick 表中。最后，我们将向改进后的聊天框发送相同的查询，以查看结果。

```python
# Install all needed libraries
%pip install --upgrade --quiet  langchain
%pip install --upgrade --quiet  langchain-openai langchain-community
%pip install --upgrade --quiet  psycopg2-binary
%pip install --upgrade --quiet  tiktoken
```

## 设置：输入用于连接 Yellowbrick 和 OpenAI API 的信息

我们的聊天机器人通过 langchain 库与 ChatGpt 集成，因此您需要首先从 OpenAI 获取 API 密钥：

要获取 OpenAI 的 API 密钥：
1. 注册 https://platform.openai.com/
2. 添加支付方式 - 您不太可能超过免费配额
3. 创建一个 API 密钥

您还需要在注册 Yellowbrick Sandbox 账户时收到的欢迎邮件中找到您的用户名、密码和数据库名。

以下内容应修改以包含您的 Yellowbrick 数据库和 OpenAPI 密钥的信息


```python
# Modify these values to match your Yellowbrick Sandbox and OpenAI API Key
YBUSER = "[SANDBOX USER]"
YBPASSWORD = "[SANDBOX PASSWORD]"
YBDATABASE = "[SANDBOX_DATABASE]"
YBHOST = "trialsandbox.sandbox.aws.yellowbrickcloud.com"

OPENAI_API_KEY = "[OPENAI API KEY]"
```


```python
# Import libraries and setup keys / login info
import os
import pathlib
import re
import sys
import urllib.parse as urlparse
from getpass import getpass

import psycopg2
from IPython.display import Markdown, display
from langchain.chains import LLMChain, RetrievalQAWithSourcesChain
from langchain_community.vectorstores import Yellowbrick
from langchain_core.documents import Document
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

# Establish connection parameters to Yellowbrick.  If you've signed up for Sandbox, fill in the information from your welcome mail here:
yellowbrick_connection_string = (
    f"postgres://{urlparse.quote(YBUSER)}:{YBPASSWORD}@{YBHOST}:5432/{YBDATABASE}"
)

YB_DOC_DATABASE = "sample_data"
YB_DOC_TABLE = "yellowbrick_documentation"
embedding_table = "my_embeddings"

# API Key for OpenAI.  Signup at https://platform.openai.com
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY

from langchain_core.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
    SystemMessagePromptTemplate,
)
```

## 第1部分：创建一个不使用向量存储的基线聊天机器人，基于ChatGpt

我们将使用langchain来查询ChatGPT。由于没有向量存储，ChatGPT将没有上下文来回答问题。

```python
# Set up the chat model and specific prompt
system_template = """If you don't know the answer, Make up your best guess."""
messages = [
    SystemMessagePromptTemplate.from_template(system_template),
    HumanMessagePromptTemplate.from_template("{question}"),
]
prompt = ChatPromptTemplate.from_messages(messages)

chain_type_kwargs = {"prompt": prompt}
llm = ChatOpenAI(
    model_name="gpt-3.5-turbo",  # Modify model_name if you have access to GPT-4
    temperature=0,
    max_tokens=256,
)

chain = LLMChain(
    llm=llm,
    prompt=prompt,
    verbose=False,
)


def print_result_simple(query):
    result = chain(query)
    output_text = f"""### Question:
  {query}
  ### Answer: 
  {result['text']}
    """
    display(Markdown(output_text))


# Use the chain to query
print_result_simple("How many databases can be in a Yellowbrick Instance?")

print_result_simple("What's an easy way to add users in bulk to Yellowbrick?")
```

## 第2部分：连接到Yellowbrick并创建嵌入表

要将文档嵌入加载到Yellowbrick中，您应该创建自己的表来存储它们。请注意，表所在的Yellowbrick数据库必须是UTF-8编码的。

在UTF-8数据库中创建一个具有以下模式的表，提供您选择的表名：

```python
# Establish a connection to the Yellowbrick database
try:
    conn = psycopg2.connect(yellowbrick_connection_string)
except psycopg2.Error as e:
    print(f"Error connecting to the database: {e}")
    exit(1)

# Create a cursor object using the connection
cursor = conn.cursor()

# Define the SQL statement to create a table
create_table_query = f"""
CREATE TABLE IF NOT EXISTS {embedding_table} (
    doc_id uuid NOT NULL,
    embedding_id smallint NOT NULL,
    embedding double precision NOT NULL
)
DISTRIBUTE ON (doc_id);
truncate table {embedding_table};
"""

# Execute the SQL query to create a table
try:
    cursor.execute(create_table_query)
    print(f"Table '{embedding_table}' created successfully!")
except psycopg2.Error as e:
    print(f"Error creating table: {e}")
    conn.rollback()

# Commit changes and close the cursor and connection
conn.commit()
cursor.close()
conn.close()
```

## 第3部分：从现有的Yellowbrick表中提取要索引的文档
从现有的Yellowbrick表中提取文档路径和内容。我们将在下一步中使用这些文档来创建嵌入。



```python
yellowbrick_doc_connection_string = (
    f"postgres://{urlparse.quote(YBUSER)}:{YBPASSWORD}@{YBHOST}:5432/{YB_DOC_DATABASE}"
)

print(yellowbrick_doc_connection_string)

# Establish a connection to the Yellowbrick database
conn = psycopg2.connect(yellowbrick_doc_connection_string)

# Create a cursor object
cursor = conn.cursor()

# Query to select all documents from the table
query = f"SELECT path, document FROM {YB_DOC_TABLE}"

# Execute the query
cursor.execute(query)

# Fetch all documents
yellowbrick_documents = cursor.fetchall()

print(f"Extracted {len(yellowbrick_documents)} documents successfully!")

# Close the cursor and connection
cursor.close()
conn.close()
```

## 第4部分：用文档加载Yellowbrick向量存储
处理文档，将其拆分为可消化的块，创建嵌入并插入Yellowbrick表中。这大约需要5分钟。



```python
# Split documents into chunks for conversion to embeddings
DOCUMENT_BASE_URL = "https://docs.yellowbrick.com/6.7.1/"  # Actual URL


separator = "\n## "  # This separator assumes Markdown docs from the repo uses ### as logical main header most of the time
chunk_size_limit = 2000
max_chunk_overlap = 200

documents = [
    Document(
        page_content=document[1],
        metadata={"source": DOCUMENT_BASE_URL + document[0].replace(".md", ".html")},
    )
    for document in yellowbrick_documents
]

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=chunk_size_limit,
    chunk_overlap=max_chunk_overlap,
    separators=[separator, "\nn", "\n", ",", " ", ""],
)
split_docs = text_splitter.split_documents(documents)

docs_text = [doc.page_content for doc in split_docs]

embeddings = OpenAIEmbeddings()
vector_store = Yellowbrick.from_documents(
    documents=split_docs,
    embedding=embeddings,
    connection_string=yellowbrick_connection_string,
    table=embedding_table,
)

print(f"Created vector store with {len(documents)} documents")
```

## 第5部分：创建一个使用Yellowbrick作为向量存储的聊天机器人

接下来，我们将Yellowbrick添加为向量存储。该向量存储已填充了表示Yellowbrick产品文档管理章节的嵌入。

我们将发送与上述相同的查询，以查看改进的响应。



```python
system_template = """Use the following pieces of context to answer the users question.
Take note of the sources and include them in the answer in the format: "SOURCES: source1 source2", use "SOURCES" in capital letters regardless of the number of sources.
If you don't know the answer, just say that "I don't know", don't try to make up an answer.
----------------
{summaries}"""
messages = [
    SystemMessagePromptTemplate.from_template(system_template),
    HumanMessagePromptTemplate.from_template("{question}"),
]
prompt = ChatPromptTemplate.from_messages(messages)

vector_store = Yellowbrick(
    OpenAIEmbeddings(),
    yellowbrick_connection_string,
    embedding_table,  # Change the table name to reflect your embeddings
)

chain_type_kwargs = {"prompt": prompt}
llm = ChatOpenAI(
    model_name="gpt-3.5-turbo",  # Modify model_name if you have access to GPT-4
    temperature=0,
    max_tokens=256,
)
chain = RetrievalQAWithSourcesChain.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vector_store.as_retriever(search_kwargs={"k": 5}),
    return_source_documents=True,
    chain_type_kwargs=chain_type_kwargs,
)


def print_result_sources(query):
    result = chain(query)
    output_text = f"""### Question: 
  {query}
  ### Answer: 
  {result['answer']}
  ### Sources: 
  {result['sources']}
  ### All relevant sources:
  {', '.join(list(set([doc.metadata['source'] for doc in result['source_documents']])))}
    """
    display(Markdown(output_text))


# Use the chain to query

print_result_sources("How many databases can be in a Yellowbrick Instance?")

print_result_sources("Whats an easy way to add users in bulk to Yellowbrick?")
```

## 第6部分：引入索引以提高性能

Yellowbrick还支持使用局部敏感哈希（Locality-Sensitive Hashing）方法进行索引。这是一种近似最近邻搜索技术，允许在准确性降低的情况下权衡相似性搜索时间。索引引入了两个可调参数：

- 超平面的数量，作为参数提供给`create_lsh_index(num_hyperplanes)`。文档越多，需要的超平面就越多。LSH是一种降维形式。原始嵌入被转换为低维向量，其中组件的数量与超平面的数量相同。
- 汉明距离，一个表示搜索广度的整数。较小的汉明距离会导致更快的检索，但准确性较低。

以下是如何在我们加载到Yellowbrick中的嵌入上创建索引。我们还将重新运行之前的聊天会话，但这次检索将使用索引。请注意，对于如此少量的文档，您不会看到在性能方面的索引好处。

```python
system_template = """Use the following pieces of context to answer the users question.
Take note of the sources and include them in the answer in the format: "SOURCES: source1 source2", use "SOURCES" in capital letters regardless of the number of sources.
If you don't know the answer, just say that "I don't know", don't try to make up an answer.
----------------
{summaries}"""
messages = [
    SystemMessagePromptTemplate.from_template(system_template),
    HumanMessagePromptTemplate.from_template("{question}"),
]
prompt = ChatPromptTemplate.from_messages(messages)

vector_store = Yellowbrick(
    OpenAIEmbeddings(),
    yellowbrick_connection_string,
    embedding_table,  # Change the table name to reflect your embeddings
)

lsh_params = Yellowbrick.IndexParams(
    Yellowbrick.IndexType.LSH, {"num_hyperplanes": 8, "hamming_distance": 2}
)
vector_store.create_index(lsh_params)

chain_type_kwargs = {"prompt": prompt}
llm = ChatOpenAI(
    model_name="gpt-3.5-turbo",  # Modify model_name if you have access to GPT-4
    temperature=0,
    max_tokens=256,
)
chain = RetrievalQAWithSourcesChain.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vector_store.as_retriever(
        k=5, search_kwargs={"index_params": lsh_params}
    ),
    return_source_documents=True,
    chain_type_kwargs=chain_type_kwargs,
)


def print_result_sources(query):
    result = chain(query)
    output_text = f"""### Question: 
  {query}
  ### Answer: 
  {result['answer']}
  ### Sources: 
  {result['sources']}
  ### All relevant sources:
  {', '.join(list(set([doc.metadata['source'] for doc in result['source_documents']])))}
    """
    display(Markdown(output_text))


# Use the chain to query

print_result_sources("How many databases can be in a Yellowbrick Instance?")

print_result_sources("Whats an easy way to add users in bulk to Yellowbrick?")
```

## 下一步：

此代码可以修改以提出不同的问题。您还可以将自己的文档加载到向量存储中。langchain 模块非常灵活，可以解析多种文件（包括 HTML、PDF 等）。

您还可以修改此代码以使用 Huggingface 嵌入模型和 Meta 的 Llama 2 LLM，以实现完全私密的聊天体验。

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
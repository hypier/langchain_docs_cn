---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/supabase.ipynb
---

# Supabase (Postgres)

>[Supabase](https://supabase.com/docs) 是一个开源的 Firebase 替代品。`Supabase` 构建在 `PostgreSQL` 之上，提供强大的 SQL 查询能力，并与现有工具和框架实现简单接口。

>[PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL)，也称为 `Postgres`，是一个免费的开源关系数据库管理系统（RDBMS），强调可扩展性和 SQL 兼容性。

本笔记本展示了如何使用 `Supabase` 和 `pgvector` 作为您的 VectorStore。

您需要通过 `pip install -qU langchain-community` 安装 `langchain-community` 以使用此集成。

要运行此笔记本，请确保：
- `pgvector` 扩展已启用
- 您已安装 `supabase-py` 包
- 您在数据库中创建了 `match_documents` 函数
- 您在 `public` 模式下有一个类似于下面的 `documents` 表。

以下函数确定余弦相似度，但您可以根据需要进行调整。

```sql
-- Enable the pgvector extension to work with embedding vectors
create extension if not exists vector;

-- Create a table to store your documents
create table
  documents (
    id uuid primary key,
    content text, -- corresponds to Document.pageContent
    metadata jsonb, -- corresponds to Document.metadata
    embedding vector (1536) -- 1536 works for OpenAI embeddings, change if needed
  );

-- Create a function to search for documents
create function match_documents (
  query_embedding vector (1536),
  filter jsonb default '{}'
) returns table (
  id uuid,
  content text,
  metadata jsonb,
  similarity float
) language plpgsql as $$
#variable_conflict use_column
begin
  return query
  select
    id,
    content,
    metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  where metadata @> filter
  order by documents.embedding <=> query_embedding;
end;
$$;
```


```python
# with pip
%pip install --upgrade --quiet  supabase

# with conda
# !conda install -c conda-forge supabase
```

我们想使用 `OpenAIEmbeddings`，因此我们必须获取 OpenAI API 密钥。


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```


```python
os.environ["SUPABASE_URL"] = getpass.getpass("Supabase URL:")
```


```python
os.environ["SUPABASE_SERVICE_KEY"] = getpass.getpass("Supabase Service Key:")
```


```python
# If you're storing your Supabase and OpenAI API keys in a .env file, you can load them with dotenv
from dotenv import load_dotenv

load_dotenv()
```

首先，我们将创建一个 Supabase 客户端并实例化 OpenAI 嵌入类。


```python
import os

from langchain_community.vectorstores import SupabaseVectorStore
from langchain_openai import OpenAIEmbeddings
from supabase.client import Client, create_client

supabase_url = os.environ.get("SUPABASE_URL")
supabase_key = os.environ.get("SUPABASE_SERVICE_KEY")
supabase: Client = create_client(supabase_url, supabase_key)

embeddings = OpenAIEmbeddings()
```

接下来，我们将加载和解析一些数据以供我们的向量存储使用（如果您已经在数据库中存储了带有嵌入的文档，可以跳过此步骤）。


```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

将上述文档插入数据库。每个文档的嵌入将自动生成。您可以根据文档的数量调整 chunk_size。默认值为 500，但降低它可能是必要的。


```python
vector_store = SupabaseVectorStore.from_documents(
    docs,
    embeddings,
    client=supabase,
    table_name="documents",
    query_name="match_documents",
    chunk_size=500,
)
```

或者，如果您已经在数据库中有带有嵌入的文档，可以直接实例化一个新的 `SupabaseVectorStore`：


```python
vector_store = SupabaseVectorStore(
    embedding=embeddings,
    client=supabase,
    table_name="documents",
    query_name="match_documents",
)
```

最后，通过执行相似性搜索来测试它：


```python
query = "What did the president say about Ketanji Brown Jackson"
matched_docs = vector_store.similarity_search(query)
```


```python
print(matched_docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

## 相似性搜索与评分

返回的距离分数是余弦距离。因此，分数越低越好。

```python
matched_docs = vector_store.similarity_search_with_relevance_scores(query)
```

```python
matched_docs[0]
```

```output
(Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../../state_of_the_union.txt'}),
 0.802509746274066)
```

## Retriever 选项

本节介绍如何将 SupabaseVectorStore 用作检索器的不同选项。

### 最大边际相关性搜索

除了在检索器对象中使用相似性搜索外，您还可以使用 `mmr`。



```python
retriever = vector_store.as_retriever(search_type="mmr")
```


```python
matched_docs = retriever.invoke(query)
```


```python
for i, d in enumerate(matched_docs):
    print(f"\n## Document {i}\n")
    print(d.page_content)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
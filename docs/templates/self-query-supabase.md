# self-query-supabase

此模板允许对 Supabase 进行自然语言结构化查询。

[Supabase](https://supabase.com/docs) 是一个开源的 Firebase 替代品，构建在 [PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) 之上。

它使用 [pgvector](https://github.com/pgvector/pgvector) 在您的表中存储嵌入。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

要获取您的 `OPENAI_API_KEY`，请前往您的 OpenAI 账户中的 [API 密钥](https://platform.openai.com/account/api-keys) 并创建一个新的密钥。

要找到您的 `SUPABASE_URL` 和 `SUPABASE_SERVICE_KEY`，请访问您 Supabase 项目的 [API 设置](https://supabase.com/dashboard/project/_/settings/api)。

- `SUPABASE_URL` 对应于项目 URL
- `SUPABASE_SERVICE_KEY` 对应于 `service_role` API 密钥


```shell
export SUPABASE_URL=
export SUPABASE_SERVICE_KEY=
export OPENAI_API_KEY=
```

## 设置 Supabase 数据库

如果您还没有设置 Supabase 数据库，请按照以下步骤操作。

1. 前往 https://database.new 来配置您的 Supabase 数据库。
2. 在控制台中，跳转到 [SQL 编辑器](https://supabase.com/dashboard/project/_/sql/new)，运行以下脚本以启用 `pgvector` 并将您的数据库设置为向量存储：

   ```sql
   -- Enable the pgvector extension to work with embedding vectors
   create extension if not exists vector;

   -- Create a table to store your documents
   create table
     documents (
       id uuid primary key,
       content text, -- corresponds to Document.pageContent
       metadata jsonb, -- corresponds to Document.metadata
       embedding vector (1536) -- 1536 works for OpenAI embeddings, change as needed
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

## 使用方法

要使用此软件包，请先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

创建一个新的 LangChain 项目，并将此软件包作为唯一的软件包安装：

```shell
langchain app new my-app --package self-query-supabase
```

要将其添加到现有项目中，请运行：

```shell
langchain app add self-query-supabase
```

将以下代码添加到您的 `server.py` 文件中：
```python
from self_query_supabase.chain import chain as self_query_supabase_chain

add_routes(app, self_query_supabase_chain, path="/self-query-supabase")
```

（可选）如果您可以访问 LangSmith，请配置它以帮助跟踪、监控和调试 LangChain 应用程序。如果您无法访问，请跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接启动一个 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行于 
[http://localhost:8000](http://localhost:8000)

您可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
访问游乐场 [http://127.0.0.1:8000/self-query-supabase/playground](http://127.0.0.1:8000/self-query-supabase/playground)

通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/self-query-supabase")
```

TODO: 设置 Supabase 数据库和安装软件包的说明。
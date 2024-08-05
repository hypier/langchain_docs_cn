# rag_lantern

此模板使用 Lantern 执行 RAG。

[Lantern](https://lantern.dev) 是一个基于 [PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) 的开源向量数据库。它使您能够在数据库中进行向量搜索和嵌入生成。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

要获取您的 `OPENAI_API_KEY`，请登录您的 OpenAI 账户并导航至 [API 密钥](https://platform.openai.com/account/api-keys) 以创建新的密钥。

要找到您的 `LANTERN_URL` 和 `LANTERN_SERVICE_KEY`，请前往您的 Lantern 项目的 [API 设置](https://lantern.dev/dashboard/project/_/settings/api)。

- `LANTERN_URL` 对应项目 URL
- `LANTERN_SERVICE_KEY` 对应 `service_role` API 密钥


```shell
export LANTERN_URL=
export LANTERN_SERVICE_KEY=
export OPENAI_API_KEY=
```

## 设置 Lantern 数据库

如果您还没有设置 Lantern 数据库，请按照以下步骤操作。

1. 前往 [https://lantern.dev](https://lantern.dev) 创建您的 Lantern 数据库。
2. 在您喜欢的 SQL 客户端中，跳转到 SQL 编辑器并运行以下脚本以将您的数据库设置为向量存储：

   ```sql
   -- Create a table to store your documents
   create table
     documents (
       id uuid primary key,
       content text, -- corresponds to Document.pageContent
       metadata jsonb, -- corresponds to Document.metadata
       embedding REAL[1536] -- 1536 works for OpenAI embeddings, change as needed
     );

   -- Create a function to search for documents
   create function match_documents (
     query_embedding REAL[1536],
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

## 设置环境变量

由于我们使用 [`Lantern`](https://python.langchain.com/docs/integrations/vectorstores/lantern) 和 [`OpenAIEmbeddings`](https://python.langchain.com/docs/integrations/text_embedding/openai)，我们需要加载它们的 API 密钥。

## 使用方法

首先，安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一的包，可以执行：

```shell
langchain app new my-app --package rag-lantern
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-lantern
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_lantern.chain import chain as rag_lantern_chain

add_routes(app, rag_lantern_chain, path="/rag-lantern")
```

（可选）现在让我们配置 LangSmith。 
LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。 
您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。 
如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接通过以下命令启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/rag-lantern/playground](http://127.0.0.1:8000/rag-lantern/playground) 访问游乐场。

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-lantern")
```
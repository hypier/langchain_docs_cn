# rag-momento-vector-index

此模板使用 Momento Vector Index (MVI) 和 OpenAI 执行 RAG。

> MVI：最具生产力、最易于使用的无服务器向量索引，适用于您的数据。要开始使用 MVI，只需注册一个帐户。无需处理基础设施、管理服务器或担心扩展问题。MVI 是一种服务，可以自动扩展以满足您的需求。可以与其他 Momento 服务结合使用，例如 Momento Cache，用于缓存提示和作为会话存储，或 Momento Topics 作为发布/订阅系统，将事件广播到您的应用程序。

要注册并访问 MVI，请访问 [Momento Console](https://console.gomomento.com/)。

## 环境设置

此模板使用 Momento Vector Index 作为向量存储，并要求设置 `MOMENTO_API_KEY` 和 `MOMENTO_INDEX_NAME`。

前往 [console](https://console.gomomento.com/) 获取 API 密钥。

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一的包，您可以执行：

```shell
langchain app new my-app --package rag-momento-vector-index
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-momento-vector-index
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_momento_vector_index import chain as rag_momento_vector_index_chain

add_routes(app, rag_momento_vector_index_chain, path="/rag-momento-vector-index")
```

（可选）现在让我们配置 LangSmith。
LangSmith 将帮助我们追踪、监控和调试 LangChain 应用程序。
您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。
如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/rag-momento-vector-index/playground](http://127.0.0.1:8000/rag-momento-vector-index/playground) 访问游乐场。

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-momento-vector-index")
```

## 索引数据

我们包含了一个示例模块来索引数据。该模块位于 `rag_momento_vector_index/ingest.py`。您会在 `chain.py` 中看到一行被注释掉的代码，调用了这个模块。请取消注释以使用。
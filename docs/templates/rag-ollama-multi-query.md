# rag-ollama-multi-query

该模板使用Ollama和OpenAI执行RAG，采用多查询检索器。

多查询检索器是查询转换的一个示例，根据用户输入的查询从不同的角度生成多个查询。

对于每个查询，它检索一组相关文档，并在所有查询中取唯一的并集以进行答案合成。

我们使用一个私有的本地LLM来执行查询生成的狭窄任务，以避免对更大LLM API的过度调用。

有关Ollama LLM执行查询扩展的示例跟踪，请参见[此处](https://smith.langchain.com/public/8017d04d-2045-4089-b47f-f2d66393a999/r)。

但我们使用OpenAI来处理更具挑战性的答案合成任务（完整跟踪示例[此处](https://smith.langchain.com/public/ec75793b-645b-498d-b855-e8d85e1f6738/r)）。

## 环境设置

要设置环境，您需要下载 Ollama。

请按照 [此处](https://python.langchain.com/docs/integrations/chat/ollama) 的说明进行操作。

您可以选择所需的 LLM 与 Ollama。

此模板使用 `zephyr`，可以通过 `ollama pull zephyr` 访问。

还有许多其他选项可在 [此处](https://ollama.ai/library) 找到。

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 使用方法

要使用此软件包，您应首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并安装此软件包，请执行：

```shell
langchain app new my-app --package rag-ollama-multi-query
```

要将此软件包添加到现有项目中，请运行：

```shell
langchain app add rag-ollama-multi-query
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_ollama_multi_query import chain as rag_ollama_multi_query_chain

add_routes(app, rag_ollama_multi_query_chain, path="/rag-ollama-multi-query")
```

（可选）现在，让我们配置 LangSmith。LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 [http://localhost:8000](http://localhost:8000)。

您可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
您可以在 [http://127.0.0.1:8000/rag-ollama-multi-query/playground](http://127.0.0.1:8000/rag-ollama-multi-query/playground) 访问游乐场。

要从代码中访问模板，请使用：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-ollama-multi-query")
```
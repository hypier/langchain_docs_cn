# rag-azure-search

此模板使用 [Azure AI Search](https://learn.microsoft.com/azure/search/search-what-is-azure-search) 作为向量存储，并结合 Azure OpenAI 聊天和嵌入模型对文档执行 RAG。

有关使用 Azure AI Search 进行 RAG 的更多详细信息，请参阅 [此笔记本](https://github.com/langchain-ai/langchain/blob/master/docs/docs/integrations/vectorstores/azuresearch.ipynb)。

## 环境设置

***先决条件:*** 现有的 [Azure AI Search](https://learn.microsoft.com/azure/search/search-what-is-azure-search) 和 [Azure OpenAI](https://learn.microsoft.com/azure/ai-services/openai/overview) 资源。

***环境变量:***

要运行此模板，您需要设置以下环境变量：

***必需:***

- AZURE_SEARCH_ENDPOINT - Azure AI Search 服务的端点。
- AZURE_SEARCH_KEY - Azure AI Search 服务的 API 密钥。
- AZURE_OPENAI_ENDPOINT - Azure OpenAI 服务的端点。
- AZURE_OPENAI_API_KEY - Azure OpenAI 服务的 API 密钥。
- AZURE_EMBEDDINGS_DEPLOYMENT - 用于嵌入的 Azure OpenAI 部署名称。
- AZURE_CHAT_DEPLOYMENT - 用于聊天的 Azure OpenAI 部署名称。

***可选:***

- AZURE_SEARCH_INDEX_NAME - 要使用的现有 Azure AI Search 索引名称。如果未提供，将创建名为 "rag-azure-search" 的索引。
- OPENAI_API_VERSION - 要使用的 Azure OpenAI API 版本。默认为 "2023-05-15"。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此包作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package rag-azure-search
```

如果您想将其添加到现有项目中，可以直接运行：

```shell
langchain app add rag-azure-search
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_azure_search import chain as rag_azure_search_chain

add_routes(app, rag_azure_search_chain, path="/rag-azure-search")
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

如果您在此目录中，则可以直接通过以下命令启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/rag-azure-search/playground](http://127.0.0.1:8000/rag-azure-search/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-azure-search")
```
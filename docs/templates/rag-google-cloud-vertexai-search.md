# rag-google-cloud-vertexai-search

此模板是一个应用程序，利用了 Google Vertex AI Search，这是一个基于机器学习的搜索服务，以及 PaLM 2 for Chat (chat-bison)。该应用程序使用检索链根据您的文档回答问题。

有关使用 Vertex AI Search 构建 RAG 应用程序的更多背景信息，请查看 [这里](https://cloud.google.com/generative-ai-app-builder/docs/enterprise-search-introduction)。

## 环境设置

在使用此模板之前，请确保您已通过 Vertex AI Search 进行身份验证。请参阅身份验证指南：[这里](https://cloud.google.com/generative-ai-app-builder/docs/authentication)。

您还需要创建：

- 一个搜索应用程序 [这里](https://cloud.google.com/generative-ai-app-builder/docs/create-engine-es)
- 一个数据存储 [这里](https://cloud.google.com/generative-ai-app-builder/docs/create-data-store-es)

适合测试该模板的数据集是 Alphabet 财报，您可以在 [这里](https://abc.xyz/investor/) 找到。数据也可以在 `gs://cloud-samples-data/gen-app-builder/search/alphabet-investor-pdfs` 获取。

设置以下环境变量：

* `GOOGLE_CLOUD_PROJECT_ID` - 您的 Google Cloud 项目 ID。
* `DATA_STORE_ID` - Vertex AI Search 中数据存储的 ID，这是一个 36 字符的字母数字值，可以在数据存储详细信息页面找到。
* `MODEL_TYPE` - Vertex AI Search 的模型类型。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此包作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package rag-google-cloud-vertexai-search
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-google-cloud-vertexai-search
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_google_cloud_vertexai_search.chain import chain as rag_google_cloud_vertexai_search_chain

add_routes(app, rag_google_cloud_vertexai_search_chain, path="/rag-google-cloud-vertexai-search")
```

（可选）现在让我们配置 LangSmith。
LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。
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

这将启动 FastAPI 应用程序，服务器在本地运行于
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以访问游乐场
在 [http://127.0.0.1:8000/rag-google-cloud-vertexai-search/playground](http://127.0.0.1:8000/rag-google-cloud-vertexai-search/playground)

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-google-cloud-vertexai-search")
```
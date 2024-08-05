# rag-google-cloud-sensitive-data-protection

此模板是一个利用 Google Vertex AI Search 的应用程序，这是一项基于机器学习的搜索服务，以及 PaLM 2 for Chat (chat-bison)。该应用程序使用检索链根据您的文档回答问题。

此模板是一个利用 Google 敏感数据保护的应用程序，这是一项用于检测和编辑文本中敏感数据的服务，以及 PaLM 2 for Chat (chat-bison)，尽管您可以使用任何模型。

有关使用敏感数据保护的更多背景信息，请查看 [here](https://cloud.google.com/dlp/docs/sensitive-data-protection-overview)。

## 环境设置

在使用此模板之前，请确保您在 Google Cloud 项目中启用了 [DLP API](https://console.cloud.google.com/marketplace/product/google/dlp.googleapis.com) 和 [Vertex AI API](https://console.cloud.google.com/marketplace/product/google/aiplatform.googleapis.com)。

有关与 Google Cloud 相关的一些常见环境故障排除步骤，请参见本 README 的底部。

设置以下环境变量：

* `GOOGLE_CLOUD_PROJECT_ID` - 您的 Google Cloud 项目 ID。
* `MODEL_TYPE` - Vertex AI Search 的模型类型（例如 `chat-bison`）

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package rag-google-cloud-sensitive-data-protection
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-google-cloud-sensitive-data-protection
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_google_cloud_sensitive_data_protection.chain import chain as rag_google_cloud_sensitive_data_protection_chain

add_routes(app, rag_google_cloud_sensitive_data_protection_chain, path="/rag-google-cloud-sensitive-data-protection")
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

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行于
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以访问 playground
在 [http://127.0.0.1:8000/rag-google-cloud-vertexai-search/playground](http://127.0.0.1:8000/rag-google-cloud-vertexai-search/playground)

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-google-cloud-sensitive-data-protection")
```

# 故障排除 Google Cloud

您可以使用其 CLI 设置您的 `gcloud` 凭据，命令为 `gcloud auth application-default login`

您可以使用以下命令设置您的 `gcloud` 项目：

```bash
gcloud config set project <your project>
gcloud auth application-default set-quota-project <your project>
export GOOGLE_CLOUD_PROJECT_ID=<your project>
```
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/databricks.ipynb
---

# Databricks

> [Databricks](https://www.databricks.com/) 湖仓平台将数据、分析和人工智能统一在一个平台上。

本笔记本提供了一个关于如何开始使用 Databricks [嵌入模型](/docs/concepts/#embedding-models) 的快速概述。有关所有 DatabricksEmbeddings 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/embeddings/langchain_community.embeddings.databricks.DatabricksEmbeddings.html)。

## 概述

`DatabricksEmbeddings` 类封装了托管在 [Databricks Model Serving](https://docs.databricks.com/en/machine-learning/model-serving/index.html) 上的嵌入模型端点。此示例笔记本展示了如何封装您的服务端点，并在您的 LangChain 应用程序中将其用作嵌入模型。

### 支持的方法

`DatabricksEmbeddings` 支持 `Embeddings` 类的所有方法，包括异步 API。

### 端点要求

服务端点 `DatabricksEmbeddings` 必须具有与 OpenAI 兼容的嵌入输入/输出格式（[参考](https://mlflow.org/docs/latest/llms/deployments/index.html#embeddings)）。只要输入格式兼容，`DatabricksEmbeddings` 可以用于托管在 [Databricks Model Serving](https://docs.databricks.com/en/machine-learning/model-serving/index.html) 上的任何端点类型：

1. 基础模型 - 精心策划的最先进基础模型列表，如 BAAI General Embedding (BGE)。这些端点可以在您的 Databricks 工作区中直接使用，无需任何设置。
2. 自定义模型 - 您还可以通过 MLflow 将自定义嵌入模型部署到服务端点，使用您选择的框架，如 LangChain、Pytorch、Transformers 等。
3. 外部模型 - Databricks 端点可以作为代理服务托管在 Databricks 之外的模型，例如像 OpenAI text-embedding-3 这样的专有模型服务。

## 设置

要访问 Databricks 模型，您需要创建一个 Databricks 账户，设置凭据（仅当您在 Databricks 工作区外时），并安装所需的包。

### 凭证（仅当您在 Databricks 外部时）

如果您在 Databricks 内部运行 LangChain 应用程序，可以跳过此步骤。

否则，您需要手动将 Databricks 工作区主机名和个人访问令牌分别设置为 `DATABRICKS_HOST` 和 `DATABRICKS_TOKEN` 环境变量。有关如何获取访问令牌的信息，请参见 [身份验证文档](https://docs.databricks.com/en/dev-tools/auth/index.html#databricks-personal-access-tokens)。

```python
import getpass
import os

os.environ["DATABRICKS_HOST"] = "https://your-workspace.cloud.databricks.com"
os.environ["DATABRICKS_TOKEN"] = getpass.getpass("Enter your Databricks access token: ")
```

### 安装

LangChain Databricks 集成位于 `langchain-community` 包中。此外，运行本笔记本中的代码需要 `mlflow >= 2.9 `。

```python
%pip install -qU langchain-community mlflow>=2.9.0
```

我们首先演示如何使用 `DatabricksEmbeddings` 查询作为基础模型端点托管的 BGE 模型。

对于其他类型的端点，在设置端点本身的方式上会有一些不同，但一旦端点准备就绪，查询方式就没有区别。

## 实例化


```python
from langchain_community.embeddings import DatabricksEmbeddings

embeddings = DatabricksEmbeddings(
    endpoint="databricks-bge-large-en",
    # Specify parameters for embedding queries and documents if needed
    # query_params={...},
    # document_params={...},
)
```

## 嵌入单个文本


```python
embeddings.embed_query("hello")[:3]
```
```output
[0.051055908203125, 0.007221221923828125, 0.003879547119140625]
```

## 嵌入文档


```python
documents = ["This is a dummy document.", "This is another dummy document."]
response = embeddings.embed_documents(documents)
print([e[:3] for e in response])  # Show first 3 elements of each embedding
```

## 包装其他类型的端点

上面的示例使用了作为基础模型 API 托管的嵌入模型。要了解如何使用其他端点类型，请参考 `ChatDatabricks` 的文档。虽然模型类型不同，但所需步骤是相同的。

* [自定义模型端点](https://python.langchain.com/v0.2/docs/integrations/chat/databricks/#wrapping-custom-model-endpoint)
* [外部模型](https://python.langchain.com/v0.2/docs/integrations/chat/databricks/#wrapping-external-models)

## API 参考

有关所有 ChatDatabricks 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/embeddings/langchain_community.embeddings.databricks.DatabricksEmbeddings.html

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)
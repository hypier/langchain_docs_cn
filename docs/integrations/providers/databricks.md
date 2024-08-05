Databricks
==========

> [Databricks](https://www.databricks.com/) 智能平台是全球首个由生成式 AI 驱动的数据智能平台。将 AI 融入您业务的每一个方面。

Databricks 以多种方式融入 LangChain 生态系统：

1. 🚀 **模型服务** - 通过高可用、低延迟的推理端点，访问最先进的 LLM，例如 DBRX、Llama3、Mixtral 或您微调的模型，访问 [Databricks 模型服务](https://www.databricks.com/product/model-serving)。LangChain 提供 LLM（`Databricks`）、聊天模型（`ChatDatabricks`）和嵌入（`DatabricksEmbeddings`）的实现，简化了您在 Databricks 模型服务上托管的模型与 LangChain 应用程序的集成。
2. 📃 **向量搜索** - [Databricks 向量搜索](https://www.databricks.com/product/machine-learning/vector-search) 是一个无服务器的向量数据库，与 Databricks 平台无缝集成。使用 `DatabricksVectorSearch`，您可以将高度可扩展和可靠的相似性搜索引擎集成到您的 LangChain 应用程序中。
3. 📊 **MLflow** - [MLflow](https://mlflow.org/) 是一个开源平台，用于管理整个 ML 生命周期，包括实验管理、评估、追踪、部署等。[MLflow 的 LangChain 集成](/docs/integrations/providers/mlflow_tracking) 简化了开发和操作现代复合 ML 系统的过程。
4. 🌐 **SQL 数据库** - [Databricks SQL](https://www.databricks.com/product/databricks-sql) 与 LangChain 中的 `SQLDatabase` 集成，允许您访问自动优化、性能卓越的数据仓库。
5. 💡 **开放模型** - Databricks 开源模型，例如 [DBRX](https://www.databricks.com/blog/introducing-dbrx-new-state-art-open-llm)，可通过 [Hugging Face Hub](https://huggingface.co/databricks/dbrx-instruct) 获取。这些模型可以直接与 LangChain 一起使用，利用其与 `transformers` 库的集成。

聊天模型
----------

`ChatDatabricks` 是一个聊天模型类，用于访问托管在 Databricks 上的聊天端点，包括最先进的模型，如 Llama3、Mixtral 和 DBRX，以及您自己的微调模型。

```
from langchain_community.chat_models.databricks import ChatDatabricks

chat_model = ChatDatabricks(endpoint="databricks-meta-llama-3-70b-instruct")
```

有关如何在您的 LangChain 应用程序中使用它的更多指导，请参见 [使用示例](/docs/integrations/chat/databricks)。

LLM
---

`Databricks` 是一个 LLM 类，用于访问托管在 Databricks 上的完成端点。

```
from langchain_community.llm.databricks import Databricks

llm = Databricks(endpoint="your-completion-endpoint")
```

有关如何在您的 LangChain 应用程序中使用它的更多指导，请参见 [使用示例](/docs/integrations/llms/databricks)。

嵌入
----------

`DatabricksEmbeddings` 是一个嵌入类，用于访问托管在 Databricks 上的文本嵌入端点，包括最先进的模型，如 BGE，以及您自己的微调模型。

```
from langchain_community.embeddings import DatabricksEmbeddings

embeddings = DatabricksEmbeddings(endpoint="databricks-bge-large-en")
```

有关如何在您的 LangChain 应用程序中使用它的更多指导，请参见 [使用示例](/docs/integrations/text_embedding/databricks)。

向量搜索
-------------

Databricks 向量搜索是一个无服务器的相似性搜索引擎，允许您在向量数据库中存储数据的向量表示，包括元数据。使用向量搜索，您可以从由 [Unity Catalog](https://www.databricks.com/product/unity-catalog) 管理的 [Delta](https://docs.databricks.com/en/introduction/delta-comparison.html) 表创建自动更新的向量搜索索引，并通过简单的 API 查询它们以返回最相似的向量。

```
from langchain_community.vectorstores import DatabricksVectorSearch

dvs = DatabricksVectorSearch(
    index, text_column="text", embedding=embeddings, columns=["source"]
)
docs = dvs.similarity_search("What is vector search?)
```

有关如何设置向量索引并将其与 LangChain 集成的更多信息，请参见 [使用示例](/docs/integrations/vectorstores/databricks_vector_search)。

MLflow 集成
------------------

在 LangChain 集成的背景下，MLflow 提供以下功能：

- **实验跟踪**：跟踪和存储来自您的 LangChain 实验的模型、工件和追踪。
- **依赖管理**：自动记录依赖库，确保开发、预发布和生产环境的一致性。
- **模型评估**：提供评估 LangChain 应用程序的原生能力。
- **追踪**：可视化追踪数据流通过您的 LangChain 应用程序。

有关使用 MLflow 与 LangChain 的全部功能，请参见 [MLflow LangChain 集成](/docs/integrations/providers/mlflow_tracking)，其中包含广泛的代码示例和指南。

SQLDatabase
-----------
您可以使用 LangChain 的 SQLDatabase 封装连接到 Databricks SQL。
```
from langchain.sql_database import SQLDatabase

db = SQLDatabase.from_databricks(catalog="samples", schema="nyctaxi")
```

有关如何将 Databricks SQL 与您的 LangChain Agent 连接作为强大的查询工具，请参见 [Databricks SQL Agent](https://docs.databricks.com/en/large-language-models/langchain.html#databricks-sql-agent)。

开放模型
-----------

要直接集成托管在 HuggingFace 上的 Databricks 开放模型，您可以使用 LangChain 的 [HuggingFace 集成](/docs/integrations/platforms/huggingface)。

```
from langchain_huggingface import HuggingFaceEndpoint

llm = HuggingFaceEndpoint(
    repo_id="databricks/dbrx-instruct",
    task="text-generation",
    max_new_tokens=512,
    do_sample=False,
    repetition_penalty=1.03,
)
llm.invoke("What is DBRX model?")
```
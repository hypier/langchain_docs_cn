---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/zilliz_cloud_pipeline.ipynb
---

# Zilliz Cloud Pipeline

> [Zilliz Cloud Pipelines](https://docs.zilliz.com/docs/pipelines) 将您的非结构化数据转换为可搜索的向量集合，串联嵌入、摄取、搜索和删除您的数据。
> 
> Zilliz Cloud Pipelines 可在 Zilliz Cloud 控制台和通过 RestFul API 使用。

本笔记本演示了如何准备 Zilliz Cloud Pipelines 并通过 LangChain Retriever 使用它们。

## 准备 Zilliz Cloud 管道

要为 LangChain Retriever 准备管道，您需要在 Zilliz Cloud 中创建和配置服务。

**1. 设置数据库**

- [注册 Zilliz Cloud](https://docs.zilliz.com/docs/register-with-zilliz-cloud)
- [创建集群](https://docs.zilliz.com/docs/create-cluster)

**2. 创建管道**

- [文档摄取、搜索、删除](https://docs.zilliz.com/docs/pipelines-doc-data)
- [文本摄取、搜索、删除](https://docs.zilliz.com/docs/pipelines-text-data)

## 使用 LangChain 检索器


```python
%pip install --upgrade --quiet langchain-milvus
```


```python
from langchain_milvus import ZillizCloudPipelineRetriever

retriever = ZillizCloudPipelineRetriever(
    pipeline_ids={
        "ingestion": "<YOUR_INGESTION_PIPELINE_ID>",  # 如果不需要添加文档，请跳过此行
        "search": "<YOUR_SEARCH_PIPELINE_ID>",  # 如果不需要获取相关文档，请跳过此行
        "deletion": "<YOUR_DELETION_PIPELINE_ID>",  # 如果不需要删除文档，请跳过此行
    },
    token="<YOUR_ZILLIZ_CLOUD_API_KEY>",
)
```

### 添加文档

要添加文档，您可以使用方法 `add_texts` 或 `add_doc_url`，该方法从文本列表或带有相应元数据的预签名/公共 URL 中插入文档到存储中。

- 如果使用 **文本摄取管道**，您可以使用方法 `add_texts`，该方法将一批文本及其相应的元数据插入 Zilliz Cloud 存储中。

    **参数：**
    - `texts`: 文本字符串列表。
    - `metadata`: 将作为摄取管道所需的保留字段插入的键值字典。默认为 None。



```python
# retriever.add_texts(
#     texts = ["example text 1e", "example text 2"],
#     metadata={"<FIELD_NAME>": "<FIELD_VALUE>"}  # skip this line if no preserved field is required by the ingestion pipeline
#     )
```

- 如果使用 **文档摄取管道**，您可以使用方法 `add_doc_url`，该方法将带有相应元数据的文档从 URL 插入到 Zilliz Cloud 存储中。

    **参数：**
    - `doc_url`: 文档 URL。
    - `metadata`: 将作为摄取管道所需的保留字段插入的键值字典。默认为 None。

以下示例适用于文档摄取管道，该管道需要将 milvus 版本作为元数据。我们将使用一个 [示例文档](https://publicdataset.zillizcloud.com/milvus_doc.md)，描述如何在 Milvus v2.3.x 中删除实体。 


```python
retriever.add_doc_url(
    doc_url="https://publicdataset.zillizcloud.com/milvus_doc.md",
    metadata={"version": "v2.3.x"},
)
```



```output
{'token_usage': 1247, 'doc_name': 'milvus_doc.md', 'num_chunks': 6}
```

### 获取相关文档

要查询检索器，可以使用方法 `get_relevant_documents`，该方法返回一个 LangChain Document 对象的列表。

**参数：**
- `query`: 要查找相关文档的字符串。
- `top_k`: 结果数量。默认为 10。
- `offset`: 在搜索结果中跳过的记录数。默认为 0。
- `output_fields`: 输出中呈现的额外字段。
- `filter`: 用于过滤搜索结果的 Milvus 表达式。默认为 ""。
- `run_manager`: 要使用的回调处理程序。

```python
retriever.get_relevant_documents(
    "Can users delete entities by complex boolean expressions?"
)
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/pathway.ipynb
---

# Pathway
> [Pathway](https://pathway.com/) 是一个开放的数据处理框架。它使您能够轻松开发与实时数据源和变化数据一起工作的 数据转换管道和机器学习应用程序。

本笔记本演示了如何使用实时的 `Pathway` 数据索引管道与 `Langchain`。您可以像查询常规向量存储一样从您的链中查询此管道的结果。然而，Pathway 在每次数据更改时更新索引，始终为您提供最新的答案。

在本笔记本中，我们将使用一个 [公共演示文档处理管道](https://pathway.com/solutions/ai-pipelines#try-it-out)，该管道：

1. 监控多个云数据源的数据变化。
2. 为数据构建向量索引。

要拥有自己的文档处理管道，请查看 [托管服务](https://pathway.com/solutions/ai-pipelines) 或 [自行构建](https://pathway.com/developers/user-guide/llm-xpack/vectorstore_pipeline/)。

我们将使用 `VectorStore` 客户端连接到索引，该客户端实现了 `similarity_search` 函数以检索匹配的文档。

本文件中使用的基本管道可以轻松构建存储在云位置的文件的简单向量索引。然而，Pathway 提供了构建实时数据管道和应用程序所需的一切，包括 SQL 类似的操作，如 groupby-缩减和不同数据源之间的连接、基于时间的数据分组和窗口处理，以及各种连接器。

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 以使用此集成。

## 查询数据管道

要实例化并配置客户端，您需要提供文档索引管道的 `url` 或 `host` 和 `port`。在下面的代码中，我们使用一个公开可用的 [demo pipeline](https://pathway.com/solutions/ai-pipelines#try-it-out)，其 REST API 可通过 `https://demo-document-indexing.pathway.stream` 访问。此演示从 [Google Drive](https://drive.google.com/drive/u/0/folders/1cULDv2OaViJBmOfG5WB0oWcgayNrGtVs) 和 [Sharepoint](https://navalgo.sharepoint.com/sites/ConnectorSandbox/Shared%20Documents/Forms/AllItems.aspx?id=%2Fsites%2FConnectorSandbox%2FShared%20Documents%2FIndexerSandbox&p=true&ga=1) 中摄取文档，并维护一个索引以检索文档。

```python
from langchain_community.vectorstores import PathwayVectorClient

client = PathwayVectorClient(url="https://demo-document-indexing.pathway.stream")
```

我们可以开始提出查询

```python
query = "What is Pathway?"
docs = client.similarity_search(query)
```

```python
print(docs[0].page_content)
```

**轮到您了！** [获取您的管道](https://pathway.com/solutions/ai-pipelines) 或上传 [新文档](https://chat-realtime-sharepoint-gdrive.demo.pathway.com/) 到演示管道并重试查询！

## 基于文件元数据的过滤

我们支持使用 [jmespath](https://jmespath.org/) 表达式进行文档过滤，例如：


```python
# take into account only sources modified later than unix timestamp
docs = client.similarity_search(query, metadata_filter="modified_at >= `1702672093`")

# take into account only sources modified later than unix timestamp
docs = client.similarity_search(query, metadata_filter="owner == `james`")

# take into account only sources with path containing 'repo_readme'
docs = client.similarity_search(query, metadata_filter="contains(path, 'repo_readme')")

# and of two conditions
docs = client.similarity_search(
    query, metadata_filter="owner == `james` && modified_at >= `1702672093`"
)

# or of two conditions
docs = client.similarity_search(
    query, metadata_filter="owner == `james` || modified_at >= `1702672093`"
)
```

## 获取索引文件的信息

 `PathwayVectorClient.get_vectorstore_statistics()` 提供了向量存储状态的基本统计信息，例如索引文件的数量和最后更新的时间戳。您可以在链中使用它来告诉用户您的知识库有多新鲜。


```python
client.get_vectorstore_statistics()
```

## 你自己的管道

### 在生产中运行
要拥有自己的 Pathway 数据索引管道，请查看 Pathway 提供的 [托管管道](https://pathway.com/solutions/ai-pipelines)。您还可以运行自己的 Pathway 管道 - 有关如何构建管道的信息，请参考 [Pathway 指南](https://pathway.com/developers/user-guide/llm-xpack/vectorstore_pipeline/)。

### 处理文档

向量化管道支持可插拔组件用于解析、拆分和嵌入文档。对于嵌入和拆分，您可以使用 [Langchain 组件](https://pathway.com/developers/user-guide/llm-xpack/vectorstore_pipeline/#langchain)，或者查看 Pathway 中可用的 [嵌入器](https://pathway.com/developers/api-docs/pathway-xpacks-llm/embedders) 和 [拆分器](https://pathway.com/developers/api-docs/pathway-xpacks-llm/splitters)。如果未提供解析器，则默认使用 `UTF-8` 解析器。您可以在 [这里](https://github.com/pathwaycom/pathway/blob/main/python/pathway/xpacks/llm/parser.py) 找到可用的解析器。

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
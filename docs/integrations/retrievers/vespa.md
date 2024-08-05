---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/vespa.ipynb
---

# Vespa

>[Vespa](https://vespa.ai/) 是一个功能齐全的搜索引擎和向量数据库。它支持向量搜索（ANN）、词汇搜索以及结构化数据中的搜索，所有这些都可以在同一个查询中进行。

本笔记本展示了如何将 `Vespa.ai` 作为 LangChain 的检索器使用。

为了创建一个检索器，我们使用 [pyvespa](https://pyvespa.readthedocs.io/en/latest/index.html) 创建一个与 `Vespa` 服务的连接。

```python
%pip install --upgrade --quiet  pyvespa
```

```python
from vespa.application import Vespa

vespa_app = Vespa(url="https://doc-search.vespa.oath.cloud")
```

这创建了与 `Vespa` 服务的连接，这里是 Vespa 文档搜索服务。使用 `pyvespa` 包，您还可以连接到一个
[Vespa Cloud 实例](https://pyvespa.readthedocs.io/en/latest/deploy-vespa-cloud.html)
或一个本地
[Docker 实例](https://pyvespa.readthedocs.io/en/latest/deploy-docker.html)。

连接到服务后，您可以设置检索器：

```python
from langchain_community.retrievers import VespaRetriever

vespa_query_body = {
    "yql": "select content from paragraph where userQuery()",
    "hits": 5,
    "ranking": "documentation",
    "locale": "en-us",
}
vespa_content_field = "content"
retriever = VespaRetriever(vespa_app, vespa_query_body, vespa_content_field)
```

这设置了一个 LangChain 检索器，从 Vespa 应用程序中获取文档。在这里，最多从 `paragraph` 文档类型的 `content` 字段中检索 5 个结果，使用 `doumentation` 作为排名方法。`userQuery()` 将被实际的查询替换，该查询由 LangChain 传递。

有关更多信息，请参考 [pyvespa 文档](https://pyvespa.readthedocs.io/en/latest/getting-started-pyvespa.html#Query)。

现在您可以返回结果并继续在 LangChain 中使用这些结果。

```python
retriever.invoke("what is vespa?")
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/stores/elasticsearch.ipynb
sidebar_label: Elasticsearch
---

# ElasticsearchEmbeddingsCache

这将帮助您开始使用 Elasticsearch [键值存储](/docs/concepts/#key-value-stores)。有关所有 `ElasticsearchEmbeddingsCache` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/cache/langchain_elasticsearch.cache.ElasticsearchEmbeddingsCache.html)。

## 概述

`ElasticsearchEmbeddingsCache` 是一个 `ByteStore` 实现，利用您的 Elasticsearch 实例高效地存储和检索嵌入。

### 集成细节

| 类别 | 包 | 本地 | JS 支持 | 包下载量 | 包最新 |
| :--- | :--- | :---: | :---: |  :---: | :---: |
| [ElasticsearchEmbeddingsCache](https://api.python.langchain.com/en/latest/cache/langchain_elasticsearch.cache.ElasticsearchEmbeddingsCache.html) | [langchain_elasticsearch](https://api.python.langchain.com/en/latest/elasticsearch_api_reference.html) | ✅ | ❌ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain_elasticsearch?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain_elasticsearch?style=flat-square&label=%20) |

## 设置

要创建一个 `ElasticsearchEmbeddingsCache` 字节存储，您需要一个 Elasticsearch 集群。您可以 [在本地设置一个](https://www.elastic.co/downloads/elasticsearch) 或创建一个 [Elastic 账户](https://www.elastic.co/elasticsearch)。

### 安装

LangChain `ElasticsearchEmbeddingsCache` 集成位于 `__package_name__` 包中：


```python
%pip install -qU langchain_elasticsearch
```

## 实例化

现在我们可以实例化我们的字节存储：


```python
from langchain_elasticsearch import ElasticsearchEmbeddingsCache

# Example config for a locally running Elasticsearch instance
kv_store = ElasticsearchEmbeddingsCache(
    es_url="https://localhost:9200",
    index_name="llm-chat-cache",
    metadata={"project": "my_chatgpt_project"},
    namespace="my_chatgpt_project",
    es_user="elastic",
    es_password="<GENERATED PASSWORD>",
    es_params={
        "ca_certs": "~/http_ca.crt",
    },
)
```

## 用法

您可以使用 `mset` 方法在键下设置数据，如下所示：


```python
kv_store.mset(
    [
        ["key1", b"value1"],
        ["key2", b"value2"],
    ]
)

kv_store.mget(
    [
        "key1",
        "key2",
    ]
)
```



```output
[b'value1', b'value2']
```


您可以使用 `mdelete` 方法删除数据：


```python
kv_store.mdelete(
    [
        "key1",
        "key2",
    ]
)

kv_store.mget(
    [
        "key1",
        "key2",
    ]
)
```



```output
[None, None]
```

## 用作嵌入缓存

与其他 `ByteStores` 一样，您可以使用 `ElasticsearchEmbeddingsCache` 实例进行 [文档摄取中的持久缓存](/docs/how_to/caching_embeddings/) 以支持 RAG。

然而，缓存的向量默认情况下是不可搜索的。开发者可以自定义 Elasticsearch 文档的构建，以添加索引向量字段。

这可以通过子类化并重写方法来实现：

```python
from typing import Any, Dict, List


class SearchableElasticsearchStore(ElasticsearchEmbeddingsCache):
    @property
    def mapping(self) -> Dict[str, Any]:
        mapping = super().mapping
        mapping["mappings"]["properties"]["vector"] = {
            "type": "dense_vector",
            "dims": 1536,
            "index": True,
            "similarity": "dot_product",
        }
        return mapping

    def build_document(self, llm_input: str, vector: List[float]) -> Dict[str, Any]:
        body = super().build_document(llm_input, vector)
        body["vector"] = vector
        return body
```

在重写映射和文档构建时，请仅进行附加修改，保持基础映射不变。

## API 参考

有关所有 `ElasticsearchEmbeddingsCache` 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/cache/langchain_elasticsearch.cache.ElasticsearchEmbeddingsCache.html

## 相关

- [键值存储概念指南](/docs/concepts/#key-value-stores)
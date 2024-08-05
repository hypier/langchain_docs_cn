---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/airbyte_gong.ipynb
sidebar_class_name: hidden
---

# Airbyte Gong（已弃用）

注意：此连接器特定的加载器已弃用。请改用 [`AirbyteLoader`](/docs/integrations/document_loaders/airbyte)。

>[Airbyte](https://github.com/airbytehq/airbyte) 是一个用于从 API、数据库和文件到数据仓库和数据湖的 ELT 管道的数据集成平台。它拥有最大的 ELT 连接器目录，支持数据仓库和数据库。

此加载器将 Gong 连接器作为文档加载器公开，允许您将各种 Gong 对象加载为文档。

## 安装

首先，您需要安装 `airbyte-source-gong` Python 包。

```python
%pip install --upgrade --quiet  airbyte-source-gong
```

## 示例

查看 [Airbyte 文档页面](https://docs.airbyte.com/integrations/sources/gong/) 以获取有关如何配置读取器的详细信息。
配置对象应遵循的 JSON 架构可以在 Github 上找到：[https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-gong/source_gong/spec.yaml](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-gong/source_gong/spec.yaml)。

一般结构如下所示：
```python
{
  "access_key": "<access key name>",
  "access_key_secret": "<access key secret>",
  "start_date": "<date from which to start retrieving records from in ISO format, e.g. 2020-10-20T00:00:00Z>",
}
```

默认情况下，所有字段作为元数据存储在文档中，文本设置为空字符串。通过转换读取器返回的文档来构建文档的文本。

```python
from langchain_community.document_loaders.airbyte import AirbyteGongLoader

config = {
    # your gong configuration
}

loader = AirbyteGongLoader(
    config=config, stream_name="calls"
)  # 检查上述链接的文档以获取所有流的列表
```

现在您可以以常规方式加载文档

```python
docs = loader.load()
```

由于 `load` 返回一个列表，因此它会阻塞，直到所有文档加载完成。为了更好地控制此过程，您还可以使用 `lazy_load` 方法，它返回一个迭代器：

```python
docs_iterator = loader.lazy_load()
```

请记住，默认情况下页面内容为空，元数据对象包含记录中的所有信息。要处理文档，请创建一个继承自基本加载器的类，并自行实现 `_handle_records` 方法：

```python
from langchain_core.documents import Document


def handle_record(record, id):
    return Document(page_content=record.data["title"], metadata=record.data)


loader = AirbyteGongLoader(
    config=config, record_handler=handle_record, stream_name="calls"
)
docs = loader.load()
```

## 增量加载

某些数据流允许增量加载，这意味着源会跟踪已同步的记录，并且不会再次加载它们。这对于数据量大且更新频繁的源非常有用。

为了利用这一点，存储加载器的 `last_state` 属性，并在再次创建加载器时传递它。这将确保仅加载新记录。

```python
last_state = loader.last_state  # store safely

incremental_loader = AirbyteGongLoader(
    config=config, stream_name="calls", state=last_state
)

new_docs = incremental_loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
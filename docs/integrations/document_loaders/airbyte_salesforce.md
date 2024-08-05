---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/airbyte_salesforce.ipynb
sidebar_class_name: hidden
---

# Airbyte Salesforce（已弃用）

注意：此连接器特定的加载器已弃用。请改用 [`AirbyteLoader`](/docs/integrations/document_loaders/airbyte)。

>[Airbyte](https://github.com/airbytehq/airbyte) 是一个用于将 API、数据库和文件的数据集成平台，支持 ELT 管道到数据仓库和数据湖。它拥有最大的 ELT 连接器目录，连接到数据仓库和数据库。

此加载器将 Salesforce 连接器作为文档加载器公开，允许您将各种 Salesforce 对象加载为文档。

## 安装

首先，您需要安装 `airbyte-source-salesforce` python 包。

```python
%pip install --upgrade --quiet  airbyte-source-salesforce
```

## 示例

查看 [Airbyte 文档页面](https://docs.airbyte.com/integrations/sources/salesforce/) 获取有关如何配置读取器的详细信息。
配置对象应遵循的 JSON 架构可以在 Github 上找到: [https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-salesforce/source_salesforce/spec.yaml](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-salesforce/source_salesforce/spec.yaml)。

一般结构如下：
```python
{
  "client_id": "<oauth client id>",
  "client_secret": "<oauth client secret>",
  "refresh_token": "<oauth refresh token>",
  "start_date": "<date from which to start retrieving records from in ISO format, e.g. 2020-10-20T00:00:00Z>",
  "is_sandbox": False, # 如果您使用的是沙箱环境，请设置为 True
  "streams_criteria": [ # 应可加载的 Salesforce 对象的过滤器数组
    {"criteria": "exacts", "value": "Account"}, # Salesforce 对象的确切名称
    {"criteria": "starts with", "value": "Asset"}, # 名称的前缀
    # 其他允许的标准: ends with, contains, starts not with, ends not with, not contains, not exacts
  ],
}
```

默认情况下，所有字段作为元数据存储在文档中，文本设置为空字符串。通过转换读取器返回的文档来构建文档的文本。


```python
from langchain_community.document_loaders.airbyte import AirbyteSalesforceLoader

config = {
    # 您的 Salesforce 配置
}

loader = AirbyteSalesforceLoader(
    config=config, stream_name="asset"
)  # 请查看上面链接的文档以获取所有流的列表
```

现在您可以按通常的方式加载文档


```python
docs = loader.load()
```

由于 `load` 返回一个列表，它将在所有文档加载完成之前阻塞。为了更好地控制此过程，您还可以使用 `lazy_load` 方法，它返回一个迭代器：


```python
docs_iterator = loader.lazy_load()
```

请记住，默认情况下页面内容为空，元数据对象包含记录的所有信息。要以不同的方式创建文档，在创建加载器时传入一个 record_handler 函数：


```python
from langchain_core.documents import Document


def handle_record(record, id):
    return Document(page_content=record.data["title"], metadata=record.data)


loader = AirbyteSalesforceLoader(
    config=config, record_handler=handle_record, stream_name="asset"
)
docs = loader.load()
```

## 增量加载

某些数据流允许增量加载，这意味着源会跟踪已同步的记录，并且不会再次加载它们。这对于数据量大且频繁更新的源非常有用。

为了利用这一点，请存储加载器的 `last_state` 属性，并在再次创建加载器时传递它。这将确保仅加载新记录。

```python
last_state = loader.last_state  # store safely

incremental_loader = AirbyteSalesforceLoader(
    config=config, stream_name="asset", state=last_state
)

new_docs = incremental_loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
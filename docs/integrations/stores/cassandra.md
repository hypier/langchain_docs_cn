---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/stores/cassandra.ipynb
sidebar_label: Cassandra
---

# CassandraByteStore

这将帮助您入门 Cassandra [键值存储](/docs/concepts/#key-value-stores)。有关所有 `CassandraByteStore` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/storage/langchain_community.storage.cassandra.CassandraByteStore.html)。

## 概述

[Cassandra](https://cassandra.apache.org/) 是一个 NoSQL、行导向、高度可扩展和高度可用的数据库。

### 集成细节

| 类 | 包 | 本地 | [JS 支持](https://js.langchain.com/v0.2/docs/integrations/stores/cassandra_storage) | 包下载量 | 包最新版本 |
| :--- | :--- | :---: | :---: |  :---: | :---: |
| [CassandraByteStore](https://api.python.langchain.com/en/latest/storage/langchain_community.storage.cassandra.CassandraByteStore.html) | [langchain_community](https://api.python.langchain.com/en/latest/community_api_reference.html) | ✅ | ✅ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain_community?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain_community?style=flat-square&label=%20) |

## 设置

`CassandraByteStore` 是 `ByteStore` 的一个实现，它将数据存储在您的 Cassandra 实例中。  
存储键必须是字符串，并将映射到 Cassandra 表的 `row_id` 列。  
存储的 `bytes` 值映射到 Cassandra 表的 `body_blob` 列。

### 安装

LangChain `CassandraByteStore` 集成位于 `langchain_community` 包中。您还需要根据您使用的初始化方法安装 `cassio` 包或 `cassandra-driver` 包作为对等依赖：

```python
%pip install -qU langchain_community
%pip install -qU cassandra-driver
%pip install -qU cassio
```

您还需要创建一个 `cassandra.cluster.Session` 对象，具体描述请参见 [Cassandra 驱动程序文档](https://docs.datastax.com/en/developer/python-driver/latest/api/cassandra/cluster/#module-cassandra.cluster)。具体细节可能有所不同（例如网络设置和身份验证），但这可能类似于：

## 实例化

您首先需要创建一个 `cassandra.cluster.Session` 对象，如 [Cassandra 驱动程序文档](https://docs.datastax.com/en/developer/python-driver/latest/api/cassandra/cluster/#module-cassandra.cluster) 中所述。具体细节会有所不同（例如网络设置和身份验证），但这可能类似于：

```python
from cassandra.cluster import Cluster

cluster = Cluster()
session = cluster.connect()
```

然后您可以创建您的存储！您还需要提供 Cassandra 实例中现有键空间的名称：

```python
from langchain_community.storage import CassandraByteStore

kv_store = CassandraByteStore(
    table="my_store",
    session=session,
    keyspace="<YOUR KEYSPACE>",
)
```

## 使用方法

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

## 使用 `cassio` 初始化

也可以使用 cassio 配置会话和键空间。


```python
import cassio

cassio.init(contact_points="127.0.0.1", keyspace="<YOUR KEYSPACE>")

store = CassandraByteStore(
    table="my_store",
)

store.mset([("k1", b"v1"), ("k2", b"v2")])
print(store.mget(["k1", "k2"]))
```

## API 参考

有关所有 `CassandraByteStore` 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/storage/langchain_community.storage.cassandra.CassandraByteStore.html

## 相关

- [键值存储概念指南](/docs/concepts/#key-value-stores)
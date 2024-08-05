---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/stores/in_memory.ipynb
sidebar_label: 内存中
---

# InMemoryByteStore

本指南将帮助您开始使用内存中的 [key-value stores](/docs/concepts/#key-value-stores)。有关所有 `InMemoryByteStore` 特性和配置的详细文档，请访问 [API reference](https://api.python.langchain.com/en/latest/stores/langchain_core.stores.InMemoryByteStore.html)。

## 概述

`InMemoryByteStore` 是 `ByteStore` 的一种非持久化实现，它将所有内容存储在 Python 字典中。它适用于演示和不需要超出 Python 进程生命周期的持久化的情况。

### 集成详情

| 类别 | 包 | 本地 | [JS 支持](https://js.langchain.com/v0.2/docs/integrations/stores/in_memory/) | 包下载量 | 包最新版本 |
| :--- | :--- | :---: | :---: |  :---: | :---: |
| [InMemoryByteStore](https://api.python.langchain.com/en/latest/stores/langchain_core.stores.InMemoryByteStore.html) | [langchain_core](https://api.python.langchain.com/en/latest/core_api_reference.html) | ✅ | ✅ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain_core?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain_core?style=flat-square&label=%20) |

### 安装

LangChain `InMemoryByteStore` 集成位于 `langchain_core` 包中：

```python
%pip install -qU langchain_core
```

## 实例化

现在您可以实例化您的字节存储：


```python
from langchain_core.stores import InMemoryByteStore

kv_store = InMemoryByteStore()
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

## API 参考

有关所有 `InMemoryByteStore` 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/stores/langchain_core.stores.InMemoryByteStore.html

## 相关

- [键值存储概念指南](/docs/concepts/#key-value-stores)
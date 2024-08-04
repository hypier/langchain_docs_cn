---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/stores/redis.ipynb
sidebar_label: Redis
---
# RedisStore

This will help you get started with Redis [key-value stores](/docs/concepts/#key-value-stores). For detailed documentation of all `RedisStore` features and configurations head to the [API reference](https://api.python.langchain.com/en/latest/storage/langchain_community.storage.redis.RedisStore.html).

## Overview

The `RedisStore` is an implementation of `ByteStore` that stores everything in your Redis instance.

### Integration details

| Class | Package | Local | [JS support](https://js.langchain.com/v0.2/docs/integrations/stores/ioredis_storage) | Package downloads | Package latest |
| :--- | :--- | :---: | :---: |  :---: | :---: |
| [RedisStore](https://api.python.langchain.com/en/latest/storage/langchain_community.storage.redis.RedisStore.html) | [langchain_community](https://api.python.langchain.com/en/latest/community_api_reference.html) | ✅ | ✅ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain_community?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain_community?style=flat-square&label=%20) |

## Setup

To create a Redis byte store, you'll need to set up a Redis instance. You can do this locally or via a provider - see our [Redis guide](/docs/integrations/providers/redis) for an overview of options.

### Installation

The LangChain `RedisStore` integration lives in the `langchain_community` package:


```python
%pip install -qU langchain_community redis
```

## Instantiation

Now we can instantiate our byte store:


```python
from langchain_community.storage import RedisStore

kv_store = RedisStore(redis_url="redis://localhost:6379")
```

## Usage

You can set data under keys like this using the `mset` method:


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


And you can delete data using the `mdelete` method:


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


## API reference

For detailed documentation of all `RedisStore` features and configurations, head to the API reference: https://api.python.langchain.com/en/latest/storage/langchain_community.storage.redis.RedisStore.html


## Related

- [Key-value store conceptual guide](/docs/concepts/#key-value-stores)

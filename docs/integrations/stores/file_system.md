---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/stores/file_system.ipynb
sidebar_label: Local Filesystem
---
# LocalFileStore

This will help you get started with local filesystem [key-value stores](/docs/concepts/#key-value-stores). For detailed documentation of all LocalFileStore features and configurations head to the [API reference](https://api.python.langchain.com/en/latest/storage/langchain.storage.file_system.LocalFileStore.html).

## Overview

The `LocalFileStore` is a persistent implementation of `ByteStore` that stores everything in a folder of your choosing. It's useful if you're using a single machine and are tolerant of files being added or deleted.

### Integration details

| Class | Package | Local | [JS support](https://js.langchain.com/v0.2/docs/integrations/stores/file_system) | Package downloads | Package latest |
| :--- | :--- | :---: | :---: |  :---: | :---: |
| [LocalFileStore](https://api.python.langchain.com/en/latest/storage/langchain.storage.file_system.LocalFileStore.html) | [langchain](https://api.python.langchain.com/en/latest/langchain_api_reference.html) | ✅ | ✅ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain?style=flat-square&label=%20) |

### Installation

The LangChain `LocalFileStore` integration lives in the `langchain` package:


```python
%pip install -qU langchain
```

## Instantiation

Now we can instantiate our byte store:


```python
from pathlib import Path

from langchain.storage import LocalFileStore

root_path = Path.cwd() / "data"  # can also be a path set by a string

kv_store = LocalFileStore(root_path)
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


You can see the created files in your `data` folder:


```python
!ls {root_path}
```
```output
key1 key2
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

For detailed documentation of all `LocalFileStore` features and configurations, head to the API reference: https://api.python.langchain.com/en/latest/storage/langchain.storage.file_system.LocalFileStore.html


## Related

- [Key-value store conceptual guide](/docs/concepts/#key-value-stores)

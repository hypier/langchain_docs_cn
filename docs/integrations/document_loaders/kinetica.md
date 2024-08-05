---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/kinetica.ipynb
---

# Kinetica

本笔记本介绍如何从 Kinetica 加载文档


```python
%pip install gpudb==7.2.0.9
```


```python
from langchain_community.document_loaders.kinetica_loader import KineticaLoader
```


```python
## 加载环境变量
import os

from dotenv import load_dotenv
from langchain_community.vectorstores import (
    KineticaSettings,
)

load_dotenv()
```


```python
# Kinetica 需要连接到数据库。
# 这是设置的方法。
HOST = os.getenv("KINETICA_HOST", "http://127.0.0.1:9191")
USERNAME = os.getenv("KINETICA_USERNAME", "")
PASSWORD = os.getenv("KINETICA_PASSWORD", "")


def create_config() -> KineticaSettings:
    return KineticaSettings(host=HOST, username=USERNAME, password=PASSWORD)
```


```python
from langchain_community.document_loaders.kinetica_loader import KineticaLoader

# 以下 `QUERY` 是一个示例，不会运行；这
# 需要替换为有效的 `QUERY`，以返回
# 数据，并且 `SCHEMA.TABLE` 组合必须在 Kinetica 中存在。

QUERY = "select text, survey_id from SCHEMA.TABLE limit 10"
kinetica_loader = KineticaLoader(
    QUERY,
    HOST,
    USERNAME,
    PASSWORD,
)
kinetica_documents = kinetica_loader.load()
print(kinetica_documents)
```


```python
from langchain_community.document_loaders.kinetica_loader import KineticaLoader

# 以下 `QUERY` 是一个示例，不会运行；这
# 需要替换为有效的 `QUERY`，以返回
# 数据，并且 `SCHEMA.TABLE` 组合必须在 Kinetica 中存在。

QUERY = "select text, survey_id as source from SCHEMA.TABLE limit 10"
kl = KineticaLoader(
    query=QUERY,
    host=HOST,
    username=USERNAME,
    password=PASSWORD,
    metadata_columns=["source"],
)
kinetica_documents = kl.load()
print(kinetica_documents)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
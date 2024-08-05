---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/fauna.ipynb
---

# Fauna

>[Fauna](https://fauna.com/) 是一个文档数据库。

查询 `Fauna` 文档


```python
%pip install --upgrade --quiet  fauna
```

## 查询数据示例


```python
from langchain_community.document_loaders.fauna import FaunaLoader

secret = "<enter-valid-fauna-secret>"
query = "Item.all()"  # Fauna 查询。假设集合名为 "Item"
field = "text"  # 包含页面内容的字段。假设字段名为 "text"

loader = FaunaLoader(query, field, secret)
docs = loader.lazy_load()

for value in docs:
    print(value)
```

### 带分页的查询
如果有更多数据，您将获得一个 `after` 值。您可以通过在查询中传递 `after` 字符串来获取光标之后的值。

要了解更多，请访问 [此链接](https://fqlx-beta--fauna-docs.netlify.app/fqlx/beta/reference/schema_entities/set/static-paginate)

```python
query = """
Item.paginate("hs+DzoPOg ... aY1hOohozrV7A")
Item.all()
"""
loader = FaunaLoader(query, field, secret)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
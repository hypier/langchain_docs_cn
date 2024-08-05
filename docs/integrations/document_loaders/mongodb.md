---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/mongodb.ipynb
---

# MongoDB

[MongoDB](https://www.mongodb.com/) 是一种 NoSQL 文档导向数据库，支持具有动态模式的类似 JSON 的文档。

## 概述

MongoDB 文档加载器从 MongoDB 数据库返回 Langchain 文档列表。

加载器需要以下参数：

*   MongoDB 连接字符串
*   MongoDB 数据库名称
*   MongoDB 集合名称
*   （可选）内容过滤字典
*   （可选）要包含在输出中的字段名称列表

输出格式如下：

- pageContent= Mongo Document
- metadata={'database': '[database_name]', 'collection': '[collection_name]'}

## 加载文档加载器


```python
# add this import for running in jupyter notebook
import nest_asyncio

nest_asyncio.apply()
```


```python
from langchain_community.document_loaders.mongodb import MongodbLoader
```


```python
loader = MongodbLoader(
    connection_string="mongodb://localhost:27017/",
    db_name="sample_restaurants",
    collection_name="restaurants",
    filter_criteria={"borough": "Bronx", "cuisine": "Bakery"},
    field_names=["name", "address"],
)
```


```python
docs = loader.load()

len(docs)
```



```output
71
```



```python
docs[0]
```



```output
Document(page_content="Morris Park Bake Shop {'building': '1007', 'coord': [-73.856077, 40.848447], 'street': 'Morris Park Ave', 'zipcode': '10462'}", metadata={'database': 'sample_restaurants', 'collection': 'restaurants'})
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
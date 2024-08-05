---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/dria_index.ipynb
---

# Dria

>[Dria](https://dria.co/) 是一个公共 RAG 模型的中心，供开发者贡献和利用共享的嵌入湖。此笔记本演示如何使用 `Dria API` 进行数据检索任务。

# 安装

确保您已安装 `dria` 包。您可以使用 pip 安装它：

```python
%pip install --upgrade --quiet dria
```

# 配置 API 密钥

设置您的 Dria API 密钥以便访问。


```python
import os

os.environ["DRIA_API_KEY"] = "DRIA_API_KEY"
```

# 初始化 Dria Retriever

创建 `DriaRetriever` 的实例。

```python
from langchain.retrievers import DriaRetriever

api_key = os.getenv("DRIA_API_KEY")
retriever = DriaRetriever(api_key=api_key)
```

# **创建知识库**

在 [Dria's Knowledge Hub](https://dria.co/knowledge) 上创建知识库


```python
contract_id = retriever.create_knowledge_base(
    name="France's AI Development",
    embedding=DriaRetriever.models.jina_embeddings_v2_base_en.value,
    category="Artificial Intelligence",
    description="Explore the growth and contributions of France in the field of Artificial Intelligence.",
)
```

# 添加数据

将数据加载到您的 Dria 知识库中。

```python
texts = [
    "The first text to add to Dria.",
    "Another piece of information to store.",
    "More data to include in the Dria knowledge base.",
]

ids = retriever.add_texts(texts)
print("Data added with IDs:", ids)
```

# 检索数据

使用检索器根据查询查找相关文档。


```python
query = "Find information about Dria."
result = retriever.invoke(query)
for doc in result:
    print(doc)
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/weaviate-hybrid.ipynb
---

# Weaviate 混合搜索

>[Weaviate](https://weaviate.io/developers/weaviate) 是一个开源向量数据库。

>[混合搜索](https://weaviate.io/blog/hybrid-search-explained) 是一种结合多种搜索算法的技术，以提高搜索结果的准确性和相关性。它利用基于关键字的搜索算法和向量搜索技术的最佳特性。

>`Weaviate 中的混合搜索` 使用稀疏和密集向量来表示搜索查询和文档的含义和上下文。

本笔记本展示了如何将 `Weaviate 混合搜索` 用作 LangChain 检索器。

设置检索器：

```python
%pip install --upgrade --quiet  weaviate-client
```

```python
import os

import weaviate

WEAVIATE_URL = os.getenv("WEAVIATE_URL")
auth_client_secret = (weaviate.AuthApiKey(api_key=os.getenv("WEAVIATE_API_KEY")),)
client = weaviate.Client(
    url=WEAVIATE_URL,
    additional_headers={
        "X-Openai-Api-Key": os.getenv("OPENAI_API_KEY"),
    },
)

# client.schema.delete_all()
```

```python
from langchain_community.retrievers import (
    WeaviateHybridSearchRetriever,
)
from langchain_core.documents import Document
```
```output

```

```python
retriever = WeaviateHybridSearchRetriever(
    client=client,
    index_name="LangChain",
    text_key="text",
    attributes=[],
    create_schema_if_missing=True,
)
```

添加一些数据：

```python
docs = [
    Document(
        metadata={
            "title": "拥抱未来：人工智能的揭示",
            "author": "Rebecca Simmons 博士",
        },
        page_content="对人工智能演变的全面分析，从其起源到未来前景。Simmons 博士涵盖了伦理考量、潜力和人工智能带来的威胁。",
    ),
    Document(
        metadata={
            "title": "共生：人类与人工智能的和谐",
            "author": "Jonathan K. Sterling 教授",
        },
        page_content="Sterling 教授探讨了人类与人工智能和谐共存的潜力。这本书讨论了人工智能如何以有益且不具破坏性的方式融入社会。",
    ),
    Document(
        metadata={"title": "人工智能：伦理困境", "author": "Rebecca Simmons 博士"},
        page_content="在她的第二本书中，Simmons 博士深入探讨了围绕人工智能开发和部署的伦理考量。这是对开发者、政策制定者和整个社会面临的困境的发人深省的审视。",
    ),
    Document(
        metadata={
            "title": "意识构建：寻找人工智能的知觉",
            "author": "Samuel Cortez 博士",
        },
        page_content="Cortez 博士带领读者探索人工智能意识这一有争议的话题。这本书提供了支持和反对真正的人工智能知觉可能性的有力论据。",
    ),
    Document(
        metadata={
            "title": "隐形的日常：日常生活中的隐秘人工智能",
            "author": "Jonathan K. Sterling 教授",
        },
        page_content="在他对《共生》的后续作品中，Sterling 教授审视了人工智能在我们日常生活中的微妙、不被注意的存在和影响。它揭示了人工智能如何在我们的日常生活中悄然融入，常常在我们没有明确意识到的情况下。",
    ),
]
```

```python
retriever.add_documents(docs)
```

```output
['3a27b0a5-8dbb-4fee-9eba-8b6bc2c252be',
 'eeb9fd9b-a3ac-4d60-a55b-a63a25d3b907',
 '7ebbdae7-1061-445f-a046-1989f2343d8f',
 'c2ab315b-3cab-467f-b23a-b26ed186318d',
 'b83765f2-e5d2-471f-8c02-c3350ade4c4f']
```

进行混合搜索：

```python
retriever.invoke("人工智能的伦理影响")
```

```output
[Document(page_content='在她的第二本书中，Simmons 博士深入探讨了围绕人工智能开发和部署的伦理考量。这是对开发者、政策制定者和整个社会面临的困境的发人深省的审视。', metadata={}),
 Document(page_content='对人工智能演变的全面分析，从其起源到未来前景。Simmons 博士涵盖了伦理考量、潜力和人工智能带来的威胁。', metadata={}),
 Document(page_content="在他对《共生》的后续作品中，Sterling 教授审视了人工智能在我们日常生活中的微妙、不被注意的存在和影响。它揭示了人工智能如何在我们的日常生活中悄然融入，常常在我们没有明确意识到的情况下。", metadata={}),
 Document(page_content='Sterling 教授探讨了人类与人工智能和谐共存的潜力。这本书讨论了人工智能如何以有益且不具破坏性的方式融入社会。', metadata={})]
```

使用 where 过滤器进行混合搜索：

```python
retriever.invoke(
    "人工智能在社会中的整合",
    where_filter={
        "path": ["author"],
        "operator": "Equal",
        "valueString": "Prof. Jonathan K. Sterling",
    },
)
```

```output
[Document(page_content='Sterling 教授探讨了人类与人工智能和谐共存的潜力。这本书讨论了人工智能如何以有益且不具破坏性的方式融入社会。', metadata={}),
 Document(page_content="在他对《共生》的后续作品中，Sterling 教授审视了人工智能在我们日常生活中的微妙、不被注意的存在和影响。它揭示了人工智能如何在我们的日常生活中悄然融入，常常在我们没有明确意识到的情况下。", metadata={})]
```

使用分数进行混合搜索：

```python
retriever.invoke(
    "人工智能在社会中的整合",
    score=True,
)
```

```output
[Document(page_content='Sterling 教授探讨了人类与人工智能和谐共存的潜力。这本书讨论了人工智能如何以有益且不具破坏性的方式融入社会。', metadata={'_additional': {'explainScore': '(bm25)\n(hybrid) Document eeb9fd9b-a3ac-4d60-a55b-a63a25d3b907 contributed 0.00819672131147541 to the score\n(hybrid) Document eeb9fd9b-a3ac-4d60-a55b-a63a25d3b907 contributed 0.00819672131147541 to the score', 'score': '0.016393442'}}),
 Document(page_content="在他对《共生》的后续作品中，Sterling 教授审视了人工智能在我们日常生活中的微妙、不被注意的存在和影响。它揭示了人工智能如何在我们的日常生活中悄然融入，常常在我们没有明确意识到的情况下。", metadata={'_additional': {'explainScore': '(bm25)\n(hybrid) Document b83765f2-e5d2-471f-8c02-c3350ade4c4f contributed 0.0078125 to the score\n(hybrid) Document b83765f2-e5d2-471f-8c02-c3350ade4c4f contributed 0.008064516129032258 to the score', 'score': '0.015877016'}}),
 Document(page_content='在她的第二本书中，Simmons 博士深入探讨了围绕人工智能开发和部署的伦理考量。这是对开发者、政策制定者和整个社会面临的困境的发人深省的审视。', metadata={'_additional': {'explainScore': '(bm25)\n(hybrid) Document 7ebbdae7-1061-445f-a046-1989f2343d8f contributed 0.008064516129032258 to the score\n(hybrid) Document 7ebbdae7-1061-445f-a046-1989f2343d8f contributed 0.0078125 to the score', 'score': '0.015877016'}}),
 Document(page_content='对人工智能演变的全面分析，从其起源到未来前景。Simmons 博士涵盖了伦理考量、潜力和人工智能带来的威胁。', metadata={'_additional': {'explainScore': '(vector) [-0.0071824766 -0.0006682752 0.001723625 -0.01897258 -0.0045127636 0.0024410256 -0.020503938 0.013768672 0.009520169 -0.037972264]...  \n(hybrid) Document 3a27b0a5-8dbb-4fee-9eba-8b6bc2c252be contributed 0.007936507936507936 to the score', 'score': '0.007936508'}})]
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)
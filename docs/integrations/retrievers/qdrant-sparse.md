---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/qdrant-sparse.ipynb
---

# Qdrant 稀疏向量

>[Qdrant](https://qdrant.tech/) 是一个开源的高性能向量搜索引擎/数据库。

>`QdrantSparseVectorRetriever` 使用在 `Qdrant` [v1.7.0](https://qdrant.tech/articles/qdrant-1.7.x/) 中引入的 [稀疏向量](https://qdrant.tech/articles/sparse-vectors/) 进行文档检索。

安装 'qdrant_client' 包：

```python
%pip install --upgrade --quiet  qdrant_client
```

```python
from qdrant_client import QdrantClient, models

client = QdrantClient(location=":memory:")
collection_name = "sparse_collection"
vector_name = "sparse_vector"

client.create_collection(
    collection_name,
    vectors_config={},
    sparse_vectors_config={
        vector_name: models.SparseVectorParams(
            index=models.SparseIndexParams(
                on_disk=False,
            )
        )
    },
)
```

```output
True
```

```python
from langchain_community.retrievers import (
    QdrantSparseVectorRetriever,
)
from langchain_core.documents import Document
```

创建一个演示编码器函数：

```python
import random


def demo_encoder(_: str) -> tuple[list[int], list[float]]:
    return (
        sorted(random.sample(range(100), 100)),
        [random.uniform(0.1, 1.0) for _ in range(100)],
    )


# 使用演示编码器创建检索器
retriever = QdrantSparseVectorRetriever(
    client=client,
    collection_name=collection_name,
    sparse_vector_name=vector_name,
    sparse_encoder=demo_encoder,
)
```

添加一些文档：

```python
docs = [
    Document(
        metadata={
            "title": "超越视野：人工智能编年史",
            "author": "卡桑德拉·米切尔博士",
        },
        page_content="对人工智能迷人旅程的深入探索，由米切尔博士叙述。这一引人入胜的叙述跨越了历史根源、当前进展和对人工智能的推测未来，提供了一个紧凑的叙事，交织了技术、伦理和社会影响。",
    ),
    Document(
        metadata={
            "title": "协同枢纽：人与机器的融合",
            "author": "本杰明·S·安德森教授",
        },
        page_content="安德森教授在《协同枢纽》中深入探讨了人机协作的协同可能性。该书阐述了一个愿景，即人类与人工智能无缝融合，创造出新的生产力、创造力和共享智能的维度。",
    ),
    Document(
        metadata={
            "title": "人工智能困境：导航未知",
            "author": "埃琳娜·罗德里格斯博士",
        },
        page_content="罗德里格斯博士在《人工智能困境》中撰写了一部引人入胜的叙述，探讨了人工智能进步带来的伦理困境的未知领域。这本书充当了一种指南，引导读者穿越开发者、政策制定者和社会在人工智能发展过程中面临的复杂道德决策的复杂地形。",
    ),
    Document(
        metadata={
            "title": "有意识的线索：编织人工智能意识",
            "author": "亚历山大·J·本内特教授",
        },
        page_content="在《有意识的线索》中，本内特教授揭示了人工智能意识之谜，呈现出一幅论证的挂毯，审视机器意识的本质。这本书引发了对追求真正人工智能意识的伦理和哲学维度的思考。",
    ),
    Document(
        metadata={
            "title": "无声的炼金术：看不见的人工智能缓解措施",
            "author": "艾米莉·福斯特博士",
        },
        page_content="在她之前工作的基础上，福斯特博士揭示了《无声的炼金术》，对人工智能在我们日常生活中隐秘存在的深刻考察。这部启发性的作品揭示了人工智能以微妙而深远的方式无形地塑造我们的日常，强调了在技术驱动的世界中提高意识的必要性。",
    ),
]
```

执行检索：

```python
retriever.add_documents(docs)
```

```output
['1a3e0d292e6444d39451d0588ce746dc',
 '19b180dd31e749359d49967e5d5dcab7',
 '8de69e56086f47748e32c9e379e6865b',
 'f528fac385954e46b89cf8607bf0ee5a',
 'c1a6249d005d4abd9192b1d0b829cebe']
```

```python
retriever.invoke(
    "人工智能的生活与伦理困境",
)
```

```output
[Document(page_content="在《有意识的线索》中，本内特教授揭示了人工智能意识之谜，呈现出一幅论证的挂毯，审视机器意识的本质。这本书引发了对追求真正人工智能意识的伦理和哲学维度的思考。", metadata={'title': '有意识的线索：编织人工智能意识', 'author': '亚历山大·J·本内特教授'}),
 Document(page_content="罗德里格斯博士在《人工智能困境》中撰写了一部引人入胜的叙述，探讨了人工智能进步带来的伦理困境的未知领域。这本书充当了一种指南，引导读者穿越开发者、政策制定者和社会在人工智能发展过程中面临的复杂道德决策的复杂地形。", metadata={'title': '人工智能困境：导航未知', 'author': '埃琳娜·罗德里格斯博士'}),
 Document(page_content="安德森教授在《协同枢纽》中深入探讨了人机协作的协同可能性。该书阐述了一个愿景，即人类与人工智能无缝融合，创造出新的生产力、创造力和共享智能的维度。", metadata={'title': '协同枢纽：人与机器的融合', 'author': '本杰明·S·安德森教授'}),
 Document(page_content='对人工智能迷人旅程的深入探索，由米切尔博士叙述。这一引人入胜的叙述跨越了历史根源、当前进展和对人工智能的推测未来，提供了一个紧凑的叙事，交织了技术、伦理和社会影响。', metadata={'title': '超越视野：人工智能编年史', 'author': '卡桑德拉·米切尔博士'})]
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)
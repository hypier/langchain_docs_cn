---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/thirdai_neuraldb.ipynb
---

# ThirdAI NeuralDB

>[NeuralDB](https://www.thirdai.com/neuraldb-enterprise/) 是由 [ThirdAI](https://www.thirdai.com/) 开发的友好于 CPU 且可精细调优的向量存储。

## 初始化

有两种初始化方法：
- 从头开始：基本模型
- 从检查点：加载之前保存的模型

在所有以下初始化方法中，如果设置了 `THIRDAI_KEY` 环境变量，则可以省略 `thirdai_key` 参数。

可以在 https://www.thirdai.com/try-bolt/ 获取 ThirdAI API 密钥。

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 才能使用此集成。

```python
from langchain_community.vectorstores import NeuralDBVectorStore

# From scratch
vectorstore = NeuralDBVectorStore.from_scratch(thirdai_key="your-thirdai-key")

# From checkpoint
vectorstore = NeuralDBVectorStore.from_checkpoint(
    # Path to a NeuralDB checkpoint. For example, if you call
    # vectorstore.save("/path/to/checkpoint.ndb") in one script, then you can
    # call NeuralDBVectorStore.from_checkpoint("/path/to/checkpoint.ndb") in
    # another script to load the saved model.
    checkpoint="/path/to/checkpoint.ndb",
    thirdai_key="your-thirdai-key",
)
```

## 插入文档源

```python
vectorstore.insert(
    # 如果您有PDF、DOCX或CSV文件，可以直接传递文档的路径
    sources=["/path/to/doc.pdf", "/path/to/doc.docx", "/path/to/doc.csv"],
    # 当为True时，这意味着NeuralDB中的基础模型将
    # 对插入的文件进行无监督预训练。默认为True。
    train=True,
    # 插入速度更快，但性能略有下降。默认为True。
    fast_mode=True,
)

from thirdai import neural_db as ndb

vectorstore.insert(
    # 如果您有其他格式的文件，或者希望配置文件的解析方式，
    # 那么您可以像这样传递NeuralDB文档对象。
    sources=[
        ndb.PDF(
            "/path/to/doc.pdf",
            version="v2",
            chunk_size=100,
            metadata={"published": 2022},
        ),
        ndb.Unstructured("/path/to/deck.pptx"),
    ]
)
```

## 相似性搜索

要查询向量存储，可以使用标准的 LangChain 向量存储方法 `similarity_search`，该方法返回一个 LangChain Document 对象的列表。每个文档对象代表来自索引文件的一段文本。例如，它可能包含来自某个索引 PDF 文件的段落。除了文本之外，文档的元数据字段还包含信息，例如文档的 ID、该文档的来源（来自哪个文件）以及文档的得分。

```python
# This returns a list of LangChain Document objects
documents = vectorstore.similarity_search("query", k=10)
```

## 微调

NeuralDBVectorStore 可以根据用户行为和特定领域知识进行微调。它可以通过两种方式进行微调：
1. 关联：向量存储将源短语与目标短语关联。当向量存储看到源短语时，它还会考虑与目标短语相关的结果。
2. 赞同：向量存储为特定查询提高文档的得分。这在您希望根据用户行为微调向量存储时非常有用。例如，如果用户搜索“汽车是如何制造的”，并且喜欢返回的文档 ID 为 52 的文档，那么我们可以为查询“汽车是如何制造的”对文档 ID 为 52 进行赞同。

```python
vectorstore.associate(source="source phrase", target="target phrase")
vectorstore.associate_batch(
    [
        ("source phrase 1", "target phrase 1"),
        ("source phrase 2", "target phrase 2"),
    ]
)

vectorstore.upvote(query="how is a car manufactured", document_id=52)
vectorstore.upvote_batch(
    [
        ("query 1", 52),
        ("query 2", 20),
    ]
)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
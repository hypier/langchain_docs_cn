---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/thirdai_neuraldb.ipynb
---

# **NeuralDB**
NeuralDB 是一个由 ThirdAI 开发的 CPU 友好且可精细调优的检索引擎。

### **初始化**
有两种初始化方法：
- 从头开始：基本模型
- 从检查点：加载之前保存的模型

对于以下所有初始化方法，如果设置了 `THIRDAI_KEY` 环境变量，则可以省略 `thirdai_key` 参数。

可以在 https://www.thirdai.com/try-bolt/ 获取 ThirdAI API 密钥。

```python
from langchain.retrievers import NeuralDBRetriever

# From scratch
retriever = NeuralDBRetriever.from_scratch(thirdai_key="your-thirdai-key")

# From checkpoint
retriever = NeuralDBRetriever.from_checkpoint(
    # Path to a NeuralDB checkpoint. For example, if you call
    # retriever.save("/path/to/checkpoint.ndb") in one script, then you can
    # call NeuralDBRetriever.from_checkpoint("/path/to/checkpoint.ndb") in
    # another script to load the saved model.
    checkpoint="/path/to/checkpoint.ndb",
    thirdai_key="your-thirdai-key",
)
```

### **插入文档来源**


```python
retriever.insert(
    # 如果您有 PDF、DOCX 或 CSV 文件，可以直接传递文档的路径
    sources=["/path/to/doc.pdf", "/path/to/doc.docx", "/path/to/doc.csv"],
    # 当为 True 时，这意味着 NeuralDB 中的基础模型将
    # 在插入的文件上进行无监督的预训练。默认为 True。
    train=True,
    # 插入速度更快，但性能略有下降。默认为 True。
    fast_mode=True,
)

from thirdai import neural_db as ndb

retriever.insert(
    # 如果您有其他格式的文件，或者希望配置文件的解析方式，
    # 那么您可以传入 NeuralDB 文档对象
    # 如下所示。
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

### **检索文档**
要查询检索器，您可以使用标准的 LangChain 检索器方法 `get_relevant_documents`，该方法返回一个 LangChain Document 对象的列表。每个文档对象代表来自索引文件的一段文本。例如，它可能包含来自某个索引 PDF 文件的段落。除了文本之外，文档的元数据字段还包含信息，例如文档的 ID、该文档的来源（来自哪个文件）以及文档的评分。

```python
# This returns a list of LangChain Document objects
documents = retriever.invoke("query", top_k=10)
```

### **微调**
NeuralDBRetriever 可以根据用户行为和特定领域知识进行微调。它可以通过两种方式进行微调：
1. 关联：检索器将源短语与目标短语关联。当检索器看到源短语时，它也会考虑与目标短语相关的结果。
2. 赞成：检索器为特定查询提高文档的得分。这在您希望根据用户行为微调检索器时非常有用。例如，如果用户搜索“汽车是如何制造的”，并喜欢返回的文档 ID 为 52，那么我们可以为查询“汽车是如何制造的”对文档 ID 为 52 进行赞成。

```python
retriever.associate(source="source phrase", target="target phrase")
retriever.associate_batch(
    [
        ("source phrase 1", "target phrase 1"),
        ("source phrase 2", "target phrase 2"),
    ]
)

retriever.upvote(query="how is a car manufactured", document_id=52)
retriever.upvote_batch(
    [
        ("query 1", 52),
        ("query 2", 20),
    ]
)
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)
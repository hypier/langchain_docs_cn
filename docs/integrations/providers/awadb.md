# AwaDB

>[AwaDB](https://github.com/awa-ai/awadb) 是一个为 LLM 应用程序的嵌入向量搜索和存储而设计的 AI 原生数据库。

## 安装与设置

```bash
pip install awadb
```

## 向量存储

```python
from langchain_community.vectorstores import AwaDB
```

查看[使用示例](/docs/integrations/vectorstores/awadb)。

## 嵌入模型

```python
from langchain_community.embeddings import AwaEmbeddings
```

查看 [使用示例](/docs/integrations/text_embedding/awadb)。
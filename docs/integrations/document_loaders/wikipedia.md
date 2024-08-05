---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/wikipedia.ipynb
---

# Wikipedia

>[Wikipedia](https://wikipedia.org/) 是一个多语言的免费在线百科全书，由志愿者社区（称为维基人）通过开放协作和使用名为 MediaWiki 的基于维基的编辑系统编写和维护。`Wikipedia` 是历史上最大、阅读量最多的参考作品。

该笔记本展示了如何将来自 `wikipedia.org` 的维基页面加载到我们下游使用的文档格式中。

## 安装

首先，您需要安装 `wikipedia` python 包。

```python
%pip install --upgrade --quiet  wikipedia
```

## 示例

`WikipediaLoader` 有以下参数：
- `query`: 用于在维基百科中查找文档的自由文本
- 可选的 `lang`: 默认值为 "en"。用于在维基百科的特定语言部分进行搜索
- 可选的 `load_max_docs`: 默认值为 100。用于限制下载文档的数量。下载所有 100 个文档需要时间，因此在实验中使用较小的数字。目前有一个硬性限制为 300。
- 可选的 `load_all_available_meta`: 默认值为 False。默认情况下，仅下载最重要的字段：`Published`（文档发布/最后更新的日期）、`title`、`Summary`。如果为 True，其他字段也会被下载。

```python
from langchain_community.document_loaders import WikipediaLoader
```

```python
docs = WikipediaLoader(query="HUNTER X HUNTER", load_max_docs=2).load()
len(docs)
```

```python
docs[0].metadata  # 文档的元信息
```

```python
docs[0].page_content[:400]  # 文档的内容
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
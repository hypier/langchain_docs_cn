---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/wikipedia.ipynb
---

# 维基百科

>[维基百科](https://wikipedia.org/) 是一个多语言的免费在线百科全书，由志愿者社区（称为维基人）通过开放协作和使用名为MediaWiki的维基编辑系统撰写和维护。`Wikipedia` 是历史上最大的、阅读量最高的参考书籍。

首先，您需要安装 `wikipedia` python 包。


```python
%pip install --upgrade --quiet  wikipedia
```


```python
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
```


```python
wikipedia = WikipediaQueryRun(api_wrapper=WikipediaAPIWrapper())
```


```python
wikipedia.run("HUNTER X HUNTER")
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)
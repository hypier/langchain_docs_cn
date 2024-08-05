---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/mojeek_search.ipynb
---

# Mojeek搜索

以下笔记将解释如何使用Mojeek搜索获取结果。请访问 [Mojeek网站](https://www.mojeek.com/services/search/web-search-api/) 获取API密钥。


```python
from langchain_community.tools import MojeekSearch
```


```python
api_key = "KEY"  # 从Mojeek网站获取
```


```python
search = MojeekSearch.config(api_key=api_key, search_kwargs={"t": 10})
```

在`search_kwargs`中，您可以添加在 [Mojeek文档](https://www.mojeek.com/support/api/search/request_parameters.html) 中找到的任何搜索参数。


```python
search.run("mojeek")
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)
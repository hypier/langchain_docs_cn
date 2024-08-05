---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/youtube.ipynb
---

# YouTube

>[YouTube Search](https://github.com/joetats/youtube_search) 包搜索 `YouTube` 视频，避免使用其受限的 API。
>
>它使用 `YouTube` 首页的表单并抓取结果页面。

本笔记本展示了如何使用工具搜索 YouTube。

改编自 [https://github.com/venuv/langchain_yt_tools](https://github.com/venuv/langchain_yt_tools)


```python
%pip install --upgrade --quiet  youtube_search
```


```python
from langchain_community.tools import YouTubeSearchTool
```


```python
tool = YouTubeSearchTool()
```


```python
tool.run("lex fridman")
```



```output
"['/watch?v=VcVfceTsD0A&pp=ygUMbGV4IGZyaWVkbWFu', '/watch?v=gPfriiHBBek&pp=ygUMbGV4IGZyaWVkbWFu']"
```


您还可以指定返回结果的数量


```python
tool.run("lex friedman,5")
```



```output
"['/watch?v=VcVfceTsD0A&pp=ygUMbGV4IGZyaWVkbWFu', '/watch?v=YVJ8gTnDC4Y&pp=ygUMbGV4IGZyaWVkbWFu', '/watch?v=Udh22kuLebg&pp=ygUMbGV4IGZyaWVkbWFu', '/watch?v=gPfriiHBBek&pp=ygUMbGV4IGZyaWVkbWFu', '/watch?v=L_Guz73e6fw&pp=ygUMbGV4IGZyaWVkbWFu']"
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)
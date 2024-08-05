---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/web_base.ipynb
---

# WebBaseLoader

这部分介绍了如何使用 `WebBaseLoader` 从 `HTML` 网页加载所有文本到我们可以后续使用的文档格式。有关加载网页的更多自定义逻辑，请查看一些子类示例，例如 `IMSDbLoader`、`AZLyricsLoader` 和 `CollegeConfidentialLoader`。

如果您不想担心网站爬虫、绕过 JS 阻止的网站和数据清理，可以考虑使用 `FireCrawlLoader` 或更快的选项 `SpiderLoader`。



```python
from langchain_community.document_loaders import WebBaseLoader
```


```python
loader = WebBaseLoader("https://www.espn.com/")
```

要绕过获取过程中的 SSL 验证错误，您可以设置 "verify" 选项：

loader.requests_kwargs = {'verify':False}


```python
data = loader.load()
```


```python
data
```



```output
```



```python
"""
# 使用这段代码测试新的自定义 BeautifulSoup 解析器

import requests
from bs4 import BeautifulSoup

html_doc = requests.get("{INSERT_NEW_URL_HERE}")
soup = BeautifulSoup(html_doc.text, 'html.parser')

# 将 BeautifulSoup 逻辑导出到 langchain_community.document_loaders.webpage.py
# 示例：transcript = soup.select_one("td[class='scrtext']").text
# BS4 文档可以在这里找到：https://www.crummy.com/software/BeautifulSoup/bs4/doc/

"""
```

## 加载多个网页

您还可以通过将一个网址列表传递给加载器来同时加载多个网页。这将返回一个与传入的网址顺序相同的文档列表。

```python
loader = WebBaseLoader(["https://www.espn.com/", "https://google.com"])
docs = loader.load()
docs
```

```output
 Document(page_content='GoogleSearch Images Maps Play YouTube News Gmail Drive More »Web History | Settings | Sign in\xa0Advanced searchAdvertisingBusiness SolutionsAbout Google© 2023 - Privacy - Terms   ', lookup_str='', metadata={'source': 'https://google.com'}, lookup_index=0)]
```

### 并发加载多个网址

通过并发抓取和解析多个网址，可以加快抓取过程。

并发请求有合理的限制，默认每秒 2 个。如果您不关心做一个好公民，或者您控制着您正在抓取的服务器并且不在乎负载，可以更改 `requests_per_second` 参数以增加最大并发请求。请注意，虽然这会加快抓取过程，但可能会导致服务器封锁您。请小心！

```python
%pip install --upgrade --quiet  nest_asyncio

# fixes a bug with asyncio and jupyter
import nest_asyncio

nest_asyncio.apply()
```
```output
Requirement already satisfied: nest_asyncio in /Users/harrisonchase/.pyenv/versions/3.9.1/envs/langchain/lib/python3.9/site-packages (1.5.6)
```

```python
loader = WebBaseLoader(["https://www.espn.com/", "https://google.com"])
loader.requests_per_second = 1
docs = loader.aload()
docs
```

```output
 Document(page_content='GoogleSearch Images Maps Play YouTube News Gmail Drive More »Web History | Settings | Sign in\xa0Advanced searchAdvertisingBusiness SolutionsAbout Google© 2023 - Privacy - Terms   ', lookup_str='', metadata={'source': 'https://google.com'}, lookup_index=0)]
```

## 加载 XML 文件或使用不同的 BeautifulSoup 解析器

您还可以查看 `SitemapLoader`，了解如何加载网站地图文件，这是使用此功能的一个示例。

```python
loader = WebBaseLoader(
    "https://www.govinfo.gov/content/pkg/CFR-2018-title10-vol3/xml/CFR-2018-title10-vol3-sec431-86.xml"
)
loader.default_parser = "xml"
docs = loader.load()
docs
```

## 使用代理

有时您可能需要使用代理来绕过 IP 阻止。您可以将代理的字典传递给加载器（以及底层的 `requests`）以使用它们。

```python
loader = WebBaseLoader(
    "https://www.walmart.com/search?q=parrots",
    proxies={
        "http": "http://{username}:{password}:@proxy.service.com:6666/",
        "https": "https://{username}:{password}:@proxy.service.com:6666/",
    },
)
docs = loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
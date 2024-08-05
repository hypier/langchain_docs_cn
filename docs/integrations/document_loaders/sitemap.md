---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/sitemap.ipynb
---

# 网站地图

扩展自 `WebBaseLoader`，`SitemapLoader` 从给定的 URL 加载网站地图，然后抓取并加载网站地图中的所有页面，返回每个页面作为文档。

抓取是并发进行的。并发请求有合理的限制，默认每秒 2 次。如果你不担心成为一个好公民，或者你控制被抓取的服务器，或者不在乎负载。请注意，虽然这会加快抓取过程，但可能会导致服务器封锁你。请小心！

```python
%pip install --upgrade --quiet  nest_asyncio
```

```python
# 修复 asyncio 和 jupyter 的一个 bug
import nest_asyncio

nest_asyncio.apply()
```

```python
from langchain_community.document_loaders.sitemap import SitemapLoader
```

```python
sitemap_loader = SitemapLoader(web_path="https://api.python.langchain.com/sitemap.xml")

docs = sitemap_loader.load()
```

你可以更改 `requests_per_second` 参数以增加最大并发请求，并使用 `requests_kwargs` 在发送请求时传递 kwargs。

```python
sitemap_loader.requests_per_second = 2
# 可选：避免 `[SSL: CERTIFICATE_VERIFY_FAILED]` 问题
sitemap_loader.requests_kwargs = {"verify": False}
```

```python
docs[0]
```

```output
Document(page_content='\n\n\n\n\n\n\n\n\n\nLangChain Python API Reference Documentation.\n\n\nYou will be automatically redirected to the new location of this page.\n\n', metadata={'source': 'https://api.python.langchain.com/en/stable/', 'loc': 'https://api.python.langchain.com/en/stable/', 'lastmod': '2024-02-09T01:10:49.422114+00:00', 'changefreq': 'weekly', 'priority': '1'})
```

## 过滤网站地图 URL

网站地图可能是庞大的文件，包含成千上万个 URL。通常，您并不需要每一个 URL。您可以通过将字符串列表或正则表达式模式传递给 `filter_urls` 参数来过滤 URL。只有与其中一个模式匹配的 URL 会被加载。

```python
loader = SitemapLoader(
    web_path="https://api.python.langchain.com/sitemap.xml",
    filter_urls=["https://api.python.langchain.com/en/latest"],
)
documents = loader.load()
```

```python
documents[0]
```

```output
Document(page_content='\n\n\n\n\n\n\n\n\n\nLangChain Python API Reference Documentation.\n\n\nYou will be automatically redirected to the new location of this page.\n\n', metadata={'source': 'https://api.python.langchain.com/en/latest/', 'loc': 'https://api.python.langchain.com/en/latest/', 'lastmod': '2024-02-12T05:26:10.971077+00:00', 'changefreq': 'daily', 'priority': '0.9'})
```

## 添加自定义抓取规则

`SitemapLoader` 使用 `beautifulsoup4` 进行抓取过程，并默认抓取页面上的每个元素。`SitemapLoader` 构造函数接受一个自定义抓取函数。此功能可以帮助您根据特定需求定制抓取过程；例如，您可能希望避免抓取标题或导航元素。

以下示例展示了如何开发和使用自定义函数以避免导航和标题元素。

导入 `beautifulsoup4` 库并定义自定义函数。

```python
pip install beautifulsoup4
```

```python
from bs4 import BeautifulSoup


def remove_nav_and_header_elements(content: BeautifulSoup) -> str:
    # Find all 'nav' and 'header' elements in the BeautifulSoup object
    nav_elements = content.find_all("nav")
    header_elements = content.find_all("header")

    # Remove each 'nav' and 'header' element from the BeautifulSoup object
    for element in nav_elements + header_elements:
        element.decompose()

    return str(content.get_text())
```

将您的自定义函数添加到 `SitemapLoader` 对象中。

```python
loader = SitemapLoader(
    "https://api.python.langchain.com/sitemap.xml",
    filter_urls=["https://api.python.langchain.com/en/latest/"],
    parsing_function=remove_nav_and_header_elements,
)
```

## 本地网站地图

网站地图加载器也可以用于加载本地文件。

```python
sitemap_loader = SitemapLoader(web_path="example_data/sitemap.xml", is_local=True)

docs = sitemap_loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
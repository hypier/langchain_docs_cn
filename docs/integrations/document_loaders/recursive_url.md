---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/recursive_url.ipynb
---

# 递归 URL

`RecursiveUrlLoader` 允许您递归抓取根 URL 的所有子链接，并将它们解析为文档。

## 设置

`RecursiveUrlLoader` 位于 `langchain-community` 包中。没有其他必需的包，不过如果安装了 `beautifulsoup4`，您将获得更丰富的默认文档元数据。

```python
%pip install -qU langchain-community beautifulsoup4
```

## 实例化

现在我们可以实例化我们的文档加载器对象并加载文档：


```python
from langchain_community.document_loaders import RecursiveUrlLoader

loader = RecursiveUrlLoader(
    "https://docs.python.org/3.9/",
    # max_depth=2,
    # use_async=False,
    # extractor=None,
    # metadata_extractor=None,
    # exclude_dirs=(),
    # timeout=10,
    # check_response_status=True,
    # continue_on_failure=True,
    # prevent_outside=True,
    # base_url=None,
    # ...
)
```

## 加载

使用 ``.load()`` 同步加载所有文档到内存中，每个文档对应一个访问过的 URL。从初始 URL 开始，我们递归遍历所有链接的 URL，直到指定的最大深度。

让我们通过一个基本示例来看看如何在 [Python 3.9 文档](https://docs.python.org/3.9/) 上使用 `RecursiveUrlLoader`。

```python
docs = loader.load()
docs[0].metadata
```
```output
/Users/bagatur/.pyenv/versions/3.9.1/lib/python3.9/html/parser.py:170: XMLParsedAsHTMLWarning: It looks like you're parsing an XML document using an HTML parser. If this really is an HTML document (maybe it's XHTML?), you can ignore or filter this warning. If it's XML, you should know that using an XML parser will be more reliable. To parse this document as XML, make sure you have the lxml package installed, and pass the keyword argument `features="xml"` into the BeautifulSoup constructor.
  k = self.parse_starttag(i)
```


```output
{'source': 'https://docs.python.org/3.9/',
 'content_type': 'text/html',
 'title': '3.9.19 文档',
 'language': None}
```


太好了！第一个文档看起来像是我们开始时的根页面。让我们查看下一个文档的元数据。


```python
docs[1].metadata
```



```output
{'source': 'https://docs.python.org/3.9/using/index.html',
 'content_type': 'text/html',
 'title': 'Python 安装和使用 — Python 3.9.19 文档',
 'language': None}
```


那个 URL 看起来像是我们根页面的子页面，这很好！让我们从元数据转到检查我们文档的内容。


```python
print(docs[0].page_content[:300])
```
```output

<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <meta charset="utf-8" /><title>3.9.19 文档</title><meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <link rel="stylesheet" href="_static/pydoctheme.css" type="text/css" />
    <link rel=
```
这确实看起来像是来自 URL https://docs.python.org/3.9/ 的 HTML，这正是我们所期待的。现在让我们看看一些可以对基本示例进行的变更，这在不同情况下可能会很有帮助。

## 添加提取器

默认情况下，加载器将每个链接的原始 HTML 设置为文档页面内容。要将此 HTML 解析为更适合人类/LLM 的格式，您可以传入自定义的 ``extractor`` 方法：

```python
import re

from bs4 import BeautifulSoup


def bs4_extractor(html: str) -> str:
    soup = BeautifulSoup(html, "lxml")
    return re.sub(r"\n\n+", "\n\n", soup.text).strip()


loader = RecursiveUrlLoader("https://docs.python.org/3.9/", extractor=bs4_extractor)
docs = loader.load()
print(docs[0].page_content[:200])
```
```output
/var/folders/td/vzm913rx77x21csd90g63_7c0000gn/T/ipykernel_10935/1083427287.py:6: XMLParsedAsHTMLWarning: It looks like you're parsing an XML document using an HTML parser. If this really is an HTML document (maybe it's XHTML?), you can ignore or filter this warning. If it's XML, you should know that using an XML parser will be more reliable. To parse this document as XML, make sure you have the lxml package installed, and pass the keyword argument `features="xml"` into the BeautifulSoup constructor.
  soup = BeautifulSoup(html, "lxml")
/Users/isaachershenson/.pyenv/versions/3.11.9/lib/python3.11/html/parser.py:170: XMLParsedAsHTMLWarning: It looks like you're parsing an XML document using an HTML parser. If this really is an HTML document (maybe it's XHTML?), you can ignore or filter this warning. If it's XML, you should know that using an XML parser will be more reliable. To parse this document as XML, make sure you have the lxml package installed, and pass the keyword argument `features="xml"` into the BeautifulSoup constructor.
  k = self.parse_starttag(i)
``````output
3.9.19 文档

下载
下载这些文档
按版本查看文档

Python 3.13（开发中）
Python 3.12（稳定版）
Python 3.11（安全修复）
Python 3.10（安全修复）
Python 3.9（安全
```
这看起来好得多！

您还可以传入 `metadata_extractor` 自定义从 HTTP 响应中提取文档元数据的方式。有关更多信息，请参见 [API 参考](https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.recursive_url_loader.RecursiveUrlLoader.html)。

## 懒加载

如果我们正在加载大量文档，并且我们的下游操作可以在所有加载的文档的子集上进行，我们可以一次懒加载一个文档，以最小化内存占用：

```python
page = []
for doc in loader.lazy_load():
    page.append(doc)
    if len(page) >= 10:
        # do some paged operation, e.g.
        # index.upsert(page)

        page = []
```
```output
/var/folders/4j/2rz3865x6qg07tx43146py8h0000gn/T/ipykernel_73962/2110507528.py:6: XMLParsedAsHTMLWarning: It looks like you're parsing an XML document using an HTML parser. If this really is an HTML document (maybe it's XHTML?), you can ignore or filter this warning. If it's XML, you should know that using an XML parser will be more reliable. To parse this document as XML, make sure you have the lxml package installed, and pass the keyword argument `features="xml"` into the BeautifulSoup constructor.
  soup = BeautifulSoup(html, "lxml")
```
在这个例子中，我们一次加载到内存中的文档数量从未超过10个。

## API 参考

这些示例展示了您可以修改默认的 `RecursiveUrlLoader` 的一些方式，但还有许多其他修改可以更好地适应您的用例。使用参数 `link_regex` 和 `exclude_dirs` 可以帮助您过滤掉不需要的 URL，`aload()` 和 `alazy_load()` 可以用于异步加载，等等。

有关配置和调用 ``RecursiveUrlLoader`` 的详细信息，请参阅 API 参考： https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.recursive_url_loader.RecursiveUrlLoader.html.

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
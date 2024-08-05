---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/url.ipynb
---

# URL

本示例介绍如何从 `URLs` 列表加载 `HTML` 文档到可以用于下游处理的 `Document` 格式。

## 非结构化 URL 加载器

对于下面的示例，请安装 `unstructured` 库，并查看 [此指南](/docs/integrations/providers/unstructured/) 以获取有关本地设置 Unstructured 的更多说明，包括设置所需的系统依赖项：

```python
%pip install --upgrade --quiet unstructured
```

```python
from langchain_community.document_loaders import UnstructuredURLLoader

urls = [
    "https://www.understandingwar.org/backgrounder/russian-offensive-campaign-assessment-february-8-2023",
    "https://www.understandingwar.org/backgrounder/russian-offensive-campaign-assessment-february-9-2023",
]
```

将 ssl_verify=False 与 headers=headers 一起传入，以绕过 ssl_verification 错误。

```python
loader = UnstructuredURLLoader(urls=urls)

data = loader.load()

data[0]
```

```output
```

## Selenium URL 加载器

这部分介绍如何使用 `SeleniumURLLoader` 从 URL 列表加载 HTML 文档。

使用 `Selenium` 可以加载需要 JavaScript 渲染的页面。

要使用 `SeleniumURLLoader`，您需要安装 `selenium` 和 `unstructured`。

```python
%pip install --upgrade --quiet selenium unstructured
```

```python
from langchain_community.document_loaders import SeleniumURLLoader

urls = [
    "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
    "https://goo.gl/maps/NDSHwePEyaHMFGwh8",
]

loader = SeleniumURLLoader(urls=urls)

data = loader.load()

data[1]
```

## Playwright URL Loader

本文介绍如何使用 `PlaywrightURLLoader` 从 URL 列表加载 HTML 文档。

[Playwright](https://playwright.dev/) 为现代 web 应用程序提供可靠的端到端测试。

与 Selenium 的情况一样，`Playwright` 允许我们加载和渲染 JavaScript 页面。

要使用 `PlaywrightURLLoader`，您需要安装 `playwright` 和 `unstructured`。此外，您还需要安装 `Playwright Chromium` 浏览器：

```python
%pip install --upgrade --quiet playwright unstructured
```

```python
!playwright install
```

目前，仅支持异步方法：

```python
from langchain_community.document_loaders import PlaywrightURLLoader

urls = [
    "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
    "https://goo.gl/maps/NDSHwePEyaHMFGwh8",
]

loader = PlaywrightURLLoader(urls=urls, remove_selectors=["header", "footer"])

data = await loader.aload()

data[0]
```

```output
Document(page_content="Rick Astley - Never Gonna Give You Up (Official Music Video)\n\nSearch\n\nWatch later\n\nShare\n\nCopy link\n\nInfo\n\nShopping\n\nTap to unmute\n\n2x\n\nIf playback doesn't begin shortly, try restarting your device.\n\nUp next\n\nLiveUpcoming\n\nPlay Now\n\nRick Astley\n\nSubscribe\n\nSubscribed\n\nThe new album, 'Are We There Yet?' out now!\n\nRick Astley - Forever and More (Official Video)3:47\n\nThis video is unavailable\n\nAre We There Yet?15 videos\n\nRick Astley\n\nSubscribe\n\nSubscribed\n\nYou're signed out\n\nVideos you watch may be added to the TV's watch history and influence TV recommendations. To avoid this, cancel and sign in to YouTube on your computer.\n\nShare\n\nAn error occurred while retrieving sharing information. Please try again later.\n\n0:00\n\n0:00 / 3:32\n\nWatch full video\n\n•\n\nScroll for details\n\n•\n              \n            \n          \n        \n        \n          \n            \n          \n        \n      \n      \n    \n    NaN / NaN\n\nNaN / NaN\n\nNaN / NaN\n\nNaN / NaN\n\nNaN / NaN\n\nSearch", metadata={'source': 'https://www.youtube.com/watch?v=dQw4w9WgXcQ'})
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
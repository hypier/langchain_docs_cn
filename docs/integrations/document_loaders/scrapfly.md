---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/scrapfly.ipynb
---

## ScrapFly
[ScrapFly](https://scrapfly.io/) 是一个具有无头浏览器功能、代理和反机器人绕过能力的网页抓取 API。它允许将网页数据提取为可访问的 LLM markdown 或文本。

#### 安装
使用 pip 安装 ScrapFly Python SDK 及所需的 Langchain 包：
```shell
pip install scrapfly-sdk langchain langchain-community
```

#### 使用方法

```python
from langchain_community.document_loaders import ScrapflyLoader

scrapfly_loader = ScrapflyLoader(
    ["https://web-scraping.dev/products"],
    api_key="Your ScrapFly API key",  # 从 https://www.scrapfly.io/ 获取您的 API 密钥
    continue_on_failure=True,  # 忽略无法处理的网页并记录其异常
)

# 从 URL 加载文档为 markdown
documents = scrapfly_loader.load()
print(documents)
```

ScrapflyLoader 还允许传递 ScrapeConfig 对象以自定义抓取请求。有关完整功能细节及其 API 参数，请参阅文档：https://scrapfly.io/docs/scrape-api/getting-started

```python
from langchain_community.document_loaders import ScrapflyLoader

scrapfly_scrape_config = {
    "asp": True,  # 绕过抓取阻止和反机器人解决方案，如 Cloudflare
    "render_js": True,  # 使用云无头浏览器启用 JavaScript 渲染
    "proxy_pool": "public_residential_pool",  # 选择代理池（数据中心或住宅）
    "country": "us",  # 选择代理位置
    "auto_scroll": True,  # 自动滚动页面
    "js": "",  # 通过无头浏览器执行自定义 JavaScript 代码
}

scrapfly_loader = ScrapflyLoader(
    ["https://web-scraping.dev/products"],
    api_key="Your ScrapFly API key",  # 从 https://www.scrapfly.io/ 获取您的 API 密钥
    continue_on_failure=True,  # 忽略无法处理的网页并记录其异常
    scrape_config=scrapfly_scrape_config,  # 传递 scrape_config 对象
    scrape_format="markdown",  # 抓取结果格式，可以是 `markdown`（默认）或 `text`
)

# 从 URL 加载文档为 markdown
documents = scrapfly_loader.load()
print(documents)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
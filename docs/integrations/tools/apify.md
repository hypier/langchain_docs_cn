---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/apify.ipynb
---

# Apify

本笔记展示了如何使用 [Apify integration](/docs/integrations/providers/apify) 进行 LangChain 的集成。

[Apify](https://apify.com) 是一个用于网页抓取和数据提取的云平台，提供了一个包含超过一千个现成应用的 [ecosystem](https://apify.com/store)，这些应用被称为 *Actors*，适用于各种网页抓取、爬虫和数据提取的用例。例如，您可以使用它提取 Google 搜索结果、Instagram 和 Facebook 个人资料、Amazon 或 Shopify 的产品、Google Maps 评论等。

在这个示例中，我们将使用 [Website Content Crawler](https://apify.com/apify/website-content-crawler) Actor，它可以深入爬取文档、知识库、帮助中心或博客等网站，并提取网页上的文本内容。然后，我们将文档输入到向量索引中，并从中回答问题。

```python
%pip install --upgrade --quiet  apify-client langchain-community langchain-openai langchain
```

首先，将 `ApifyWrapper` 导入您的源代码：

```python
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.utilities import ApifyWrapper
from langchain_core.documents import Document
from langchain_openai import OpenAI
from langchain_openai.embeddings import OpenAIEmbeddings
```

使用您的 [Apify API token](https://docs.apify.com/platform/integrations/api#api-token) 初始化它，并为本示例的目的，也使用您的 OpenAI API 密钥：

```python
import os

os.environ["OPENAI_API_KEY"] = "Your OpenAI API key"
os.environ["APIFY_API_TOKEN"] = "Your Apify API token"

apify = ApifyWrapper()
```

然后运行 Actor，等待其完成，并从 Apify 数据集中获取结果到 LangChain 文档加载器中。

请注意，如果您已经在 Apify 数据集中有一些结果，您可以直接使用 `ApifyDatasetLoader` 加载它们，如 [此笔记本](/docs/integrations/document_loaders/apify_dataset) 中所示。在该笔记本中，您还会找到 `dataset_mapping_function` 的解释，该函数用于将 Apify 数据集记录中的字段映射到 LangChain `Document` 字段。

```python
loader = apify.call_actor(
    actor_id="apify/website-content-crawler",
    run_input={"startUrls": [{"url": "https://python.langchain.com"}]},
    dataset_mapping_function=lambda item: Document(
        page_content=item["text"] or "", metadata={"source": item["url"]}
    ),
)
```

从爬取的文档初始化向量索引：

```python
index = VectorstoreIndexCreator(embedding=OpenAIEmbeddings()).from_loaders([loader])
```

最后，查询向量索引：

```python
query = "What is LangChain?"
result = index.query_with_sources(query, llm=OpenAI())
```

```python
print(result["answer"])
print(result["sources"])
```
```output
 LangChain is a standard interface through which you can interact with a variety of large language models (LLMs). It provides modules that can be used to build language model applications, and it also provides chains and agents with memory capabilities.

https://python.langchain.com/en/latest/modules/models/llms.html, https://python.langchain.com/en/latest/getting_started/getting_started.html
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)
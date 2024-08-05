---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/pebblo.ipynb
---

# Pebblo Safe DocumentLoader

> [Pebblo](https://daxa-ai.github.io/pebblo/) 使开发者能够安全地加载数据，并在不担心组织的合规性和安全要求的情况下将其 Gen AI 应用程序推广到部署。该项目识别加载数据中发现的语义主题和实体，并在 UI 或 PDF 报告中对其进行总结。

Pebblo 由两个组件组成。

1. Pebblo Safe DocumentLoader for Langchain
1. Pebblo Server

本文档描述了如何使用 Pebblo Safe DocumentLoader 增强您现有的 Langchain DocumentLoader，以便深入了解输入到 Gen-AI Langchain 应用程序中的主题和实体类型。有关 `Pebblo Server` 的详细信息，请参阅此 [pebblo server](https://daxa-ai.github.io/pebblo/daemon) 文档。

Pebblo Safeloader 使 Langchain `DocumentLoader` 的安全数据摄取成为可能。这是通过用 `Pebblo Safe DocumentLoader` 包装文档加载器调用来完成的。

注意：要在 Pebblo 的默认 URL（localhost:8000）以外的某个 URL 上配置 Pebblo 服务器，请在 `PEBBLO_CLASSIFIER_URL` 环境变量中放入正确的 URL。这也可以通过 `classifier_url` 关键字参数进行配置。参考：[server-configurations](https://daxa-ai.github.io/pebblo/config)

#### 如何启用 Pebblo 文档加载？

假设一个使用 `CSVLoader` 读取 CSV 文档进行推理的 Langchain RAG 应用程序代码片段。

以下是使用 `CSVLoader` 的文档加载代码片段。

```python
from langchain_community.document_loaders import CSVLoader

loader = CSVLoader("data/corp_sens_data.csv")
documents = loader.load()
print(documents)
```

可以通过对上述代码片段进行几行代码更改来启用 Pebblo SafeLoader。

```python
from langchain_community.document_loaders import CSVLoader, PebbloSafeLoader

loader = PebbloSafeLoader(
    CSVLoader("data/corp_sens_data.csv"),
    name="acme-corp-rag-1",  # App name (Mandatory)
    owner="Joe Smith",  # Owner (Optional)
    description="Support productivity RAG application",  # Description (Optional)
)
documents = loader.load()
print(documents)
```

### 将语义主题和身份发送到 Pebblo 云服务器

要将语义数据发送到 pebblo-cloud，将 api-key 作为参数传递给 PebbloSafeLoader，或者将 api-key 放入 `PEBBLO_API_KEY` 环境变量中。

```python
from langchain_community.document_loaders import CSVLoader, PebbloSafeLoader

loader = PebbloSafeLoader(
    CSVLoader("data/corp_sens_data.csv"),
    name="acme-corp-rag-1",  # App name (Mandatory)
    owner="Joe Smith",  # Owner (Optional)
    description="Support productivity RAG application",  # Description (Optional)
    api_key="my-api-key",  # API key (Optional, can be set in the environment variable PEBBLO_API_KEY)
)
documents = loader.load()
print(documents)
```

### 为加载的元数据添加语义主题和实体

要为加载文档的元数据添加语义主题和语义实体，将 `load_semantic` 设置为 True 作为参数，或者定义一个新的环境变量 `PEBBLO_LOAD_SEMANTIC`，并将其设置为 True。

```python
from langchain_community.document_loaders import CSVLoader, PebbloSafeLoader

loader = PebbloSafeLoader(
    CSVLoader("data/corp_sens_data.csv"),
    name="acme-corp-rag-1",  # App name (Mandatory)
    owner="Joe Smith",  # Owner (Optional)
    description="Support productivity RAG application",  # Description (Optional)
    api_key="my-api-key",  # API key (Optional, can be set in the environment variable PEBBLO_API_KEY)
    load_semantic=True,  # Load semantic data (Optional, default is False, can be set in the environment variable PEBBLO_LOAD_SEMANTIC)
)
documents = loader.load()
print(documents[0].metadata)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)
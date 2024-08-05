---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/bedrock.ipynb
---

# Bedrock

>[Amazon Bedrock](https://aws.amazon.com/bedrock/) 是一项完全托管的服务，提供来自 `AI21 Labs`、`Anthropic`、`Cohere`、`Meta`、`Stability AI` 和 `Amazon` 等领先 AI 公司的一系列高性能基础模型 (FMs)，通过单一 API 访问，并提供构建具备安全性、隐私性和负责任 AI 的生成 AI 应用所需的广泛能力。使用 `Amazon Bedrock`，您可以轻松实验和评估适合您用例的顶级 FMs，使用微调和 `Retrieval Augmented Generation` (`RAG`) 等技术私下定制它们，并构建能够使用您的企业系统和数据源执行任务的代理。由于 `Amazon Bedrock` 是无服务器的，您无需管理任何基础设施，您可以安全地将生成 AI 能力集成并部署到您已经熟悉的 AWS 服务中的应用程序中。




```python
%pip install --upgrade --quiet  boto3
```


```python
from langchain_community.embeddings import BedrockEmbeddings

embeddings = BedrockEmbeddings(
    credentials_profile_name="bedrock-admin", region_name="us-east-1"
)
```


```python
embeddings.embed_query("This is a content of the document")
```


```python
embeddings.embed_documents(
    ["This is a content of the document", "This is another document"]
)
```


```python
# async embed query
await embeddings.aembed_query("This is a content of the document")
```


```python
# async embed documents
await embeddings.aembed_documents(
    ["This is a content of the document", "This is another document"]
)
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)
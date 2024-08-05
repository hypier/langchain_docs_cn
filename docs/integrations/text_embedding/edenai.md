---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/edenai.ipynb
---

# EDEN AI

Eden AI 正在通过联合最佳的 AI 提供商来彻底改变 AI 领域，使用户能够解锁无限可能，挖掘人工智能的真正潜力。通过一个全面且无忧的平台，它允许用户快速将 AI 功能部署到生产环境中，使用户能够通过单一 API 轻松访问所有 AI 能力。(网站: https://edenai.co/)

本示例介绍如何使用 LangChain 与 Eden AI 嵌入模型进行交互

-----------------------------------------------------------------------------------


访问 EDENAI 的 API 需要一个 API 密钥，

您可以通过创建一个账户 https://app.edenai.run/user/register 并前往 https://app.edenai.run/admin/account/settings 来获取。

一旦我们有了密钥，我们想通过运行以下命令将其设置为环境变量：

```shell
export EDENAI_API_KEY="..."
```


如果您不想设置环境变量，可以通过在初始化 EdenAI 嵌入类时直接传递名为 edenai_api_key 的参数来传递密钥：

```python
from langchain_community.embeddings.edenai import EdenAiEmbeddings
```


```python
embeddings = EdenAiEmbeddings(edenai_api_key="...", provider="...")
```

## 调用模型


EdenAI API 汇集了各种提供商。

要访问特定模型，您可以在调用时简单地使用“提供商”。

```python
embeddings = EdenAiEmbeddings(provider="openai")
```

```python
docs = ["It's raining right now", "cats are cute"]
document_result = embeddings.embed_documents(docs)
```

```python
query = "my umbrella is broken"
query_result = embeddings.embed_query(query)
```

```python
import numpy as np

query_numpy = np.array(query_result)
for doc_res, doc in zip(document_result, docs):
    document_numpy = np.array(doc_res)
    similarity = np.dot(query_numpy, document_numpy) / (
        np.linalg.norm(query_numpy) * np.linalg.norm(document_numpy)
    )
    print(f'Cosine similarity between "{doc}" and query: {similarity}')
```
```output
Cosine similarity between "It's raining right now" and query: 0.849261496107252
Cosine similarity between "cats are cute" and query: 0.7525900655705218
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/google_vertex_ai_palm.ipynb
---

# Google Vertex AI PaLM 

>[Vertex AI PaLM API](https://cloud.google.com/vertex-ai/docs/generative-ai/learn/overview) 是 Google Cloud 上的一个服务，提供嵌入模型。

注意：此集成与 Google PaLM 集成是分开的。

默认情况下，Google Cloud [不使用](https://cloud.google.com/vertex-ai/docs/generative-ai/data-governance#foundation_model_development) 客户数据来训练其基础模型，这是 Google Cloud AI/ML 隐私承诺的一部分。有关 Google 如何处理数据的更多详细信息，请参见 [Google 的客户数据处理附录 (CDPA)](https://cloud.google.com/terms/data-processing-addendum)。

要使用 Vertex AI PaLM，您必须安装 `langchain-google-vertexai` Python 包，并且：
- 为您的环境配置凭据（gcloud、工作负载身份等...）
- 将服务帐户 JSON 文件的路径存储为 GOOGLE_APPLICATION_CREDENTIALS 环境变量

此代码库使用 `google.auth` 库，首先查找上述提到的应用凭据变量，然后查找系统级别的身份验证。

有关更多信息，请参见：
- https://cloud.google.com/docs/authentication/application-default-credentials#GAC
- https://googleapis.dev/python/google-auth/latest/reference/google.auth.html#module-google.auth




```python
%pip install --upgrade --quiet langchain langchain-google-vertexai
```


```python
from langchain_google_vertexai import VertexAIEmbeddings
```


```python
embeddings = VertexAIEmbeddings()
```


```python
text = "This is a test document."
```


```python
query_result = embeddings.embed_query(text)
```


```python
doc_result = embeddings.embed_documents([text])
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)
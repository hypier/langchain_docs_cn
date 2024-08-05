---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/yandex.ipynb
---

# YandexGPT

本笔记本介绍如何使用 Langchain 与 [YandexGPT](https://cloud.yandex.com/en/services/yandexgpt) 嵌入模型。

要使用它，您需要安装 `yandexcloud` python 包。

```python
%pip install --upgrade --quiet  yandexcloud
```

首先，您应该 [创建服务账户](https://cloud.yandex.com/en/docs/iam/operations/sa/create)，并赋予 `ai.languageModels.user` 角色。

接下来，您有两种认证选项：
- [IAM 令牌](https://cloud.yandex.com/en/docs/iam/operations/iam-token/create-for-sa)。
  您可以在构造函数参数 `iam_token` 中指定令牌，或在环境变量 `YC_IAM_TOKEN` 中指定。
- [API 密钥](https://cloud.yandex.com/en/docs/iam/operations/api-key/create)。
  您可以在构造函数参数 `api_key` 中指定密钥，或在环境变量 `YC_API_KEY` 中指定。

要指定模型，您可以使用 `model_uri` 参数，详细信息请参见 [文档](https://cloud.yandex.com/en/docs/yandexgpt/concepts/models#yandexgpt-embeddings)。

默认情况下，将使用从参数 `folder_id` 或环境变量 `YC_FOLDER_ID` 指定的文件夹中获取的最新版本的 `text-search-query`。

```python
from langchain_community.embeddings.yandex import YandexGPTEmbeddings
```

```python
embeddings = YandexGPTEmbeddings()
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

```python
query_result[:5]
```

```output
[-0.021392822265625,
 0.096435546875,
 -0.046966552734375,
 -0.0183258056640625,
 -0.00555419921875]
```

```python
doc_result[0][:5]
```

```output
[-0.021392822265625,
 0.096435546875,
 -0.046966552734375,
 -0.0183258056640625,
 -0.00555419921875]
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)
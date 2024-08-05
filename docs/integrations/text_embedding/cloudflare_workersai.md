---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/cloudflare_workersai.ipynb
---

# Cloudflare Workers AI

>[Cloudflare, Inc. (维基百科)](https://en.wikipedia.org/wiki/Cloudflare) 是一家美国公司，提供内容分发网络服务、云网络安全、DDoS 缓解和 ICANN 认证的域名注册服务。

>[Cloudflare Workers AI](https://developers.cloudflare.com/workers-ai/) 允许您通过 REST API 在 `Cloudflare` 网络上运行机器学习模型。

>[Cloudflare AI 文档](https://developers.cloudflare.com/workers-ai/models/text-embeddings/) 列出了所有可用的文本嵌入模型。

## 设置

需要 Cloudflare 账户 ID 和 API 令牌。请参阅 [此文档](https://developers.cloudflare.com/workers-ai/get-started/rest-api/) 获取如何获取它们的信息。



```python
import getpass

my_account_id = getpass.getpass("Enter your Cloudflare account ID:\n\n")
my_api_token = getpass.getpass("Enter your Cloudflare API token:\n\n")
```

## 示例


```python
from langchain_community.embeddings.cloudflare_workersai import (
    CloudflareWorkersAIEmbeddings,
)
```


```python
embeddings = CloudflareWorkersAIEmbeddings(
    account_id=my_account_id,
    api_token=my_api_token,
    model_name="@cf/baai/bge-small-en-v1.5",
)
# 单字符串嵌入
query_result = embeddings.embed_query("test")
len(query_result), query_result[:3]
```



```output
(384, [-0.033627357333898544, 0.03982774540781975, 0.03559349477291107])
```



```python
# 批量字符串嵌入
batch_query_result = embeddings.embed_documents(["test1", "test2", "test3"])
len(batch_query_result), len(batch_query_result[0])
```



```output
(3, 384)
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)
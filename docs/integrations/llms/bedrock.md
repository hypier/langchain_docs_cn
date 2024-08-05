---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/bedrock.ipynb
---

# Bedrock

:::caution
您当前正在查看有关使用 Amazon Bedrock 模型作为 [文本补全模型](/docs/concepts/#llms) 的文档。许多在 Bedrock 上可用的流行模型是 [聊天补全模型](/docs/concepts/#chat-models)。

您可能想查看 [此页面](/docs/integrations/chat/bedrock/)。
:::

>[Amazon Bedrock](https://aws.amazon.com/bedrock/) 是一项完全托管的服务，提供来自 `AI21 Labs`、`Anthropic`、`Cohere`、`Meta`、`Stability AI` 和 `Amazon` 等领先 AI 公司的一系列高性能基础模型（FMs），通过单一 API 访问，并提供构建具有安全性、隐私性和负责任 AI 的生成 AI 应用所需的广泛功能。使用 `Amazon Bedrock`，您可以轻松地实验和评估适合您用例的顶级 FMs，使用微调和 `检索增强生成`（`RAG`）等技术私下定制它们，并构建利用您的企业系统和数据源执行任务的代理。由于 `Amazon Bedrock` 是无服务器的，您无需管理任何基础设施，并且可以使用您已经熟悉的 AWS 服务将生成 AI 功能安全集成和部署到您的应用中。



```python
%pip install --upgrade --quiet langchain_aws
```


```python
from langchain_aws import BedrockLLM

llm = BedrockLLM(
    credentials_profile_name="bedrock-admin", model_id="amazon.titan-text-express-v1"
)
```

### 自定义模型


```python
custom_llm = BedrockLLM(
    credentials_profile_name="bedrock-admin",
    provider="cohere",
    model_id="<Custom model ARN>",  # ARN 类似 'arn:aws:bedrock:...' 通过配置自定义模型获得
    model_kwargs={"temperature": 1},
    streaming=True,
)

custom_llm.invoke(input="What is the recipe of mayonnaise?")
```

## Amazon Bedrock 的保护措施

[Amazon Bedrock 的保护措施](https://aws.amazon.com/bedrock/guardrails/) 基于特定用例的政策评估用户输入和模型响应，并提供额外的保护层，无论底层模型如何。保护措施可以应用于多个模型，包括 Anthropic Claude、Meta Llama 2、Cohere Command、AI21 Labs Jurassic 和 Amazon Titan Text，以及微调模型。
**注意**：Amazon Bedrock 的保护措施目前处于预览阶段，尚未普遍提供。如果您希望访问此功能，请通过您通常的 AWS 支持联系渠道进行联系。
在本节中，我们将设置一个具有特定保护措施的 Bedrock 语言模型，其中包括跟踪功能。   

```python
from typing import Any

from langchain_core.callbacks import AsyncCallbackHandler


class BedrockAsyncCallbackHandler(AsyncCallbackHandler):
    # Async callback handler that can be used to handle callbacks from langchain.

    async def on_llm_error(self, error: BaseException, **kwargs: Any) -> Any:
        reason = kwargs.get("reason")
        if reason == "GUARDRAIL_INTERVENED":
            print(f"Guardrails: {kwargs}")


# Guardrails for Amazon Bedrock with trace
llm = BedrockLLM(
    credentials_profile_name="bedrock-admin",
    model_id="<Model_ID>",
    model_kwargs={},
    guardrails={"id": "<Guardrail_ID>", "version": "<Version>", "trace": True},
    callbacks=[BedrockAsyncCallbackHandler()],
)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)
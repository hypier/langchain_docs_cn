---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/huggingface_endpoint.ipynb
---

# Huggingface 端点

> [Hugging Face Hub](https://huggingface.co/docs/hub/index) 是一个平台，拥有超过 120,000 个模型、20,000 个数据集和 50,000 个演示应用（Spaces），所有内容均为开源和公开可用，提供一个人们可以轻松协作和共同构建机器学习的在线平台。

`Hugging Face Hub` 还提供各种端点来构建机器学习应用。
本示例展示了如何连接到不同的端点类型。

特别地，文本生成推理由 [Text Generation Inference](https://github.com/huggingface/text-generation-inference) 提供支持：一个定制构建的 Rust、Python 和 gRPC 服务器，用于超快速的文本生成推理。


```python
from langchain_huggingface import HuggingFaceEndpoint
```

## 安装与设置

要使用此功能，您需要安装 ``huggingface_hub`` python [包](https://huggingface.co/docs/huggingface_hub/installation)。

```python
%pip install --upgrade --quiet huggingface_hub
```

```python
# 获取令牌: https://huggingface.co/docs/api-inference/quicktour#get-your-api-token

from getpass import getpass

HUGGINGFACEHUB_API_TOKEN = getpass()
```

```python
import os

os.environ["HUGGINGFACEHUB_API_TOKEN"] = HUGGINGFACEHUB_API_TOKEN
```

## 准备示例


```python
from langchain_huggingface import HuggingFaceEndpoint
```


```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
```


```python
question = "Who won the FIFA World Cup in the year 1994? "

template = """Question: {question}

Answer: Let's think step by step."""

prompt = PromptTemplate.from_template(template)
```

## 示例

以下是如何访问免费的 [Serverless Endpoints](https://huggingface.co/inference-endpoints/serverless) API 的 `HuggingFaceEndpoint` 集成的示例。

```python
repo_id = "mistralai/Mistral-7B-Instruct-v0.2"

llm = HuggingFaceEndpoint(
    repo_id=repo_id,
    max_length=128,
    temperature=0.5,
    huggingfacehub_api_token=HUGGINGFACEHUB_API_TOKEN,
)
llm_chain = prompt | llm
print(llm_chain.invoke({"question": question}))
```

## 专用端点

免费的无服务器 API 让您可以快速实现解决方案和迭代，但对于重负载使用情况，可能会受到速率限制，因为负载与其他请求共享。

对于企业工作负载，最佳选择是使用 [Inference Endpoints - Dedicated](https://huggingface.co/inference-endpoints/dedicated)。这提供了一个完全托管的基础设施，提供更多的灵活性和速度。这些资源提供持续的支持和正常运行时间保证，以及自动扩展等选项。

```python
# Set the url to your Inference Endpoint below
your_endpoint_url = "https://fayjubiy2xqn36z0.us-east-1.aws.endpoints.huggingface.cloud"
```

```python
llm = HuggingFaceEndpoint(
    endpoint_url=f"{your_endpoint_url}",
    max_new_tokens=512,
    top_k=10,
    top_p=0.95,
    typical_p=0.95,
    temperature=0.01,
    repetition_penalty=1.03,
)
llm("What did foo say about bar?")
```

### 流式传输


```python
from langchain_core.callbacks import StreamingStdOutCallbackHandler
from langchain_huggingface import HuggingFaceEndpoint

llm = HuggingFaceEndpoint(
    endpoint_url=f"{your_endpoint_url}",
    max_new_tokens=512,
    top_k=10,
    top_p=0.95,
    typical_p=0.95,
    temperature=0.01,
    repetition_penalty=1.03,
    streaming=True,
)
llm("What did foo say about bar?", callbacks=[StreamingStdOutCallbackHandler()])
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)
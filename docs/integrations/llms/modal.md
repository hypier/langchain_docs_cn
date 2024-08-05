---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/modal.ipynb
---

# Modal

[Modal 云平台](https://modal.com/docs/guide) 提供方便的按需访问服务器无关的云计算，用户可以通过本地计算机上的 Python 脚本进行访问。 
使用 `modal` 来运行您自己的自定义 LLM 模型，而不是依赖 LLM API。

此示例介绍了如何使用 LangChain 与 `modal` HTTPS [网络端点](https://modal.com/docs/guide/webhooks) 进行交互。

[_使用 LangChain 进行问答_](https://modal.com/docs/guide/ex/potus_speech_qanda) 是另一个如何将 LangChain 与 `Modal` 一起使用的示例。在该示例中，Modal 端到端运行 LangChain 应用，并使用 OpenAI 作为其 LLM API。

```python
%pip install --upgrade --quiet  modal
```

```python
# 注册一个 Modal 账户并获取一个新的令牌。

!modal token new
```
```output
Launching login page in your browser window...
If this is not showing up, please copy this URL into your web browser manually:
https://modal.com/token-flow/tf-Dzm3Y01234mqmm1234Vcu3
```
[`langchain.llms.modal.Modal`](https://github.com/langchain-ai/langchain/blame/master/langchain/llms/modal.py) 集成类要求您部署一个具有符合以下 JSON 接口的网络端点的 Modal 应用：

1. LLM 提示作为 `str` 值在键 `"prompt"` 下接受
2. LLM 响应作为 `str` 值在键 `"prompt"` 下返回

**示例请求 JSON:**

```json
{
    "prompt": "Identify yourself, bot!",
    "extra": "args are allowed",
}
```

**示例响应 JSON:**

```json
{
    "prompt": "This is the LLM speaking",
}
```

一个满足此接口的“虚拟” Modal 网络端点函数示例为

```python
...
...

class Request(BaseModel):
    prompt: str

@stub.function()
@modal.web_endpoint(method="POST")
def web(request: Request):
    _ = request  # ignore input
    return {"prompt": "hello world"}
```

* 请参阅 Modal 的 [网络端点](https://modal.com/docs/guide/webhooks#passing-arguments-to-web-endpoints) 指南，了解设置满足此接口的端点的基础知识。
* 请参阅 Modal 的 ['Run Falcon-40B with AutoGPTQ'](https://modal.com/docs/guide/ex/falcon_gptq) 开源 LLM 示例，作为您自定义 LLM 的起点！

一旦您部署了 Modal 网络端点，您可以将其 URL 传递给 `langchain.llms.modal.Modal` LLM 类。该类可以作为您链中的构建块。

```python
from langchain.chains import LLMChain
from langchain_community.llms import Modal
from langchain_core.prompts import PromptTemplate
```

```python
template = """Question: {question}

Answer: Let's think step by step."""
```

```python
endpoint_url = "https://ecorp--custom-llm-endpoint.modal.run"  # REPLACE ME with your deployed Modal web endpoint's URL
llm = Modal(endpoint_url=endpoint_url)
```

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

```python
question = "What NFL team won the Super Bowl in the year Justin Beiber was born?"

llm_chain.run(question)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)
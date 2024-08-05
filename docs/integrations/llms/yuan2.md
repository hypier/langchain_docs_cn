---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/yuan2.ipynb
---

# Yuan2.0

[Yuan2.0](https://github.com/IEIT-Yuan/Yuan-2.0) 是由 IEIT System 开发的新一代基础大型语言模型。我们已经发布了三个模型，Yuan 2.0-102B、Yuan 2.0-51B 和 Yuan 2.0-2B。我们为其他开发者提供了相关的预训练、微调和推理服务脚本。Yuan2.0 基于 Yuan1.0，利用更广泛的高质量预训练数据和指令微调数据集来增强模型对语义、数学、推理、代码、知识等方面的理解。

本例介绍了如何使用 LangChain 与 `Yuan2.0`(2B/51B/102B) 推理进行文本生成。

Yuan2.0 设置了一个推理服务，用户只需请求推理 API 即可获取结果，详细信息见 [Yuan2.0 Inference-Server](https://github.com/IEIT-Yuan/Yuan-2.0/blob/main/docs/inference_server.md)。

```python
from langchain.chains import LLMChain
from langchain_community.llms.yuan2 import Yuan2
```

```python
# default infer_api for a local deployed Yuan2.0 inference server
infer_api = "http://127.0.0.1:8000/yuan"

# direct access endpoint in a proxied environment
# import os
# os.environ["no_proxy"]="localhost,127.0.0.1,::1"

yuan_llm = Yuan2(
    infer_api=infer_api,
    max_tokens=2048,
    temp=1.0,
    top_p=0.9,
    use_history=False,
)

# turn on use_history only when you want the Yuan2.0 to keep track of the conversation history
# and send the accumulated context to the backend model api, which make it stateful. By default it is stateless.
# llm.use_history = True
```

```python
question = "请介绍一下中国。"
```

```python
print(yuan_llm.invoke(question))
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)
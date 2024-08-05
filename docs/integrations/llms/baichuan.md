---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/baichuan.ipynb
---

# 百川 LLM
百川公司 (https://www.baichuan-ai.com/) 是一家专注于解决人类基本需求的中国初创企业，涵盖效率、健康和幸福。

```python
##Installing the langchain packages needed to use the integration
%pip install -qU langchain-community
```

## 前提条件
需要一个 API 密钥才能访问 Baichuan LLM API。访问 https://platform.baichuan-ai.com/ 获取您的 API 密钥。

## 使用 Baichuan LLM


```python
import os

os.environ["BAICHUAN_API_KEY"] = "YOUR_API_KEY"
```


```python
from langchain_community.llms import BaichuanLLM

# 加载模型
llm = BaichuanLLM()

res = llm.invoke("What's your name?")
print(res)
```


```python
res = llm.generate(prompts=["你好！"])
res
```


```python
for res in llm.stream("Who won the second world war?"):
    print(res)
```


```python
import asyncio


async def run_aio_stream():
    async for res in llm.astream("Write a poem about the sun."):
        print(res)


asyncio.run(run_aio_stream())
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)
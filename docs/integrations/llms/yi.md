---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/yi.ipynb
---

# Yi
[01.AI](https://www.lingyiwanwu.com/en)由李开复博士创立，是一家处于AI 2.0前沿的全球公司。他们提供尖端的大型语言模型，包括Yi系列，参数范围从6B到数百亿。01.AI还提供多模态模型、开放API平台以及开源选项，如Yi-34B/9B/6B和Yi-VL。

```python
## Installing the langchain packages needed to use the integration
%pip install -qU langchain-community
```

## 前提条件
访问 Yi LLM API 需要一个 API 密钥。请访问 https://www.lingyiwanwu.com/ 获取您的 API 密钥。在申请 API 密钥时，您需要指定是用于国内（中国）还是国际使用。

## 使用 Yi LLM


```python
import os

os.environ["YI_API_KEY"] = "YOUR_API_KEY"

from langchain_community.llms import YiLLM

# 加载模型
llm = YiLLM(model="yi-large")

# 如有需要可以指定区域（默认是 "auto"）
# llm = YiLLM(model="yi-large", region="domestic")  # 或 "international"

# 基本用法
res = llm.invoke("你叫什么名字？")
print(res)
```


```python
# 生成方法
res = llm.generate(
    prompts=[
        "解释大语言模型的概念。",
        "人工智能在医疗保健中的潜在应用是什么？",
    ]
)
print(res)
```


```python
# 流式传输
for chunk in llm.stream("描述 Yi 语言模型系列的关键特性。"):
    print(chunk, end="", flush=True)
```


```python
# 异步流式传输
import asyncio


async def run_aio_stream():
    async for chunk in llm.astream(
        "根据李开复博士的愿景，写一篇关于人工智能未来的简报。"
    ):
        print(chunk, end="", flush=True)


asyncio.run(run_aio_stream())
```


```python
# 调整参数
llm_with_params = YiLLM(
    model="yi-large",
    temperature=0.7,
    top_p=0.9,
)

res = llm_with_params(
    "提出一个可以惠及社会的创新 AI 应用。"
)
print(res)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)
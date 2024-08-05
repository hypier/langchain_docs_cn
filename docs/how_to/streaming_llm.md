---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/streaming_llm.ipynb
---

# 如何从 LLM 流式传输响应

所有 `LLM` 都实现了 [Runnable interface](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable)，该接口提供了标准可运行方法的 **默认** 实现（即 `ainvoke`、`batch`、`abatch`、`stream`、`astream`、`astream_events`）。

**默认** 的流式实现提供了一个 `Iterator`（或用于异步流式传输的 `AsyncIterator`），它生成一个单一值：来自底层聊天模型提供者的最终输出。

逐个标记流式输出的能力取决于提供者是否实现了适当的流式支持。

请查看哪些 [集成支持逐个标记流式传输](/docs/integrations/llms/)。

:::note

**默认** 实现不提供逐个标记流式传输的支持，但它确保模型可以替换为任何其他模型，因为它支持相同的标准接口。

:::

## 同步流

下面我们使用 `|` 来帮助可视化标记之间的分隔符。

```python
from langchain_openai import OpenAI

llm = OpenAI(model="gpt-3.5-turbo-instruct", temperature=0, max_tokens=512)
for chunk in llm.stream("Write me a 1 verse song about sparkling water."):
    print(chunk, end="|", flush=True)
```
```output


|Spark|ling| water|,| oh| so clear|
|Bubbles dancing|,| without| fear|
|Refreshing| taste|,| a| pure| delight|
|Spark|ling| water|,| my| thirst|'s| delight||
```

## 异步流

让我们看看如何在异步环境中使用 `astream` 进行流式传输。


```python
from langchain_openai import OpenAI

llm = OpenAI(model="gpt-3.5-turbo-instruct", temperature=0, max_tokens=512)
async for chunk in llm.astream("Write me a 1 verse song about sparkling water."):
    print(chunk, end="|", flush=True)
```
```output


|Spark|ling| water|,| oh| so clear|
|Bubbles dancing|,| without| fear|
|Refreshing| taste|,| a| pure| delight|
|Spark|ling| water|,| my| thirst|'s| delight||
```

## 异步事件流

LLMs 还支持标准的 [astream events](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.astream_events) 方法。

:::tip

`astream_events` 在实现包含多个步骤的更大 LLM 应用程序（例如，涉及 `agent` 的应用程序）时最为有用。
:::

```python
from langchain_openai import OpenAI

llm = OpenAI(model="gpt-3.5-turbo-instruct", temperature=0, max_tokens=512)

idx = 0

async for event in llm.astream_events(
    "Write me a 1 verse song about goldfish on the moon", version="v1"
):
    idx += 1
    if idx >= 5:  # Truncate the output
        print("...Truncated")
        break
    print(event)
```
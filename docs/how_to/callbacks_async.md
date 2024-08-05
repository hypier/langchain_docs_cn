---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/callbacks_async.ipynb
---

# 如何在异步环境中使用回调

:::info 前提条件

本指南假设您熟悉以下概念：

- [回调](/docs/concepts/#callbacks)
- [自定义回调处理程序](/docs/how_to/custom_callbacks)
:::

如果您计划使用异步 API，建议使用并扩展 [`AsyncCallbackHandler`](https://api.python.langchain.com/en/latest/callbacks/langchain_core.callbacks.base.AsyncCallbackHandler.html) 以避免阻塞事件。

:::warning
如果在使用异步方法运行您的 LLM / Chain / Tool / Agent 时使用了同步 `CallbackHandler`，它仍然可以工作。然而，在底层，它将通过 [`run_in_executor`](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.run_in_executor) 被调用，如果您的 `CallbackHandler` 不是线程安全的，可能会导致问题。
:::

:::danger

如果您使用的是 `python<=3.10`，请记住在从 `RunnableLambda`、`RunnableGenerator` 或 `@tool` 中调用其他 `runnable` 时传播 `config` 或 `callbacks`。如果不这样做，回调将不会传播到被调用的子 runnable。
:::


```python
import asyncio
from typing import Any, Dict, List

from langchain_anthropic import ChatAnthropic
from langchain_core.callbacks import AsyncCallbackHandler, BaseCallbackHandler
from langchain_core.messages import HumanMessage
from langchain_core.outputs import LLMResult


class MyCustomSyncHandler(BaseCallbackHandler):
    def on_llm_new_token(self, token: str, **kwargs) -> None:
        print(f"Sync handler being called in a `thread_pool_executor`: token: {token}")


class MyCustomAsyncHandler(AsyncCallbackHandler):
    """异步回调处理程序，可用于处理来自 langchain 的回调。"""

    async def on_llm_start(
        self, serialized: Dict[str, Any], prompts: List[str], **kwargs: Any
    ) -> None:
        """在链开始运行时运行。"""
        print("zzzz....")
        await asyncio.sleep(0.3)
        class_name = serialized["name"]
        print("Hi! I just woke up. Your llm is starting")

    async def on_llm_end(self, response: LLMResult, **kwargs: Any) -> None:
        """在链结束运行时运行。"""
        print("zzzz....")
        await asyncio.sleep(0.3)
        print("Hi! I just woke up. Your llm is ending")


# 要启用流式处理，我们在 ChatModel 构造函数中传入 `streaming=True`
# 此外，我们传入一个包含自定义处理程序的列表
chat = ChatAnthropic(
    model="claude-3-sonnet-20240229",
    max_tokens=25,
    streaming=True,
    callbacks=[MyCustomSyncHandler(), MyCustomAsyncHandler()],
)

await chat.agenerate([[HumanMessage(content="Tell me a joke")]])
```
```output
zzzz....
Hi! I just woke up. Your llm is starting
Sync handler being called in a `thread_pool_executor`: token: Here
Sync handler being called in a `thread_pool_executor`: token: 's
Sync handler being called in a `thread_pool_executor`: token:  a
Sync handler being called in a `thread_pool_executor`: token:  little
Sync handler being called in a `thread_pool_executor`: token:  joke
Sync handler being called in a `thread_pool_executor`: token:  for
Sync handler being called in a `thread_pool_executor`: token:  you
Sync handler being called in a `thread_pool_executor`: token: :
Sync handler being called in a `thread_pool_executor`: token: 

Why
Sync handler being called in a `thread_pool_executor`: token:  can
Sync handler being called in a `thread_pool_executor`: token: 't
Sync handler being called in a `thread_pool_executor`: token:  a
Sync handler being called in a `thread_pool_executor`: token:  bicycle
Sync handler being called in a `thread_pool_executor`: token:  stan
Sync handler being called in a `thread_pool_executor`: token: d up
Sync handler being called in a `thread_pool_executor`: token:  by
Sync handler being called in a `thread_pool_executor`: token:  itself
Sync handler being called in a `thread_pool_executor`: token: ?
Sync handler being called in a `thread_pool_executor`: token:  Because
Sync handler being called in a `thread_pool_executor`: token:  it
Sync handler being called in a `thread_pool_executor`: token: 's
Sync handler being called in a `thread_pool_executor`: token:  two
Sync handler being called in a `thread_pool_executor`: token: -
Sync handler being called in a `thread_pool_executor`: token: tire
zzzz....
Hi! I just woke up. Your llm is ending
```


```output
LLMResult(generations=[[ChatGeneration(text="Here's a little joke for you:\n\nWhy can't a bicycle stand up by itself? Because it's two-tire", message=AIMessage(content="Here's a little joke for you:\n\nWhy can't a bicycle stand up by itself? Because it's two-tire", id='run-8afc89e8-02c0-4522-8480-d96977240bd4-0'))]], llm_output={}, run=[RunInfo(run_id=UUID('8afc89e8-02c0-4522-8480-d96977240bd4'))])
```

## 下一步

您现在已经学会了如何创建自己的自定义回调处理程序。

接下来，查看本节中的其他操作指南，例如 [如何将回调附加到可运行对象](/docs/how_to/callbacks_attach)。
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/custom_callbacks.ipynb
---

# 如何创建自定义回调处理器

:::info 前提条件

本指南假设您熟悉以下概念：

- [回调](/docs/concepts/#callbacks)

:::

LangChain 有一些内置的回调处理器，但您通常会希望创建自己的具有自定义逻辑的处理器。

要创建一个自定义回调处理器，我们需要确定我们希望回调处理器处理的 [事件](https://api.python.langchain.com/en/latest/callbacks/langchain_core.callbacks.base.BaseCallbackHandler.html#langchain-core-callbacks-base-basecallbackhandler)，以及当事件被触发时我们希望回调处理器执行的操作。然后，我们只需将回调处理器附加到对象上，例如通过 [构造函数](/docs/how_to/callbacks_constructor) 或 [在运行时](/docs/how_to/callbacks_runtime)。

在下面的示例中，我们将实现一个带有自定义处理器的流式传输。

在我们的自定义回调处理器 `MyCustomHandler` 中，我们实现了 `on_llm_new_token` 处理器来打印我们刚刚收到的令牌。然后，我们将自定义处理器作为构造函数回调附加到模型对象上。


```python
from langchain_anthropic import ChatAnthropic
from langchain_core.callbacks import BaseCallbackHandler
from langchain_core.prompts import ChatPromptTemplate


class MyCustomHandler(BaseCallbackHandler):
    def on_llm_new_token(self, token: str, **kwargs) -> None:
        print(f"My custom handler, token: {token}")


prompt = ChatPromptTemplate.from_messages(["Tell me a joke about {animal}"])

# To enable streaming, we pass in `streaming=True` to the ChatModel constructor
# Additionally, we pass in our custom handler as a list to the callbacks parameter
model = ChatAnthropic(
    model="claude-3-sonnet-20240229", streaming=True, callbacks=[MyCustomHandler()]
)

chain = prompt | model

response = chain.invoke({"animal": "bears"})
```
```output
My custom handler, token: Here
My custom handler, token: 's
My custom handler, token:  a
My custom handler, token:  bear
My custom handler, token:  joke
My custom handler, token:  for
My custom handler, token:  you
My custom handler, token: :
My custom handler, token: 

Why
My custom handler, token:  di
My custom handler, token: d the
My custom handler, token:  bear
My custom handler, token:  dissol
My custom handler, token: ve
My custom handler, token:  in
My custom handler, token:  water
My custom handler, token: ?
My custom handler, token: 
Because
My custom handler, token:  it
My custom handler, token:  was
My custom handler, token:  a
My custom handler, token:  polar
My custom handler, token:  bear
My custom handler, token: !
```
您可以查看 [此参考页面](https://api.python.langchain.com/en/latest/callbacks/langchain_core.callbacks.base.BaseCallbackHandler.html#langchain-core-callbacks-base-basecallbackhandler) 以获取您可以处理的事件列表。请注意，`handle_chain_*` 事件适用于大多数 LCEL 可运行对象。

## 后续步骤

您现在已经学习了如何创建自己的自定义回调处理程序。

接下来，请查看本节中的其他操作指南，例如 [如何将回调附加到可运行对象](/docs/how_to/callbacks_attach)。
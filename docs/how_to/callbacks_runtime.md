---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/callbacks_runtime.ipynb
---

# 如何在运行时传递回调

:::info 前提条件

本指南假设您熟悉以下概念：

- [回调](/docs/concepts/#callbacks)
- [自定义回调处理程序](/docs/how_to/custom_callbacks)

:::

在许多情况下，在运行对象时传递处理程序更为有利。当我们在执行运行时通过 `callbacks` 关键字参数传递 [`CallbackHandlers`](https://api.python.langchain.com/en/latest/callbacks/langchain_core.callbacks.base.BaseCallbackHandler.html#langchain-core-callbacks-base-basecallbackhandler)，这些回调将由所有参与执行的嵌套对象发出。例如，当一个处理程序被传递给一个代理时，它将用于与该代理相关的所有回调以及与代理执行相关的所有对象，在这种情况下，是工具和 LLM。

这避免了我们必须手动将处理程序附加到每个单独的嵌套对象。以下是一个示例：

```python
from typing import Any, Dict, List

from langchain_anthropic import ChatAnthropic
from langchain_core.callbacks import BaseCallbackHandler
from langchain_core.messages import BaseMessage
from langchain_core.outputs import LLMResult
from langchain_core.prompts import ChatPromptTemplate


class LoggingHandler(BaseCallbackHandler):
    def on_chat_model_start(
        self, serialized: Dict[str, Any], messages: List[List[BaseMessage]], **kwargs
    ) -> None:
        print("聊天模型已启动")

    def on_llm_end(self, response: LLMResult, **kwargs) -> None:
        print(f"聊天模型结束，响应: {response}")

    def on_chain_start(
        self, serialized: Dict[str, Any], inputs: Dict[str, Any], **kwargs
    ) -> None:
        print(f"链 {serialized.get('name')} 已启动")

    def on_chain_end(self, outputs: Dict[str, Any], **kwargs) -> None:
        print(f"链结束，输出: {outputs}")


callbacks = [LoggingHandler()]
llm = ChatAnthropic(model="claude-3-sonnet-20240229")
prompt = ChatPromptTemplate.from_template("1 + {number} 等于多少？")

chain = prompt | llm

chain.invoke({"number": "2"}, config={"callbacks": callbacks})
```
```output
链 RunnableSequence 已启动
链 ChatPromptTemplate 已启动
链结束，输出: messages=[HumanMessage(content='1 + 2 等于多少？')]
聊天模型已启动
聊天模型结束，响应: generations=[[ChatGeneration(text='1 + 2 = 3', message=AIMessage(content='1 + 2 = 3', response_metadata={'id': 'msg_01D8Tt5FdtBk5gLTfBPm2tac', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}}, id='run-bb0dddd8-85f3-4e6b-8553-eaa79f859ef8-0'))]] llm_output={'id': 'msg_01D8Tt5FdtBk5gLTfBPm2tac', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}} run=None
链结束，输出: content='1 + 2 = 3' response_metadata={'id': 'msg_01D8Tt5FdtBk5gLTfBPm2tac', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}} id='run-bb0dddd8-85f3-4e6b-8553-eaa79f859ef8-0'
```


```output
AIMessage(content='1 + 2 = 3', response_metadata={'id': 'msg_01D8Tt5FdtBk5gLTfBPm2tac', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}}, id='run-bb0dddd8-85f3-4e6b-8553-eaa79f859ef8-0')
```


如果已经有与某个模块关联的回调，这些回调将在运行时传递的回调之外运行。

## 下一步

您现在已经了解了如何在运行时传递回调。

接下来，请查看本节中的其他操作指南，例如如何[将回调传递到模块构造函数中](/docs/how_to/custom_callbacks)。
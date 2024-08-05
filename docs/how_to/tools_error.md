---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/tools_error.ipynb
---

# 如何处理工具错误

:::info 前提条件

本指南假设您熟悉以下概念：
- [聊天模型](/docs/concepts/#chat-models)
- [LangChain 工具](/docs/concepts/#tools)
- [如何使用模型调用工具](/docs/how_to/tool_calling)

:::

使用 LLM 调用工具通常比单纯的提示更可靠，但并不完美。模型可能会尝试调用不存在的工具，或者未能返回与请求的模式匹配的参数。保持模式简单、减少一次传递的工具数量，以及使用良好的名称和描述等策略可以帮助降低这种风险，但并非万无一失。

本指南涵盖了一些将错误处理构建到您的链中的方法，以减轻这些失败模式。

## 设置

我们需要安装以下软件包：


```python
%pip install --upgrade --quiet langchain-core langchain-openai
```

如果您想在 [LangSmith](https://docs.smith.langchain.com/) 中跟踪您的运行，请取消注释并设置以下环境变量：


```python
import getpass
import os

# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 链

假设我们有以下（虚拟）工具和工具调用链。我们将故意使我们的工具复杂化，以试图让模型出错。

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm"/>


```python
# Define tool
from langchain_core.tools import tool


@tool
def complex_tool(int_arg: int, float_arg: float, dict_arg: dict) -> int:
    """Do something complex with a complex tool."""
    return int_arg * float_arg


llm_with_tools = llm.bind_tools(
    [complex_tool],
)

# Define chain
chain = llm_with_tools | (lambda msg: msg.tool_calls[0]["args"]) | complex_tool
```

我们可以看到，当我们尝试用相当明确的输入来调用这个链时，模型未能正确调用工具（它忘记了 `dict_arg` 参数）。

```python
chain.invoke(
    "use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"
)
```

```output
---------------------------------------------------------------------------
``````output
ValidationError                           Traceback (most recent call last)
``````output
Cell In[6], line 1
----> 1 chain.invoke(
      2     "use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"
      3 )
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/langchain_core/runnables/base.py:2572, in RunnableSequence.invoke(self, input, config, **kwargs)
   2570             input = step.invoke(input, config, **kwargs)
   2571         else:
-> 2572             input = step.invoke(input, config)
   2573 # finish the root run
   2574 except BaseException as e:
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/langchain_core/tools.py:380, in BaseTool.invoke(self, input, config, **kwargs)
    373 def invoke(
    374     self,
    375     input: Union[str, Dict],
    376     config: Optional[RunnableConfig] = None,
    377     **kwargs: Any,
    378 ) -> Any:
    379     config = ensure_config(config)
--> 380     return self.run(
    381         input,
    382         callbacks=config.get("callbacks"),
    383         tags=config.get("tags"),
    384         metadata=config.get("metadata"),
    385         run_name=config.get("run_name"),
    386         run_id=config.pop("run_id", None),
    387         config=config,
    388         **kwargs,
    389     )
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/langchain_core/tools.py:537, in BaseTool.run(self, tool_input, verbose, start_color, color, callbacks, tags, metadata, run_name, run_id, config, **kwargs)
    535 except ValidationError as e:
    536     if not self.handle_validation_error:
--> 537         raise e
    538     elif isinstance(self.handle_validation_error, bool):
    539         observation = "Tool input validation error"
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/langchain_core/tools.py:526, in BaseTool.run(self, tool_input, verbose, start_color, color, callbacks, tags, metadata, run_name, run_id, config, **kwargs)
    524 context = copy_context()
    525 context.run(_set_config_context, child_config)
--> 526 parsed_input = self._parse_input(tool_input)
    527 tool_args, tool_kwargs = self._to_args_and_kwargs(parsed_input)
    528 observation = (
    529     context.run(
    530         self._run, *tool_args, run_manager=run_manager, **tool_kwargs
   (...)
    533     else context.run(self._run, *tool_args, **tool_kwargs)
    534 )
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/langchain_core/tools.py:424, in BaseTool._parse_input(self, tool_input)
    422 else:
    423     if input_args is not None:
--> 424         result = input_args.parse_obj(tool_input)
    425         return {
    426             k: getattr(result, k)
    427             for k, v in result.dict().items()
    428             if k in tool_input
    429         }
    430 return tool_input
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/pydantic/main.py:526, in pydantic.main.BaseModel.parse_obj()
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/pydantic/main.py:341, in pydantic.main.BaseModel.__init__()
``````output
ValidationError: 1 validation error for complex_toolSchema
dict_arg
  field required (type=value_error.missing)
```

## 尝试/异常工具调用

处理错误最简单的方法是对工具调用步骤进行尝试/异常处理，并在出现错误时返回有用的信息：

```python
from typing import Any

from langchain_core.runnables import Runnable, RunnableConfig


def try_except_tool(tool_args: dict, config: RunnableConfig) -> Runnable:
    try:
        complex_tool.invoke(tool_args, config=config)
    except Exception as e:
        return f"Calling tool with arguments:\n\n{tool_args}\n\nraised the following error:\n\n{type(e)}: {e}"


chain = llm_with_tools | (lambda msg: msg.tool_calls[0]["args"]) | try_except_tool

print(
    chain.invoke(
        "use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"
    )
)
```
```output
Calling tool with arguments:

{'int_arg': 5, 'float_arg': 2.1}

raised the following error:

<class 'pydantic.error_wrappers.ValidationError'>: 1 validation error for complex_toolSchema
dict_arg
  field required (type=value_error.missing)
```

## 回退

在工具调用错误的情况下，我们也可以尝试回退到更好的模型。在这种情况下，我们将回退到一个相同的链，使用 `gpt-4-1106-preview` 代替 `gpt-3.5-turbo`。

```python
chain = llm_with_tools | (lambda msg: msg.tool_calls[0]["args"]) | complex_tool

better_model = ChatOpenAI(model="gpt-4-1106-preview", temperature=0).bind_tools(
    [complex_tool], tool_choice="complex_tool"
)

better_chain = better_model | (lambda msg: msg.tool_calls[0]["args"]) | complex_tool

chain_with_fallback = chain.with_fallbacks([better_chain])

chain_with_fallback.invoke(
    "use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"
)
```

```output
10.5
```

查看这个链条运行的 [LangSmith 跟踪](https://smith.langchain.com/public/00e91fc2-e1a4-4b0f-a82e-e6b3119d196c/r)，我们可以看到第一个链条调用按预期失败，而回退则成功。

## 重试与异常

为了更进一步，我们可以尝试自动重新运行链，并传入异常，这样模型可能能够纠正其行为：

```python
from langchain_core.messages import AIMessage, HumanMessage, ToolCall, ToolMessage
from langchain_core.prompts import ChatPromptTemplate


class CustomToolException(Exception):
    """自定义 LangChain 工具异常。"""

    def __init__(self, tool_call: ToolCall, exception: Exception) -> None:
        super().__init__()
        self.tool_call = tool_call
        self.exception = exception


def tool_custom_exception(msg: AIMessage, config: RunnableConfig) -> Runnable:
    try:
        return complex_tool.invoke(msg.tool_calls[0]["args"], config=config)
    except Exception as e:
        raise CustomToolException(msg.tool_calls[0], e)


def exception_to_messages(inputs: dict) -> dict:
    exception = inputs.pop("exception")

    # 将历史消息添加到原始输入中，以便模型知道它在上一个工具调用中犯了错误。
    messages = [
        AIMessage(content="", tool_calls=[exception.tool_call]),
        ToolMessage(
            tool_call_id=exception.tool_call["id"], content=str(exception.exception)
        ),
        HumanMessage(
            content="上一个工具调用引发了异常。尝试用更正的参数再次调用工具。不要重复错误。"
        ),
    ]
    inputs["last_output"] = messages
    return inputs


# 我们在提示中添加了一个 last_output MessagesPlaceholder，如果没有传入，
# 则不会影响提示，但如果需要，可以插入任意消息列表。
# 我们将在重试时使用它来插入错误消息。
prompt = ChatPromptTemplate.from_messages(
    [("human", "{input}"), ("placeholder", "{last_output}")]
)
chain = prompt | llm_with_tools | tool_custom_exception

# 如果初始链调用失败，我们将其重新运行，并将异常作为消息传入。
self_correcting_chain = chain.with_fallbacks(
    [exception_to_messages | chain], exception_key="exception"
)
```


```python
self_correcting_chain.invoke(
    {
        "input": "use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"
    }
)
```



```output
10.5
```


我们的链成功了！查看 [LangSmith 跟踪](https://smith.langchain.com/public/c11e804c-e14f-4059-bd09-64766f999c14/r)，我们可以看到，确实我们的初始链仍然失败，只有在重试时链才成功。

## 下一步

现在您已经了解了一些处理工具调用错误的策略。接下来，您可以学习更多关于如何使用工具的信息：

- 少量示例提示 [与工具](/docs/how_to/tools_few_shot/)
- 流式 [工具调用](/docs/how_to/tool_streaming/)
- 将 [运行时值传递给工具](/docs/how_to/tool_runtime)

您还可以查看一些工具调用的更具体用法：

- 从模型获取 [结构化输出](/docs/how_to/structured_output/)
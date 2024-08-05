---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/tool_configure.ipynb
---

# 如何从工具访问 RunnableConfig

:::info 前提条件

本指南假设您熟悉以下概念：

- [LangChain 工具](/docs/concepts/#tools)
- [自定义工具](/docs/how_to/custom_tools)
- [LangChain 表达语言 (LCEL)](/docs/concepts/#langchain-expression-language-lcel)
- [配置可运行行为](/docs/how_to/configure/)

:::

如果您有一个调用聊天模型、检索器或其他可运行对象的工具，您可能希望访问这些可运行对象的内部事件或使用额外属性进行配置。本指南将向您展示如何正确手动传递参数，以便您可以使用 `astream_events()` 方法来实现这一点。

工具是可运行对象，您可以在接口级别将它们视为任何其他可运行对象 - 您可以像往常一样对它们调用 `invoke()`、`batch()` 和 `stream()`。然而，在编写自定义工具时，您可能希望调用其他可运行对象，例如聊天模型或检索器。为了正确追踪和配置这些子调用，您需要手动访问并传递工具当前的 [`RunnableConfig`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.config.RunnableConfig.html) 对象。本指南将向您展示如何做到这一点的一些示例。

:::caution 兼容性

本指南要求 `langchain-core>=0.2.16`。

:::

## 通过参数类型推断

要从您的自定义工具访问活动配置对象，您需要在工具的签名中添加一个参数，类型为 `RunnableConfig`。当您调用工具时，LangChain 会检查工具的签名，查找类型为 `RunnableConfig` 的参数，如果存在，则用正确的值填充该参数。

**注意：** 参数的实际名称并不重要，只有类型才重要。

为了说明这一点，定义一个自定义工具，该工具接受两个参数 - 一个类型为字符串，另一个类型为 `RunnableConfig`：

```python
%pip install -qU langchain_core
```

```python
from langchain_core.runnables import RunnableConfig
from langchain_core.tools import tool


@tool
async def reverse_tool(text: str, special_config_param: RunnableConfig) -> str:
    """一个测试工具，将输入文本与可配置参数结合起来。"""
    return (text + special_config_param["configurable"]["additional_field"])[::-1]
```

然后，如果我们使用包含 `configurable` 字段的 `config` 调用该工具，我们可以看到 `additional_field` 被正确传递：

```python
await reverse_tool.ainvoke(
    {"text": "abc"}, config={"configurable": {"additional_field": "123"}}
)
```

```output
'321cba'
```

## 下一步

您现在已经了解了如何在工具中配置和流式传输事件。接下来，请查看以下指南以获取更多关于使用工具的信息：

- [从自定义工具中的子运行流式传输事件](/docs/how_to/tool_stream_events/)
- 将 [工具结果传递回模型](/docs/how_to/tool_results_pass_to_model)

您还可以查看一些工具调用的更具体用法：

- 构建 [使用工具的链和代理](/docs/how_to#tools)
- 从模型获取 [结构化输出](/docs/how_to/structured_output/)
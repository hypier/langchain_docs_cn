---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/custom_tools.ipynb
---

# 如何创建工具

在构建代理时，您需要为其提供一个可以使用的 `Tool` 列表。除了被调用的实际函数，Tool 由几个组件组成：

| 属性            | 类型                      | 描述                                                                                                         |
|-----------------|---------------------------|--------------------------------------------------------------------------------------------------------------|
| name          | str                     | 在提供给 LLM 或代理的一组工具中必须是唯一的。                                                               |
| description   | str                     | 描述工具的功能。被 LLM 或代理用作上下文。                                                                   |
| args_schema   | Pydantic BaseModel      | 可选但推荐，可以用于提供更多信息（例如，少量示例）或对预期参数进行验证。                                    |
| return_direct   | boolean      | 仅对代理相关。当为 True 时，在调用给定工具后，代理将停止并直接将结果返回给用户。                          |

LangChain 支持从以下方式创建工具：

1. 函数；
2. LangChain [Runnables](/docs/concepts#runnable-interface)；
3. 通过从 [BaseTool](https://api.python.langchain.com/en/latest/tools/langchain_core.tools.BaseTool.html) 子类化 -- 这是最灵活的方法，提供了最大的控制权，但需要更多的努力和代码。

从函数创建工具可能对大多数用例来说足够，并且可以通过简单的 [@tool 装饰器](https://api.python.langchain.com/en/latest/tools/langchain_core.tools.tool.html#langchain_core.tools.tool) 来完成。如果需要更多配置，例如同时指定同步和异步实现，也可以使用 [StructuredTool.from_function](https://api.python.langchain.com/en/latest/tools/langchain_core.tools.StructuredTool.html#langchain_core.tools.StructuredTool.from_function) 类方法。

在本指南中，我们提供了这些方法的概述。

:::tip

如果工具具有精心选择的名称、描述和 JSON 模式，模型的表现将更好。
:::

## 从函数创建工具

### @tool 装饰器

这个 `@tool` 装饰器是定义自定义工具的最简单方法。默认情况下，装饰器使用函数名称作为工具名称，但可以通过将字符串作为第一个参数传递来覆盖此设置。此外，装饰器将使用函数的文档字符串作为工具的描述，因此必须提供文档字符串。

```python
from langchain_core.tools import tool


@tool
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


# 让我们检查与该工具相关的一些属性。
print(multiply.name)
print(multiply.description)
print(multiply.args)
```
```output
multiply
Multiply two numbers.
{'a': {'title': 'A', 'type': 'integer'}, 'b': {'title': 'B', 'type': 'integer'}}
```
或者创建一个 **async** 实现，如下所示：

```python
from langchain_core.tools import tool


@tool
async def amultiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b
```

注意，`@tool` 支持解析注解、嵌套模式和其他特性：

```python
from typing import Annotated, List


@tool
def multiply_by_max(
    a: Annotated[str, "scale factor"],
    b: Annotated[List[int], "list of ints over which to take maximum"],
) -> int:
    """Multiply a by the maximum of b."""
    return a * max(b)


multiply_by_max.args_schema.schema()
```

```output
{'title': 'multiply_by_maxSchema',
 'description': 'Multiply a by the maximum of b.',
 'type': 'object',
 'properties': {'a': {'title': 'A',
   'description': 'scale factor',
   'type': 'string'},
  'b': {'title': 'B',
   'description': 'list of ints over which to take maximum',
   'type': 'array',
   'items': {'type': 'integer'}}},
 'required': ['a', 'b']}
```

您还可以通过将工具名称和 JSON 参数传递给工具装饰器来进行自定义。

```python
from langchain.pydantic_v1 import BaseModel, Field


class CalculatorInput(BaseModel):
    a: int = Field(description="first number")
    b: int = Field(description="second number")


@tool("multiplication-tool", args_schema=CalculatorInput, return_direct=True)
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


# 让我们检查与该工具相关的一些属性。
print(multiply.name)
print(multiply.description)
print(multiply.args)
print(multiply.return_direct)
```
```output
multiplication-tool
Multiply two numbers.
{'a': {'title': 'A', 'description': 'first number', 'type': 'integer'}, 'b': {'title': 'B', 'description': 'second number', 'type': 'integer'}}
True
```
#### 文档字符串解析

`@tool` 可以选择解析 [Google 风格的文档字符串](https://google.github.io/styleguide/pyguide.html#383-functions-and-methods)，并将文档字符串组件（如参数描述）与工具模式的相关部分关联。要切换此行为，请指定 `parse_docstring`：

```python
@tool(parse_docstring=True)
def foo(bar: str, baz: int) -> str:
    """The foo.

    Args:
        bar: The bar.
        baz: The baz.
    """
    return bar


foo.args_schema.schema()
```

```output
{'title': 'fooSchema',
 'description': 'The foo.',
 'type': 'object',
 'properties': {'bar': {'title': 'Bar',
   'description': 'The bar.',
   'type': 'string'},
  'baz': {'title': 'Baz', 'description': 'The baz.', 'type': 'integer'}},
 'required': ['bar', 'baz']}
```

:::caution
默认情况下，如果文档字符串无法正确解析，`@tool(parse_docstring=True)` 将引发 `ValueError`。有关详细信息和示例，请参见 [API 参考](https://api.python.langchain.com/en/latest/tools/langchain_core.tools.tool.html)。
:::

### StructuredTool

`StructuredTool.from_function` 类方法提供了比 `@tool` 装饰器更多的配置选项，而不需要太多额外的代码。

```python
from langchain_core.tools import StructuredTool


def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


async def amultiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


calculator = StructuredTool.from_function(func=multiply, coroutine=amultiply)

print(calculator.invoke({"a": 2, "b": 3}))
print(await calculator.ainvoke({"a": 2, "b": 5}))
```
```output
6
10
```
要进行配置：

```python
class CalculatorInput(BaseModel):
    a: int = Field(description="first number")
    b: int = Field(description="second number")


def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


calculator = StructuredTool.from_function(
    func=multiply,
    name="Calculator",
    description="multiply numbers",
    args_schema=CalculatorInput,
    return_direct=True,
    # coroutine= ... <- you can specify an async method if desired as well
)

print(calculator.invoke({"a": 2, "b": 3}))
print(calculator.name)
print(calculator.description)
print(calculator.args)
```
```output
6
Calculator
multiply numbers
{'a': {'title': 'A', 'description': 'first number', 'type': 'integer'}, 'b': {'title': 'B', 'description': 'second number', 'type': 'integer'}}
```

## 从 Runnables 创建工具

LangChain [Runnables](/docs/concepts#runnable-interface) 接受字符串或 `dict` 输入，可以使用 [as_tool](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.as_tool) 方法转换为工具，该方法允许指定名称、描述和参数的附加模式信息。

示例用法：

```python
from langchain_core.language_models import GenericFakeChatModel
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [("human", "Hello. Please respond in the style of {answer_style}.")]
)

# Placeholder LLM
llm = GenericFakeChatModel(messages=iter(["hello matey"]))

chain = prompt | llm | StrOutputParser()

as_tool = chain.as_tool(
    name="Style responder", description="Description of when to use tool."
)
as_tool.args
```

```output
{'answer_style': {'title': 'Answer Style', 'type': 'string'}}
```

有关更多详细信息，请参见 [此指南](/docs/how_to/convert_runnable_to_tool)。

## 子类 BaseTool

您可以通过从 `BaseTool` 子类化来定义自定义工具。这提供了对工具定义的最大控制，但需要编写更多代码。

```python
from typing import Optional, Type

from langchain.pydantic_v1 import BaseModel
from langchain_core.callbacks import (
    AsyncCallbackManagerForToolRun,
    CallbackManagerForToolRun,
)
from langchain_core.tools import BaseTool


class CalculatorInput(BaseModel):
    a: int = Field(description="第一个数字")
    b: int = Field(description="第二个数字")


class CustomCalculatorTool(BaseTool):
    name = "Calculator"
    description = "用于回答数学问题时非常有用"
    args_schema: Type[BaseModel] = CalculatorInput
    return_direct: bool = True

    def _run(
        self, a: int, b: int, run_manager: Optional[CallbackManagerForToolRun] = None
    ) -> str:
        """使用工具。"""
        return a * b

    async def _arun(
        self,
        a: int,
        b: int,
        run_manager: Optional[AsyncCallbackManagerForToolRun] = None,
    ) -> str:
        """异步使用工具。"""
        # 如果计算很简单，您可以像下面这样委托给同步实现。
        # 如果同步计算很复杂，您应该删除整个 _arun 方法。
        # LangChain 将自动提供更好的实现，确保不会阻塞其他异步代码。
        return self._run(a, b, run_manager=run_manager.get_sync())
```

```python
multiply = CustomCalculatorTool()
print(multiply.name)
print(multiply.description)
print(multiply.args)
print(multiply.return_direct)

print(multiply.invoke({"a": 2, "b": 3}))
print(await multiply.ainvoke({"a": 2, "b": 3}))
```
```output
Calculator
用于回答数学问题时非常有用
{'a': {'title': 'A', 'description': '第一个数字', 'type': 'integer'}, 'b': {'title': 'B', 'description': '第二个数字', 'type': 'integer'}}
True
6
6
```

## 如何创建异步工具

LangChain Tools 实现了 [Runnable 接口 🏃](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html)。

所有 Runnables 都暴露了 `invoke` 和 `ainvoke` 方法（以及其他方法如 `batch`、`abatch`、`astream` 等）。

因此，即使您只提供工具的 `sync` 实现，您仍然可以使用 `ainvoke` 接口，但有一些重要事项需要了解：

* LangChain 默认提供异步实现，假设函数的计算开销较大，因此它将把执行委托给另一个线程。
* 如果您在异步代码库中工作，应该创建异步工具而不是同步工具，以避免由于线程带来的小开销。
* 如果您需要同步和异步实现，请使用 `StructuredTool.from_function` 或从 `BaseTool` 子类化。
* 如果同时实现同步和异步，并且同步代码运行速度较快，请覆盖默认的 LangChain 异步实现并直接调用同步代码。
* 您不能也不应该在异步工具上使用同步 `invoke`。

```python
from langchain_core.tools import StructuredTool


def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


calculator = StructuredTool.from_function(func=multiply)

print(calculator.invoke({"a": 2, "b": 3}))
print(
    await calculator.ainvoke({"a": 2, "b": 5})
)  # 使用默认的 LangChain 异步实现会产生小开销
```
```output
6
10
```

```python
from langchain_core.tools import StructuredTool


def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


async def amultiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


calculator = StructuredTool.from_function(func=multiply, coroutine=amultiply)

print(calculator.invoke({"a": 2, "b": 3}))
print(
    await calculator.ainvoke({"a": 2, "b": 5})
)  # 使用提供的 amultiply，没有额外开销
```
```output
6
10
```
在仅提供异步定义时，您不应该也不能使用 `.invoke`。

```python
@tool
async def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


try:
    multiply.invoke({"a": 2, "b": 3})
except NotImplementedError:
    print("引发未实现错误。您不应该这样做。")
```
```output
引发未实现错误。您不应该这样做。
```

## 处理工具错误

如果您正在使用带有代理的工具，您可能需要一个错误处理策略，以便代理能够从错误中恢复并继续执行。

一个简单的策略是在工具内部抛出 `ToolException`，并使用 `handle_tool_error` 指定一个错误处理程序。

当指定错误处理程序时，异常将被捕获，错误处理程序将决定从工具返回哪个输出。

您可以将 `handle_tool_error` 设置为 `True`、字符串值或函数。如果是函数，该函数应接受一个 `ToolException` 作为参数并返回一个值。

请注意，仅仅抛出 `ToolException` 是无效的。您需要首先设置工具的 `handle_tool_error`，因为其默认值为 `False`。


```python
from langchain_core.tools import ToolException


def get_weather(city: str) -> int:
    """获取指定城市的天气。"""
    raise ToolException(f"错误：没有名为 {city} 的城市。")
```

这是一个使用默认 `handle_tool_error=True` 行为的示例。


```python
get_weather_tool = StructuredTool.from_function(
    func=get_weather,
    handle_tool_error=True,
)

get_weather_tool.invoke({"city": "foobar"})
```



```output
'错误：没有名为 foobar 的城市。'
```


我们可以将 `handle_tool_error` 设置为一个字符串，该字符串将始终被返回。


```python
get_weather_tool = StructuredTool.from_function(
    func=get_weather,
    handle_tool_error="没有这样的城市，但那里可能在 0K 以上！",
)

get_weather_tool.invoke({"city": "foobar"})
```



```output
"没有这样的城市，但那里可能在 0K 以上！"
```


使用函数处理错误：


```python
def _handle_error(error: ToolException) -> str:
    return f"工具执行期间发生了以下错误：`{error.args[0]}`"


get_weather_tool = StructuredTool.from_function(
    func=get_weather,
    handle_tool_error=_handle_error,
)

get_weather_tool.invoke({"city": "foobar"})
```



```output
'工具执行期间发生了以下错误：`错误：没有名为 foobar 的城市。`'
```

## 返回工具执行的工件

有时我们希望将工具执行的工件提供给链或代理中的下游组件，但又不希望将其暴露给模型本身。例如，如果一个工具返回自定义对象，如文档，我们可能希望将一些视图或元数据传递给模型，而不将原始输出传递给模型。同时，我们可能希望能够在其他地方访问这个完整的输出，例如在下游工具中。

Tool 和 [ToolMessage](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.tool.ToolMessage.html) 接口使得能够区分工具输出中用于模型的部分（这是 ToolMessage.content）和用于模型外部使用的部分（ToolMessage.artifact）。

:::info 需要 ``langchain-core >= 0.2.19``

此功能是在 ``langchain-core == 0.2.19`` 中添加的。请确保您的包是最新的。

:::

如果我们希望工具区分消息内容和其他工件，我们需要在定义工具时指定 `response_format="content_and_artifact"`，并确保返回一个元组 (content, artifact)：

```python
import random
from typing import List, Tuple

from langchain_core.tools import tool


@tool(response_format="content_and_artifact")
def generate_random_ints(min: int, max: int, size: int) -> Tuple[str, List[int]]:
    """生成在 [min, max] 范围内的 size 个随机整数。"""
    array = [random.randint(min, max) for _ in range(size)]
    content = f"成功生成了 {size} 个随机整数的数组，范围在 [{min}, {max}] 之间。"
    return content, array
```

如果我们直接使用工具参数调用工具，我们将仅返回输出的内容部分：

```python
generate_random_ints.invoke({"min": 0, "max": 9, "size": 10})
```

```output
'成功生成了 10 个随机整数的数组，范围在 [0, 9] 之间。'
```

如果我们使用 ToolCall 调用工具（例如由工具调用模型生成的工具），我们将收到一个包含工具生成的内容和工件的 ToolMessage：

```python
generate_random_ints.invoke(
    {
        "name": "generate_random_ints",
        "args": {"min": 0, "max": 9, "size": 10},
        "id": "123",  # 必需
        "type": "tool_call",  # 必需
    }
)
```

```output
ToolMessage(content='成功生成了 10 个随机整数的数组，范围在 [0, 9] 之间。', name='generate_random_ints', tool_call_id='123', artifact=[1, 4, 2, 5, 3, 9, 0, 4, 7, 7])
```

当我们从 BaseTool 子类化时也可以这样做：

```python
from langchain_core.tools import BaseTool


class GenerateRandomFloats(BaseTool):
    name: str = "generate_random_floats"
    description: str = "生成在 [min, max] 范围内的 size 个随机浮点数。"
    response_format: str = "content_and_artifact"

    ndigits: int = 2

    def _run(self, min: float, max: float, size: int) -> Tuple[str, List[float]]:
        range_ = max - min
        array = [
            round(min + (range_ * random.random()), ndigits=self.ndigits)
            for _ in range(size)
        ]
        content = f"生成了 {size} 个浮点数，范围在 [{min}, {max}] 之间，四舍五入到 {self.ndigits} 位小数。"
        return content, array

    # 可选定义等效的异步方法

    # async def _arun(self, min: float, max: float, size: int) -> Tuple[str, List[float]]:
    #     ...
```

```python
rand_gen = GenerateRandomFloats(ndigits=4)

rand_gen.invoke(
    {
        "name": "generate_random_floats",
        "args": {"min": 0.1, "max": 3.3333, "size": 3},
        "id": "123",
        "type": "tool_call",
    }
)
```

```output
ToolMessage(content='生成了 3 个浮点数，范围在 [0.1, 3.3333] 之间，四舍五入到 4 位小数。', name='generate_random_floats', tool_call_id='123', artifact=[1.4277, 0.7578, 2.4871])
```
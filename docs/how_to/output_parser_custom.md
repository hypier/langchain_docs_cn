---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/output_parser_custom.ipynb
---

# 如何创建自定义输出解析器

在某些情况下，您可能希望实现一个自定义解析器，将模型输出结构化为自定义格式。

实现自定义解析器有两种方法：

1. 使用 LCEL 中的 `RunnableLambda` 或 `RunnableGenerator` -- 我们强烈推荐这种方法用于大多数用例
2. 通过从基础类之一继承来进行输出解析 -- 这是比较困难的实现方式

这两种方法之间的区别主要是表面的，主要体现在触发的回调（例如，`on_chain_start` 与 `on_parser_start`）以及在像 LangSmith 这样的追踪平台中可视化 `RunnableLambda` 与解析器的方式。

## 可运行的 Lambda 和生成器

推荐的解析方式是使用 **可运行的 Lambda** 和 **可运行的生成器**！

在这里，我们将进行一个简单的解析，它会反转模型输出的大小写。

例如，如果模型输出：“Meow”，解析器将生成“mEOW”。

```python
from typing import Iterable

from langchain_anthropic.chat_models import ChatAnthropic
from langchain_core.messages import AIMessage, AIMessageChunk

model = ChatAnthropic(model_name="claude-2.1")


def parse(ai_message: AIMessage) -> str:
    """Parse the AI message."""
    return ai_message.content.swapcase()


chain = model | parse
chain.invoke("hello")
```

```output
'hELLO!'
```

:::tip

LCEL 在使用 `|` 语法组合时，会自动将函数 `parse` 升级为 `RunnableLambda(parse)`。

如果你不喜欢这样，你可以手动导入 `RunnableLambda`，然后运行 `parse = RunnableLambda(parse)`。
:::

流式传输有效吗？

```python
for chunk in chain.stream("tell me about yourself in one sentence"):
    print(chunk, end="|", flush=True)
```
```output
i'M cLAUDE, AN ai ASSISTANT CREATED BY aNTHROPIC TO BE HELPFUL, HARMLESS, AND HONEST.|
```
不，它无效，因为解析器在解析输出之前会聚合输入。

如果我们想实现一个流式解析器，可以让解析器接受输入的可迭代对象，并在结果可用时生成结果。

```python
from langchain_core.runnables import RunnableGenerator


def streaming_parse(chunks: Iterable[AIMessageChunk]) -> Iterable[str]:
    for chunk in chunks:
        yield chunk.content.swapcase()


streaming_parse = RunnableGenerator(streaming_parse)
```

:::important

请将流式解析器包装在 `RunnableGenerator` 中，因为我们可能会停止使用 `|` 语法自动升级它。
:::

```python
chain = model | streaming_parse
chain.invoke("hello")
```

```output
'hELLO!'
```

让我们确认流式传输有效！

```python
for chunk in chain.stream("tell me about yourself in one sentence"):
    print(chunk, end="|", flush=True)
```
```output
i|'M| cLAUDE|,| AN| ai| ASSISTANT| CREATED| BY| aN|THROP|IC| TO| BE| HELPFUL|,| HARMLESS|,| AND| HONEST|.|
```

## 从解析基类继承

实现解析器的另一种方法是从 `BaseOutputParser`、`BaseGenerationOutputParser` 或其他基解析器继承，具体取决于您的需求。

一般来说，我们 **不** 推荐这种方法用于大多数用例，因为这会导致需要编写更多代码而没有显著的好处。

最简单的输出解析器类型扩展了 `BaseOutputParser` 类，并必须实现以下方法：

* `parse`：接受模型的字符串输出并进行解析
* （可选）`_type`：识别解析器的名称。

当聊天模型或 LLM 的输出格式不正确时，可以抛出 `OutputParserException` 来表示解析因输入错误而失败。使用此异常可以让使用解析器的代码以一致的方式处理异常。

:::tip 解析器是可运行的！ 🏃

因为 `BaseOutputParser` 实现了 `Runnable` 接口，所以您以这种方式创建的任何自定义解析器都将成为有效的 LangChain 可运行对象，并将受益于自动异步支持、批量接口、日志支持等。
:::

### 简单解析器

这是一个简单的解析器，可以解析**字符串**表示的布尔值（例如，`YES`或`NO`），并将其转换为相应的`boolean`类型。

```python
from langchain_core.exceptions import OutputParserException
from langchain_core.output_parsers import BaseOutputParser


# [bool]描述了一个通用的参数化。
# 它基本上指示解析的返回类型
# 在这种情况下，返回类型是True或False
class BooleanOutputParser(BaseOutputParser[bool]):
    """自定义布尔解析器。"""

    true_val: str = "YES"
    false_val: str = "NO"

    def parse(self, text: str) -> bool:
        cleaned_text = text.strip().upper()
        if cleaned_text not in (self.true_val.upper(), self.false_val.upper()):
            raise OutputParserException(
                f"BooleanOutputParser expected output value to either be "
                f"{self.true_val} or {self.false_val} (case-insensitive). "
                f"Received {cleaned_text}."
            )
        return cleaned_text == self.true_val.upper()

    @property
    def _type(self) -> str:
        return "boolean_output_parser"
```

```python
parser = BooleanOutputParser()
parser.invoke("YES")
```

```output
True
```

```python
try:
    parser.invoke("MEOW")
except Exception as e:
    print(f"Triggered an exception of type: {type(e)}")
```
```output
Triggered an exception of type: <class 'langchain_core.exceptions.OutputParserException'>
```
让我们测试更改参数化

```python
parser = BooleanOutputParser(true_val="OKAY")
parser.invoke("OKAY")
```

```output
True
```

让我们确认其他LCEL方法是否存在

```python
parser.batch(["OKAY", "NO"])
```

```output
[True, False]
```

```python
await parser.abatch(["OKAY", "NO"])
```

```output
[True, False]
```

```python
from langchain_anthropic.chat_models import ChatAnthropic

anthropic = ChatAnthropic(model_name="claude-2.1")
anthropic.invoke("say OKAY or NO")
```

```output
AIMessage(content='OKAY')
```

让我们测试一下我们的解析器是否有效！

```python
chain = anthropic | parser
chain.invoke("say OKAY or NO")
```

```output
True
```

:::note
该解析器可以处理来自LLM的输出（字符串）或来自聊天模型的输出（`AIMessage`）!
:::

### 解析原始模型输出

有时，模型输出中除了原始文本之外还有额外的重要元数据。一个例子是工具调用，其中打算传递给被调用函数的参数以单独的属性返回。如果您需要这种更细粒度的控制，您可以改为子类化 `BaseGenerationOutputParser` 类。

此类需要一个单一的方法 `parse_result`。该方法接受原始模型输出（例如，`Generation` 或 `ChatGeneration` 的列表）并返回解析后的输出。

支持 `Generation` 和 `ChatGeneration` 使解析器能够与常规 LLM 以及聊天模型一起工作。

```python
from typing import List

from langchain_core.exceptions import OutputParserException
from langchain_core.messages import AIMessage
from langchain_core.output_parsers import BaseGenerationOutputParser
from langchain_core.outputs import ChatGeneration, Generation


class StrInvertCase(BaseGenerationOutputParser[str]):
    """一个示例解析器，它反转消息中字符的大小写。

    这是一个仅用于演示目的的示例解析，旨在使示例尽可能简单。
    """

    def parse_result(self, result: List[Generation], *, partial: bool = False) -> str:
        """将模型生成的列表解析为特定格式。

        参数：
            result: 要解析的生成列表。假设这些生成是单个模型输入的不同候选输出。
                许多解析器假设只传入了一个生成。
                我们将对此进行断言。
            partial: 是否允许部分结果。这用于支持流式处理的解析器。
        """
        if len(result) != 1:
            raise NotImplementedError(
                "此输出解析器只能与单个生成一起使用。"
            )
        generation = result[0]
        if not isinstance(generation, ChatGeneration):
            # 说明这个仅适用于聊天生成
            raise OutputParserException(
                "此输出解析器只能与聊天生成一起使用。"
            )
        return generation.message.content.swapcase()


chain = anthropic | StrInvertCase()
```

让我们来看看新的解析器！它应该反转模型的输出。

```python
chain.invoke("Tell me a short sentence about yourself")
```

```output
'hELLO! mY NAME IS cLAUDE.'
```
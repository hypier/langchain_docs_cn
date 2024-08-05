---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/prompts_partial.ipynb
sidebar_position: 4
---

# 如何部分格式化提示模板

:::info 前提条件

本指南假设您对以下概念有所了解：
- [提示模板](/docs/concepts/#prompt-templates)

:::

就像将参数部分绑定到函数一样，部分格式化提示模板也是有意义的——例如，传入所需值的一个子集，以创建一个新的提示模板，该模板只期望其余值的子集。

LangChain 通过两种方式支持这一点：

1. 使用字符串值进行部分格式化。
2. 使用返回字符串值的函数进行部分格式化。

在下面的示例中，我们将讨论这两种用例的动机以及如何在 LangChain 中实现。

## 部分字符串

一个常见的使用场景是，如果你在提示中先获取到某些变量，可以对提示模板进行部分处理。例如，假设你有一个提示模板，需要两个变量，`foo` 和 `baz`。如果你在链的早期阶段获取到 `foo` 的值，但在后期获取到 `baz` 的值，那么将两个变量都传递整个链会很不方便。相反，你可以使用 `foo` 的值对提示模板进行部分处理，然后将部分处理后的提示模板传递下去，仅使用它。下面是一个示例：

```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template("{foo}{bar}")
partial_prompt = prompt.partial(foo="foo")
print(partial_prompt.format(bar="baz"))
```
```output
foobaz
```
你也可以直接用部分变量初始化提示。

```python
prompt = PromptTemplate(
    template="{foo}{bar}", input_variables=["bar"], partial_variables={"foo": "foo"}
)
print(prompt.format(bar="baz"))
```
```output
foobaz
```

## 使用函数的部分

另一个常见的用法是与函数进行部分应用。使用场景是当你有一个变量，你知道你总是想以一种常见的方式获取它。一个典型的例子就是日期或时间。想象一下，你有一个提示，你总是想要当前的日期。你不能在提示中硬编码它，并且将它与其他输入变量一起传递很不方便。在这种情况下，能够使用一个总是返回当前日期的函数来部分应用提示是非常方便的。



```python
from datetime import datetime


def _get_datetime():
    now = datetime.now()
    return now.strftime("%m/%d/%Y, %H:%M:%S")


prompt = PromptTemplate(
    template="Tell me a {adjective} joke about the day {date}",
    input_variables=["adjective", "date"],
)
partial_prompt = prompt.partial(date=_get_datetime)
print(partial_prompt.format(adjective="funny"))
```
```output
Tell me a funny joke about the day 04/21/2024, 19:43:57
```
你也可以直接用部分变量初始化提示，这在这个工作流程中通常更有意义。



```python
prompt = PromptTemplate(
    template="Tell me a {adjective} joke about the day {date}",
    input_variables=["adjective"],
    partial_variables={"date": _get_datetime},
)
print(prompt.format(adjective="funny"))
```
```output
Tell me a funny joke about the day 04/21/2024, 19:43:57
```

## 下一步

您现在已经学习了如何将变量部分应用于您的提示模板。

接下来，请查看本节中关于提示模板的其他操作指南，例如 [向您的提示模板添加少量示例](/docs/how_to/few_shot_examples_chat)。
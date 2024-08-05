---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/output_parser_json.ipynb
---

# 如何解析 JSON 输出

:::info 前提条件

本指南假设您对以下概念有一定了解：
- [聊天模型](/docs/concepts/#chat-models)
- [输出解析器](/docs/concepts/#output-parsers)
- [提示模板](/docs/concepts/#prompt-templates)
- [结构化输出](/docs/how_to/structured_output)
- [将可运行项串联在一起](/docs/how_to/sequence/)

:::

虽然一些模型提供者支持 [内置的结构化输出返回方式](/docs/how_to/structured_output)，但并非所有提供者都支持。我们可以使用输出解析器帮助用户通过提示指定任意 JSON 架构，查询模型以获取符合该架构的输出，最后将该架构解析为 JSON。

:::note
请记住，大型语言模型是泄漏的抽象！您必须使用具有足够容量的 LLM 来生成格式良好的 JSON。
:::

[`JsonOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.json.JsonOutputParser.html) 是一个内置选项，可以用于提示和解析 JSON 输出。虽然它的功能与 [`PydanticOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.pydantic.PydanticOutputParser.html) 相似，但它还支持流式返回部分 JSON 对象。

以下是如何与 [Pydantic](https://docs.pydantic.dev/) 一起使用的示例，以方便声明期望的架构：

```python
%pip install -qU langchain langchain-openai

import os
from getpass import getpass

os.environ["OPENAI_API_KEY"] = getpass()
```

```python
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI

model = ChatOpenAI(temperature=0)


# 定义您所需的数据结构。
class Joke(BaseModel):
    setup: str = Field(description="设置笑话的问题")
    punchline: str = Field(description="解决笑话的答案")


# 以及一个旨在提示语言模型填充数据结构的查询。
joke_query = "告诉我一个笑话。"

# 设置解析器 + 将指令注入提示模板。
parser = JsonOutputParser(pydantic_object=Joke)

prompt = PromptTemplate(
    template="回答用户查询。\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)

chain = prompt | model | parser

chain.invoke({"query": joke_query})
```

```output
{'setup': "为什么自行车无法独自站立？",
 'punchline': '因为它太累了！'}
```

请注意，我们将 `format_instructions` 从解析器直接传递到提示中。您可以并且应该尝试在提示的其他部分添加自己的格式提示，以增强或替换默认指令：

```python
parser.get_format_instructions()
```

```output
'输出应格式化为符合以下 JSON 架构的 JSON 实例。\n\n作为示例，对于架构 {"properties": {"foo": {"title": "Foo", "description": "字符串列表", "type": "array", "items": {"type": "string"}}}, "required": ["foo"]}\n对象 {"foo": ["bar", "baz"]} 是该架构的格式良好的实例。对象 {"properties": {"foo": ["bar", "baz"]}} 不是格式良好的。\n\n以下是输出架构：\n```\n{"properties": {"setup": {"title": "设置", "description": "设置笑话的问题", "type": "string"}, "punchline": {"title": "答案", "description": "解决笑话的答案", "type": "string"}}, "required": ["setup", "punchline"]}\n```'
```

## 流式处理

如上所述，`JsonOutputParser` 和 `PydanticOutputParser` 之间的一个关键区别是 `JsonOutputParser` 输出解析器支持流式处理部分数据块。以下是示例：

```python
for s in chain.stream({"query": joke_query}):
    print(s)
```
```output
{}
{'setup': ''}
{'setup': 'Why'}
{'setup': 'Why couldn'}
{'setup': "Why couldn't"}
{'setup': "Why couldn't the"}
{'setup': "Why couldn't the bicycle"}
{'setup': "Why couldn't the bicycle stand"}
{'setup': "Why couldn't the bicycle stand up"}
{'setup': "Why couldn't the bicycle stand up by"}
{'setup': "Why couldn't the bicycle stand up by itself"}
{'setup': "Why couldn't the bicycle stand up by itself?"}
{'setup': "Why couldn't the bicycle stand up by itself?", 'punchline': ''}
{'setup': "Why couldn't the bicycle stand up by itself?", 'punchline': 'Because'}
{'setup': "Why couldn't the bicycle stand up by itself?", 'punchline': 'Because it'}
{'setup': "Why couldn't the bicycle stand up by itself?", 'punchline': 'Because it was'}
{'setup': "Why couldn't the bicycle stand up by itself?", 'punchline': 'Because it was two'}
{'setup': "Why couldn't the bicycle stand up by itself?", 'punchline': 'Because it was two tired'}
{'setup': "Why couldn't the bicycle stand up by itself?", 'punchline': 'Because it was two tired!'}
```

## 无需 Pydantic

您也可以在不使用 Pydantic 的情况下使用 `JsonOutputParser`。这将提示模型返回 JSON，但并未提供关于模式应如何的具体信息。

```python
joke_query = "Tell me a joke."

parser = JsonOutputParser()

prompt = PromptTemplate(
    template="Answer the user query.\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)

chain = prompt | model | parser

chain.invoke({"query": joke_query})
```

```output
{'response': "Sure! Here's a joke for you: Why couldn't the bicycle stand up by itself? Because it was two tired!"}
```

## 下一步

您现在已经学习了一种提示模型返回结构化 JSON 的方法。接下来，请查看 [获取结构化输出的更广泛指南](/docs/how_to/structured_output) 以了解其他技术。
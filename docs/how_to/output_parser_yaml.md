---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/output_parser_yaml.ipynb
---

# 如何解析 YAML 输出

:::info 前提条件

本指南假设您熟悉以下概念：
- [聊天模型](/docs/concepts/#chat-models)
- [输出解析器](/docs/concepts/#output-parsers)
- [提示模板](/docs/concepts/#prompt-templates)
- [结构化输出](/docs/how_to/structured_output)
- [将可运行对象串联在一起](/docs/how_to/sequence/)

:::

不同提供商的 LLM 可能在特定数据的训练上具有不同的优势。这也意味着某些模型在生成 JSON 以外格式的输出时可能“更好”且更可靠。

此输出解析器允许用户指定任意模式，并查询 LLM 以获取符合该模式的输出，使用 YAML 格式化其响应。

:::note
请记住，大型语言模型是泄漏的抽象！您必须使用具有足够容量的 LLM 来生成格式良好的 YAML。
:::



```python
%pip install -qU langchain langchain-openai

import os
from getpass import getpass

os.environ["OPENAI_API_KEY"] = getpass()
```

我们使用 [Pydantic](https://docs.pydantic.dev) 和 [`YamlOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain.output_parsers.yaml.YamlOutputParser.html#langchain.output_parsers.yaml.YamlOutputParser) 来声明我们的数据模型，并为模型提供更多上下文，以便它知道应该生成什么类型的 YAML：


```python
from langchain.output_parsers import YamlOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI


# 定义您所需的数据结构。
class Joke(BaseModel):
    setup: str = Field(description="设置笑话的问题")
    punchline: str = Field(description="解决笑话的答案")


model = ChatOpenAI(temperature=0)

# 以及一个旨在提示语言模型填充数据结构的查询。
joke_query = "给我讲个笑话。"

# 设置一个解析器 + 将指令注入到提示模板中。
parser = YamlOutputParser(pydantic_object=Joke)

prompt = PromptTemplate(
    template="回答用户查询。\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)

chain = prompt | model | parser

chain.invoke({"query": joke_query})
```



```output
Joke(setup="为什么自行车找不到回家的路？", punchline='因为它失去了方向感！')
```


解析器将自动解析输出 YAML 并创建一个包含数据的 Pydantic 模型。我们可以看到解析器的 `format_instructions`，这些指令会被添加到提示中：


```python
parser.get_format_instructions()
```



```output
'输出应格式化为符合以下给定 JSON 模式的 YAML 实例。\n\n# 示例\n## 模式\n```\n{"title": "Players", "description": "一组玩家", "type": "array", "items": {"$ref": "#/definitions/Player"}, "definitions": {"Player": {"title": "Player", "type": "object", "properties": {"name": {"title": "Name", "description": "玩家姓名", "type": "string"}, "avg": {"title": "Avg", "description": "击球平均值", "type": "number"}}, "required": ["name", "avg"]}}}\n```\n## 格式良好的实例\n```\n- name: John Doe\n  avg: 0.3\n- name: Jane Maxfield\n  avg: 1.4\n```\n\n## 模式\n```\n{"properties": {"habit": { "description": "一种常见的日常习惯", "type": "string" }, "sustainable_alternative": { "description": "对该习惯的环保替代品", "type": "string"}}, "required": ["habit", "sustainable_alternative"]}\n```\n## 格式良好的实例\n```\nhabit: 使用一次性水瓶进行日常补水。\nsustainable_alternative: 切换到可重复使用的水瓶，以减少塑料废物并降低您的环境足迹。\n``` \n\n请遵循标准的 YAML 格式约定，缩进为 2 个空格，并确保数据类型严格遵循以下 JSON 模式： \n```\n{"properties": {"setup": {"title": "Setup", "description": "设置笑话的问题", "type": "string"}, "punchline": {"title": "Punchline", "description": "解决笑话的答案", "type": "string"}}, "required": ["setup", "punchline"]}\n```\n\n请确保始终将 YAML 输出用三个反引号（```）括起来。请不要添加任何其他内容，除了有效的 YAML 输出！'
```


您可以并且应该尝试在提示的其他部分添加自己的格式提示，以增强或替换默认指令。

## 下一步

您现在已经学习了如何提示模型返回 XML。接下来，请查看 [获取结构化输出的更广泛指南](/docs/how_to/structured_output)，以获取其他相关技术。
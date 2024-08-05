---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/extraction_parse.ipynb
---

# 如何单独使用提示（不调用工具）进行提取

生成结构化输出不需要调用工具的功能。能够很好地遵循提示指令的LLM可以被要求以给定格式输出信息。

这种方法依赖于设计良好的提示，然后解析LLM的输出，以便更好地提取信息。

要在没有工具调用功能的情况下提取数据：

1. 指示LLM生成遵循预期格式的文本（例如，具有特定模式的JSON）；
2. 使用 [输出解析器](/docs/concepts#output-parsers) 将模型响应结构化为所需的Python对象。

首先我们选择一个LLM：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="model" />

:::tip
本教程旨在简单，但通常应该确实包括参考示例以挖掘性能！
:::

## 使用 PydanticOutputParser

以下示例使用内置的 `PydanticOutputParser` 来解析聊天模型的输出。

```python
from typing import List, Optional

from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field, validator


class Person(BaseModel):
    """关于一个人的信息。"""

    name: str = Field(..., description="这个人的名字")
    height_in_meters: float = Field(
        ..., description="这个人的身高以米为单位。"
    )


class People(BaseModel):
    """文本中所有人的识别信息。"""

    people: List[Person]


# 设置解析器
parser = PydanticOutputParser(pydantic_object=People)

# 提示
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "回答用户查询。将输出包裹在 `json` 标签中\n{format_instructions}",
        ),
        ("human", "{query}"),
    ]
).partial(format_instructions=parser.get_format_instructions())
```

让我们看看发送给模型的信息

```python
query = "Anna is 23 years old and she is 6 feet tall"
```

```python
print(prompt.format_prompt(query=query).to_string())
```
```output
System: Answer the user query. Wrap the output in `json` tags
The output should be formatted as a JSON instance that conforms to the JSON schema below.

As an example, for the schema {"properties": {"foo": {"title": "Foo", "description": "a list of strings", "type": "array", "items": {"type": "string"}}}, "required": ["foo"]}
the object {"foo": ["bar", "baz"]} is a well-formatted instance of the schema. The object {"properties": {"foo": ["bar", "baz"]}} is not well-formatted.

Here is the output schema:
```
{"description": "Identifying information about all people in a text.", "properties": {"people": {"title": "People", "type": "array", "items": {"$ref": "#/definitions/Person"}}}, "required": ["people"], "definitions": {"Person": {"title": "Person", "description": "Information about a person.", "type": "object", "properties": {"name": {"title": "Name", "description": "The name of the person", "type": "string"}, "height_in_meters": {"title": "Height In Meters", "description": "The height of the person expressed in meters.", "type": "number"}}, "required": ["name", "height_in_meters"]}}}
```
Human: Anna is 23 years old and she is 6 feet tall
```
定义完提示后，我们只需将提示、模型和输出解析器串联在一起：

```python
chain = prompt | model | parser
chain.invoke({"query": query})
```

```output
People(people=[Person(name='Anna', height_in_meters=1.83)])
```

查看相关的 [Langsmith trace](https://smith.langchain.com/public/92ed52a3-92b9-45af-a663-0a9c00e5e396/r)。

注意，模式在两个地方出现：

1. 在提示中，通过 `parser.get_format_instructions()`；
2. 在链中，以接收格式化的输出并将其结构化为 Python 对象（在这种情况下，是 Pydantic 对象 `People`）。

## 自定义解析

如果需要，可以使用 `LangChain` 和 `LCEL` 轻松创建自定义提示和解析器。

要创建自定义解析器，请定义一个函数，将模型的输出（通常是一个 [AIMessage](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessage.html)）解析为您选择的对象。

请参见下面的 JSON 解析器的简单实现。

```python
import json
import re
from typing import List, Optional

from langchain_anthropic.chat_models import ChatAnthropic
from langchain_core.messages import AIMessage
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field, validator


class Person(BaseModel):
    """关于一个人的信息。"""

    name: str = Field(..., description="这个人的名字")
    height_in_meters: float = Field(
        ..., description="这个人的身高，以米为单位。"
    )


class People(BaseModel):
    """文本中所有人的身份信息。"""

    people: List[Person]


# 提示
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "回答用户查询。将您的答案输出为与给定架构匹配的 JSON：```json\n{schema}\n```. "
            "确保将答案用 ```json 和 ``` 标签包裹起来",
        ),
        ("human", "{query}"),
    ]
).partial(schema=People.schema())


# 自定义解析器
def extract_json(message: AIMessage) -> List[dict]:
    """从字符串中提取 JSON 内容，该字符串中 JSON 嵌入在 ```json 和 ``` 标签之间。

    参数：
        text (str): 包含 JSON 内容的文本。

    返回：
        list: 提取的 JSON 字符串列表。
    """
    text = message.content
    # 定义正则表达式模式以匹配 JSON 块
    pattern = r"```json(.*?)```"

    # 在字符串中查找模式的所有不重叠匹配项
    matches = re.findall(pattern, text, re.DOTALL)

    # 返回匹配的 JSON 字符串列表，去除任何前导或尾随空格
    try:
        return [json.loads(match.strip()) for match in matches]
    except Exception:
        raise ValueError(f"解析失败: {message}")
```


```python
query = "Anna is 23 years old and she is 6 feet tall"
print(prompt.format_prompt(query=query).to_string())
```
```output
System: Answer the user query. Output your answer as JSON that  matches the given schema: ```json
{'title': 'People', 'description': 'Identifying information about all people in a text.', 'type': 'object', 'properties': {'people': {'title': 'People', 'type': 'array', 'items': {'$ref': '#/definitions/Person'}}}, 'required': ['people'], 'definitions': {'Person': {'title': 'Person', 'description': 'Information about a person.', 'type': 'object', 'properties': {'name': {'title': 'Name', 'description': 'The name of the person', 'type': 'string'}, 'height_in_meters': {'title': 'Height In Meters', 'description': 'The height of the person expressed in meters.', 'type': 'number'}}, 'required': ['name', 'height_in_meters']}}}
```. Make sure to wrap the answer in ```json and ``` tags
Human: Anna is 23 years old and she is 6 feet tall
```

```python
chain = prompt | model | extract_json
chain.invoke({"query": query})
```



```output
[{'people': [{'name': 'Anna', 'height_in_meters': 1.83}]}]
```

## 其他库

如果您正在考虑使用解析方法进行提取，请查看 [Kor](https://eyurtsev.github.io/kor/) 库。它是由 `LangChain` 的维护者之一编写的，帮助构建一个考虑示例的提示，允许控制格式（例如，JSON 或 CSV），并在 TypeScript 中表达模式。它似乎工作得很好！
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/structured_output.ipynb
sidebar_position: 3
keywords: [结构化输出, json, 信息提取, with_structured_output]
---

# 如何从模型返回结构化数据

:::info 前提条件

本指南假设您对以下概念有一定的了解：
- [聊天模型](/docs/concepts/#chat-models)
- [函数/工具调用](/docs/concepts/#functiontool-calling)
:::

通常，将模型的输出与特定模式匹配是非常有用的。一个常见的用例是从文本中提取数据以插入数据库或与其他下游系统一起使用。本指南介绍了从模型获取结构化输出的几种策略。

## `.with_structured_output()` 方法

<span data-heading-keywords="with_structured_output"></span>

:::info 支持的模型

您可以在这里找到 [支持此方法的模型列表](/docs/integrations/chat/)。

:::

这是获取结构化输出最简单且最可靠的方法。`with_structured_output()` 针对提供结构化输出的本机 API 的模型实现，例如工具/函数调用或 JSON 模式，并在底层利用这些能力。

该方法接受一个模式作为输入，指定所需输出属性的名称、类型和描述。该方法返回一个类似模型的 Runnable，除了输出字符串或消息外，它输出与给定模式对应的对象。模式可以指定为 TypedDict 类、[JSON Schema](https://json-schema.org/) 或 Pydantic 类。如果使用 TypedDict 或 JSON Schema，则 Runnable 将返回一个字典；如果使用 Pydantic 类，则将返回一个 Pydantic 对象。

作为示例，让我们让模型生成一个笑话，并将设置与笑点分开：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs
  customVarName="llm"
/>

### Pydantic 类

如果我们希望模型返回一个 Pydantic 对象，只需传入所需的 Pydantic 类。使用 Pydantic 的主要优点是模型生成的输出将会被验证。如果缺少任何必需字段或字段类型错误，Pydantic 将会引发错误。

```python
from typing import Optional

from langchain_core.pydantic_v1 import BaseModel, Field


# Pydantic
class Joke(BaseModel):
    """Joke to tell user."""

    setup: str = Field(description="The setup of the joke")
    punchline: str = Field(description="The punchline to the joke")
    rating: Optional[int] = Field(
        default=None, description="How funny the joke is, from 1 to 10"
    )


structured_llm = llm.with_structured_output(Joke)

structured_llm.invoke("Tell me a joke about cats")
```

```output
Joke(setup='Why was the cat sitting on the computer?', punchline='Because it wanted to keep an eye on the mouse!', rating=7)
```

:::tip
除了 Pydantic 类的结构外，Pydantic 类的名称、文档字符串以及参数的名称和提供的描述也非常重要。大多数情况下，`with_structured_output` 是在使用模型的函数/工具调用 API，你可以有效地将所有这些信息视为添加到模型提示中的内容。
:::

### TypedDict 或 JSON Schema

如果您不想使用 Pydantic，明确不想对参数进行验证，或者希望能够流式输出模型结果，您可以使用 TypedDict 类定义您的模式。我们可以选择性地使用 LangChain 支持的特殊 `Annotated` 语法，允许您指定字段的默认值和描述。请注意，如果模型没有生成默认值，则默认值*不会*自动填充，仅用于定义传递给模型的模式。

:::info 需求

- 核心: `langchain-core>=0.2.26`
- 类型扩展: 强烈建议从 `typing_extensions` 导入 `Annotated` 和 `TypedDict`，而不是从 `typing` 导入，以确保在不同 Python 版本之间的一致行为。

:::


```python
from typing_extensions import Annotated, TypedDict


# TypedDict
class Joke(TypedDict):
    """Joke to tell user."""

    setup: Annotated[str, ..., "The setup of the joke"]

    # Alternatively, we could have specified setup as:

    # setup: str                    # no default, no description
    # setup: Annotated[str, ...]    # no default, no description
    # setup: Annotated[str, "foo"]  # default, no description

    punchline: Annotated[str, ..., "The punchline of the joke"]
    rating: Annotated[Optional[int], None, "How funny the joke is, from 1 to 10"]


structured_llm = llm.with_structured_output(Joke)

structured_llm.invoke("Tell me a joke about cats")
```



```output
{'setup': 'Why was the cat sitting on the computer?',
 'punchline': 'Because it wanted to keep an eye on the mouse!',
 'rating': 7}
```


同样，我们可以传入一个 [JSON Schema](https://json-schema.org/) 字典。这不需要任何导入或类，并且非常清楚每个参数的文档，代价是稍微冗长一些。


```python
json_schema = {
    "title": "joke",
    "description": "Joke to tell user.",
    "type": "object",
    "properties": {
        "setup": {
            "type": "string",
            "description": "The setup of the joke",
        },
        "punchline": {
            "type": "string",
            "description": "The punchline to the joke",
        },
        "rating": {
            "type": "integer",
            "description": "How funny the joke is, from 1 to 10",
            "default": None,
        },
    },
    "required": ["setup", "punchline"],
}
structured_llm = llm.with_structured_output(json_schema)

structured_llm.invoke("Tell me a joke about cats")
```



```output
{'setup': 'Why was the cat sitting on the computer?',
 'punchline': 'Because it wanted to keep an eye on the mouse!',
 'rating': 7}
```

### 在多个模式之间进行选择

让模型从多个模式中选择的最简单方法是创建一个具有联合类型属性的父模式：

```python
from typing import Union


# Pydantic
class Joke(BaseModel):
    """要告诉用户的笑话。"""

    setup: str = Field(description="笑话的开场白")
    punchline: str = Field(description="笑话的高潮")
    rating: Optional[int] = Field(
        default=None, description="笑话的幽默程度，从1到10"
    )


class ConversationalResponse(BaseModel):
    """以对话的方式回应。要友善和乐于助人。"""

    response: str = Field(description="对用户查询的对话回应")


class Response(BaseModel):
    output: Union[Joke, ConversationalResponse]


structured_llm = llm.with_structured_output(Response)

structured_llm.invoke("告诉我一个关于猫的笑话")
```


```output
Response(output=Joke(setup='为什么猫坐在电脑上？', punchline='为了盯着鼠标！', rating=8))
```


```python
structured_llm.invoke("你今天怎么样？")
```


```output
Response(output=ConversationalResponse(response="我只是一个数字助手，所以没有感情，但我在这里准备帮助你。今天我能帮你什么？"))
```


或者，您可以直接使用工具调用，让模型在选项之间进行选择，如果您的 [选择的模型支持它](/docs/integrations/chat/)。这涉及更多的解析和设置，但在某些情况下可以带来更好的性能，因为您不必使用嵌套模式。有关更多细节，请参见 [本指南](/docs/how_to/tool_calling)。

### 流式输出

当输出类型为字典（即，当架构指定为 TypedDict 类或 JSON Schema 字典）时，我们可以从结构化模型中流式输出。

:::info

请注意，输出的是已经聚合的块，而不是增量。

:::


```python
from typing_extensions import Annotated, TypedDict


# TypedDict
class Joke(TypedDict):
    """Joke to tell user."""

    setup: Annotated[str, ..., "The setup of the joke"]
    punchline: Annotated[str, ..., "The punchline of the joke"]
    rating: Annotated[Optional[int], None, "How funny the joke is, from 1 to 10"]


structured_llm = llm.with_structured_output(Joke)

for chunk in structured_llm.stream("Tell me a joke about cats"):
    print(chunk)
```
```output
{}
{'setup': ''}
{'setup': 'Why'}
{'setup': 'Why was'}
{'setup': 'Why was the'}
{'setup': 'Why was the cat'}
{'setup': 'Why was the cat sitting'}
{'setup': 'Why was the cat sitting on'}
{'setup': 'Why was the cat sitting on the'}
{'setup': 'Why was the cat sitting on the computer'}
{'setup': 'Why was the cat sitting on the computer?'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': ''}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on the'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on the mouse'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on the mouse!'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on the mouse!', 'rating': 7}
```

### Few-shot prompting

对于更复杂的模式，向提示中添加少量示例非常有用。这可以通过几种方式来实现。

最简单和最通用的方法是在提示中的系统消息中添加示例：

```python
from langchain_core.prompts import ChatPromptTemplate

system = """You are a hilarious comedian. Your specialty is knock-knock jokes. \
Return a joke which has the setup (the response to "Who's there?") and the final punchline (the response to "<setup> who?").

Here are some examples of jokes:

example_user: Tell me a joke about planes
example_assistant: {{"setup": "Why don't planes ever get tired?", "punchline": "Because they have rest wings!", "rating": 2}}

example_user: Tell me another joke about planes
example_assistant: {{"setup": "Cargo", "punchline": "Cargo 'vroom vroom', but planes go 'zoom zoom'!", "rating": 10}}

example_user: Now about caterpillars
example_assistant: {{"setup": "Caterpillar", "punchline": "Caterpillar really slow, but watch me turn into a butterfly and steal the show!", "rating": 5}}"""

prompt = ChatPromptTemplate.from_messages([("system", system), ("human", "{input}")])

few_shot_structured_llm = prompt | structured_llm
few_shot_structured_llm.invoke("what's something funny about woodpeckers")
```


```output
{'setup': 'Woodpecker',
 'punchline': "Woodpecker who? Woodpecker who can't find a tree is just a bird with a headache!",
 'rating': 7}
```


当用于结构化输出的底层方法是工具调用时，我们可以将示例作为显式工具调用传入。您可以查看您使用的模型是否在其 API 参考中使用工具调用。

```python
from langchain_core.messages import AIMessage, HumanMessage, ToolMessage

examples = [
    HumanMessage("Tell me a joke about planes", name="example_user"),
    AIMessage(
        "",
        name="example_assistant",
        tool_calls=[
            {
                "name": "joke",
                "args": {
                    "setup": "Why don't planes ever get tired?",
                    "punchline": "Because they have rest wings!",
                    "rating": 2,
                },
                "id": "1",
            }
        ],
    ),
    # Most tool-calling models expect a ToolMessage(s) to follow an AIMessage with tool calls.
    ToolMessage("", tool_call_id="1"),
    # Some models also expect an AIMessage to follow any ToolMessages,
    # so you may need to add an AIMessage here.
    HumanMessage("Tell me another joke about planes", name="example_user"),
    AIMessage(
        "",
        name="example_assistant",
        tool_calls=[
            {
                "name": "joke",
                "args": {
                    "setup": "Cargo",
                    "punchline": "Cargo 'vroom vroom', but planes go 'zoom zoom'!",
                    "rating": 10,
                },
                "id": "2",
            }
        ],
    ),
    ToolMessage("", tool_call_id="2"),
    HumanMessage("Now about caterpillars", name="example_user"),
    AIMessage(
        "",
        tool_calls=[
            {
                "name": "joke",
                "args": {
                    "setup": "Caterpillar",
                    "punchline": "Caterpillar really slow, but watch me turn into a butterfly and steal the show!",
                    "rating": 5,
                },
                "id": "3",
            }
        ],
    ),
    ToolMessage("", tool_call_id="3"),
]
system = """You are a hilarious comedian. Your specialty is knock-knock jokes. \
Return a joke which has the setup (the response to "Who's there?") \
and the final punchline (the response to "<setup> who?")."""

prompt = ChatPromptTemplate.from_messages(
    [("system", system), ("placeholder", "{examples}"), ("human", "{input}")]
)
few_shot_structured_llm = prompt | structured_llm
few_shot_structured_llm.invoke({"input": "crocodiles", "examples": examples})
```


```output
{'setup': 'Crocodile',
 'punchline': 'Crocodile be seeing you later, alligator!',
 'rating': 7}
```


有关使用工具调用时少量提示的更多信息，请参见 [here](/docs/how_to/function_calling/#Few-shot-prompting)。

### (高级) 指定输出结构的方法

对于支持多种输出结构方式的模型（即，它们同时支持工具调用和 JSON 模式），您可以通过 `method=` 参数指定使用哪种方法。

:::info JSON 模式

如果使用 JSON 模式，您仍然需要在模型提示中指定所需的模式。您传递给 `with_structured_output` 的模式只会用于解析模型输出，而不会像工具调用那样传递给模型。

要查看您使用的模型是否支持 JSON 模式，请检查其在 [API 参考](https://api.python.langchain.com/en/latest/langchain_api_reference.html) 中的条目。

:::


```python
structured_llm = llm.with_structured_output(None, method="json_mode")

structured_llm.invoke(
    "Tell me a joke about cats, respond in JSON with `setup` and `punchline` keys"
)
```



```output
{'setup': 'Why was the cat sitting on the computer?',
 'punchline': 'Because it wanted to keep an eye on the mouse!'}
```

### (高级) 原始输出

LLMs 在生成结构化输出时并不完美，尤其是在模式变得复杂时。您可以通过传递 `include_raw=True` 来避免引发异常并自行处理原始输出。这会将输出格式更改为包含原始消息输出、`parsed` 值（如果成功）以及任何结果错误：

```python
structured_llm = llm.with_structured_output(Joke, include_raw=True)

structured_llm.invoke("Tell me a joke about cats")
```



```output
{'raw': AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_f25ZRmh8u5vHlOWfTUw8sJFZ', 'function': {'arguments': '{"setup":"Why was the cat sitting on the computer?","punchline":"Because it wanted to keep an eye on the mouse!","rating":7}', 'name': 'Joke'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 33, 'prompt_tokens': 93, 'total_tokens': 126}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'stop', 'logprobs': None}, id='run-d880d7e2-df08-4e9e-ad92-dfc29f2fd52f-0', tool_calls=[{'name': 'Joke', 'args': {'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on the mouse!', 'rating': 7}, 'id': 'call_f25ZRmh8u5vHlOWfTUw8sJFZ', 'type': 'tool_call'}], usage_metadata={'input_tokens': 93, 'output_tokens': 33, 'total_tokens': 126}),
 'parsed': {'setup': 'Why was the cat sitting on the computer?',
  'punchline': 'Because it wanted to keep an eye on the mouse!',
  'rating': 7},
 'parsing_error': None}
```

## 直接提示和解析模型输出

并非所有模型都支持 `.with_structured_output()`，因为并非所有模型都具备工具调用或 JSON 模式支持。对于这类模型，您需要直接提示模型使用特定格式，并使用输出解析器从原始模型输出中提取结构化响应。

### 使用 `PydanticOutputParser`

以下示例使用内置的 [`PydanticOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.pydantic.PydanticOutputParser.html) 来解析聊天模型的输出，该模型被提示以匹配给定的 Pydantic 模式。请注意，我们直接从解析器的方法向提示中添加 `format_instructions`：

```python
from typing import List

from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field


class Person(BaseModel):
    """关于一个人的信息."""

    name: str = Field(..., description="这个人的名字")
    height_in_meters: float = Field(
        ..., description="这个人的身高，以米为单位."
    )


class People(BaseModel):
    """文本中所有人的身份信息."""

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

让我们看看发送给模型的信息：

```python
query = "Anna is 23 years old and she is 6 feet tall"

print(prompt.invoke(query).to_string())
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
现在让我们调用它：

```python
chain = prompt | llm | parser

chain.invoke({"query": query})
```

```output
People(people=[Person(name='Anna', height_in_meters=1.8288)])
```

有关使用输出解析器与结构化输出提示技术的深入探讨，请参见 [本指南](/docs/how_to/output_parser_structured)。

### 自定义解析

您还可以使用 [LangChain 表达式语言 (LCEL)](/docs/concepts/#langchain-expression-language) 创建自定义提示和解析器，通过一个简单的函数解析模型的输出：

```python
import json
import re
from typing import List

from langchain_core.messages import AIMessage
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field


class Person(BaseModel):
    """关于一个人的信息。"""

    name: str = Field(..., description="这个人的名字")
    height_in_meters: float = Field(
        ..., description="这个人的身高，以米为单位。"
    )


class People(BaseModel):
    """文本中所有人的识别信息。"""

    people: List[Person]


# 提示
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "回答用户查询。将您的答案输出为与给定模式匹配的 JSON：```json\n{schema}\n```. "
            "确保将答案包裹在 ```json 和 ``` 标签中",
        ),
        ("human", "{query}"),
    ]
).partial(schema=People.schema())


# 自定义解析器
def extract_json(message: AIMessage) -> List[dict]:
    """从一个字符串中提取 JSON 内容，该字符串中 JSON 嵌入在 ```json 和 ``` 标签之间。

    参数：
        text (str): 包含 JSON 内容的文本。

    返回：
        list: 提取的 JSON 字符串列表。
    """
    text = message.content
    # 定义正则表达式模式以匹配 JSON 块
    pattern = r"```json(.*?)```"

    # 在字符串中找到所有不重叠的模式匹配
    matches = re.findall(pattern, text, re.DOTALL)

    # 返回匹配的 JSON 字符串列表，去除任何前导或尾随空格
    try:
        return [json.loads(match.strip()) for match in matches]
    except Exception:
        raise ValueError(f"解析失败: {message}")
```

这是发送给模型的提示：

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
当我们调用它时，它看起来是这样的：

```python
chain = prompt | llm | extract_json

chain.invoke({"query": query})
```

```output
[{'people': [{'name': 'Anna', 'height_in_meters': 1.8288}]}]
```
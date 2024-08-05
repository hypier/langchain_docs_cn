# 如何在不同的 Pydantic 版本中使用 LangChain

- Pydantic v2 于 2023 年 6 月发布 (https://docs.pydantic.dev/2.0/blog/pydantic-v2-final/)。
- v2 包含了一些重大更改 (https://docs.pydantic.dev/2.0/migration/)。
- Pydantic 1 的生命周期结束于 2024 年 6 月。LangChain 将在不久的将来停止对 Pydantic 1 的支持，并可能内部迁移到 Pydantic 2。时间表暂定为 9 月。此更改将伴随主 langchain 包的次版本提升至 0.3.x。

从 `langchain>=0.0.267` 开始，LangChain 允许用户安装 Pydantic V1 或 V2。

在内部，LangChain 继续通过 Pydantic 2 的 v1 命名空间使用 [Pydantic V1](https://docs.pydantic.dev/latest/migration/#continue-using-pydantic-v1-features)。

由于 Pydantic 不支持混合 .v1 和 .v2 对象，用户在使用 LangChain 与 Pydantic 时应注意一些问题。

## 1. 将 Pydantic 对象传递给 LangChain API

大多数接受 Pydantic 对象的 LangChain API 已更新为同时接受 Pydantic v1 和 v2 对象。

* 如果安装了 `pydantic 1`，则 Pydantic v1 对象对应于 `pydantic.BaseModel` 的子类；如果安装了 `pydantic 2`，则对应于 `pydantic.v1.BaseModel` 的子类。
* 如果安装了 `pydantic 2`，则 Pydantic v2 对象对应于 `pydantic.BaseModel` 的子类。

| API                                    | Pydantic 1 | Pydantic 2                                                     |
|----------------------------------------|------------|----------------------------------------------------------------|
| `BaseChatModel.bind_tools`             | Yes        | langchain-core>=0.2.23, 适当版本的合作包                          |
| `BaseChatModel.with_structured_output` | Yes        | langchain-core>=0.2.23, 适当版本的合作包                          |
| `Tool.from_function`                   | Yes        | langchain-core>=0.2.23                                         |
| `StructuredTool.from_function`         | Yes        | langchain-core>=0.2.23                                         |

通过 `bind_tools` 或 `with_structured_output` API 接受 pydantic v2 对象的合作包：

| 包名称               | pydantic v1 | pydantic v2 |
|---------------------|-------------|-------------|
| langchain-mistralai | Yes         | >=0.1.11    |
| langchain-anthropic | Yes         | >=0.1.21    |
| langchain-robocorp  | Yes         | >=0.0.10    |
| langchain-openai    | Yes         | >=0.1.19    |
| langchain-fireworks | Yes         | >=0.1.5     |

未来将更新其他合作包以接受 Pydantic v2 对象。

如果您仍然遇到这些 API 或其他接受 Pydantic 对象的 API 的问题，请提交问题，我们会处理。

示例：

在 `langchain-core<0.2.23` 之前，传递 Pydantic v1 对象给 LangChain API。

```python
from langchain_openai import ChatOpenAI
from pydantic.v1 import BaseModel # <-- Note v1 namespace

class Person(BaseModel):
    """Personal information"""
    name: str
    
model = ChatOpenAI()
model = model.with_structured_output(Person)

model.invoke('Bob is a person.')
```

在 `langchain-core>=0.2.23` 之后，传递 Pydantic v1 或 v2 对象给 LangChain API。

```python
from langchain_openai import ChatOpenAI
from pydantic import BaseModel

class Person(BaseModel):
    """Personal information"""
    name: str
    
    
model = ChatOpenAI()
model = model.with_structured_output(Person)

model.invoke('Bob is a person.')
```

## 2. 子类化 LangChain 模型

由于 LangChain 内部使用 Pydantic v1，如果您正在子类化 LangChain 模型，您应该使用 Pydantic v1 的基本类型。

**示例 1：通过继承扩展**

**是的** 

```python
from pydantic.v1 import validator
from langchain_core.tools import BaseTool

class CustomTool(BaseTool): # BaseTool 是 v1 代码
    x: int = Field(default=1)

    def _run(*args, **kwargs):
        return "hello"

    @validator('x') # v1 代码
    @classmethod
    def validate_x(cls, x: int) -> int:
        return 1
    

CustomTool(
    name='custom_tool',
    description="hello",
    x=1,
)
```

将 Pydantic v2 的基本类型与 Pydantic v1 的基本类型混合可能会引发难以理解的错误

**不可以** 

```python
from pydantic import Field, field_validator # pydantic v2
from langchain_core.tools import BaseTool

class CustomTool(BaseTool): # BaseTool 是 v1 代码
    x: int = Field(default=1)

    def _run(*args, **kwargs):
        return "hello"

    @field_validator('x') # v2 代码
    @classmethod
    def validate_x(cls, x: int) -> int:
        return 1
    

CustomTool( 
    name='custom_tool',
    description="hello",
    x=1,
)
```

## 3. 禁用在 Pydantic v2 模型中使用的 LangChain 对象的运行时验证

例如，

```python
from typing import Annotated

from langchain_openai import ChatOpenAI # <-- ChatOpenAI 使用 pydantic v1
from pydantic import BaseModel, SkipValidation


class Foo(BaseModel): # <-- BaseModel 来自 Pydantic v2
    model: Annotated[ChatOpenAI, SkipValidation()]

Foo(model=ChatOpenAI(api_key="hello"))
```

## 4: LangServe 无法在运行 Pydantic 2 时生成 OpenAPI 文档

如果您使用的是 Pydantic 2，您将无法使用 LangServe 生成 OpenAPI 文档。

如果您需要 OpenAPI 文档，您可以选择安装 Pydantic 1：

`pip install pydantic==1.10.17`

或者使用 LangChain 中的 `APIHandler` 对象手动创建 API 的路由。

参见： https://python.langchain.com/v0.2/docs/langserve/#pydantic
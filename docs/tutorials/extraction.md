---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/tutorials/extraction.ipynb
sidebar_position: 4
---

# 构建提取链

:::info 先决条件

本指南假设您熟悉以下概念：

- [聊天模型](/docs/concepts/#chat-models)
- [工具](/docs/concepts/#tools)
- [工具调用](/docs/concepts/#function-tool-calling)

:::

在本教程中，我们将构建一个链，以从非结构化文本中提取结构化信息。

:::important
本教程仅适用于支持 **工具调用** 的模型
:::

## 设置

### Jupyter Notebook

本指南（以及文档中的大多数其他指南）使用 [Jupyter notebooks](https://jupyter.org/) 并假设读者也使用它。Jupyter notebooks 非常适合学习如何使用 LLM 系统，因为有时事情可能会出错（意外输出、API 故障等），在交互式环境中逐步阅读指南是更好理解它们的好方法。

本教程和其他教程在 Jupyter notebook 中运行可能最为方便。有关如何安装的说明，请参见 [这里](https://jupyter.org/install)。

### 安装

要安装 LangChain，请运行：

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CodeBlock from "@theme/CodeBlock";

<Tabs>
  <TabItem value="pip" label="Pip" default>
    <CodeBlock language="bash">pip install langchain</CodeBlock>
  </TabItem>
  <TabItem value="conda" label="Conda">
    <CodeBlock language="bash">conda install langchain -c conda-forge</CodeBlock>
  </TabItem>
</Tabs>

有关更多详细信息，请参阅我们的 [安装指南](/docs/how_to/installation).

### LangSmith

您使用 LangChain 构建的许多应用程序将包含多个步骤和多次调用 LLM。  
随着这些应用程序变得越来越复杂，能够检查您的链或代理内部到底发生了什么变得至关重要。  
最好的方法是使用 [LangSmith](https://smith.langchain.com)。

在您在上述链接注册后，请确保设置您的环境变量以开始记录跟踪信息：

```shell
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="..."
```

或者，如果在笔记本中，您可以通过以下方式设置它们：

```python
import getpass
import os

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 模式

首先，我们需要描述我们想要从文本中提取的信息。

我们将使用 Pydantic 来定义一个示例模式，以提取个人信息。

```python
from typing import Optional

from langchain_core.pydantic_v1 import BaseModel, Field


class Person(BaseModel):
    """关于一个人的信息。"""

    # ^ 实体 Person 的文档字符串。
    # 此文档字符串作为模式 Person 的描述发送给 LLM，
    # 并且可以帮助改善提取结果。

    # 请注意：
    # 1. 每个字段都是 `optional` -- 这允许模型拒绝提取它！
    # 2. 每个字段都有一个 `description` -- 这个描述被 LLM 使用。
    # 有一个好的描述可以帮助改善提取结果。
    name: Optional[str] = Field(default=None, description="这个人的名字")
    hair_color: Optional[str] = Field(
        default=None, description="如果已知，这个人的头发颜色"
    )
    height_in_meters: Optional[str] = Field(
        default=None, description="以米为单位测量的身高"
    )
```

在定义模式时，有两个最佳实践：

1. 记录 **属性** 和 **模式** 本身：这些信息会发送给 LLM，并用于提高信息提取的质量。
2. 不要强迫 LLM 编造信息！上述我们为属性使用了 `Optional`，允许 LLM 输出 `None`，如果它不知道答案。

:::important
为了获得最佳性能，请详细记录模式，并确保模型在文本中没有可提取的信息时不会强制返回结果。
:::

## 提取器

让我们使用上面定义的模式创建一个信息提取器。

```python
from typing import Optional

from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.pydantic_v1 import BaseModel, Field

# 定义一个自定义提示以提供指令和任何额外的上下文。
# 1) 您可以在提示模板中添加示例以提高提取质量
# 2) 引入额外的参数以考虑上下文（例如，包含提取文本的文档的元数据。）
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "您是一个专业的提取算法。"
            "只从文本中提取相关信息。"
            "如果您不知道要提取的属性的值，"
            "则返回该属性的值为null。",
        ),
        # 请参阅如何通过
        # 参考示例提高性能的说明。
        # MessagesPlaceholder('examples'),
        ("human", "{text}"),
    ]
)
```

我们需要使用一个支持函数/工具调用的模型。

请查看[文档](/docs/concepts#function-tool-calling)，以获取可以与此API一起使用的一些模型列表。

```python
from langchain_mistralai import ChatMistralAI

llm = ChatMistralAI(model="mistral-large-latest", temperature=0)

runnable = prompt | llm.with_structured_output(schema=Person)
```
```output
/Users/harrisonchase/workplace/langchain/libs/core/langchain_core/_api/beta_decorator.py:87: LangChainBetaWarning: 方法 `ChatMistralAI.with_structured_output` 正在测试中。它正在积极开发中，因此API可能会发生变化。
  warn_beta(
```
让我们测试一下

```python
text = "Alan Smith is 6 feet tall and has blond hair."
runnable.invoke({"text": text})
```

```output
Person(name='Alan Smith', hair_color='blond', height_in_meters='1.83')
```

:::important 

提取是生成性的 🤯

LLM是生成模型，因此它们可以做一些非常酷的事情，例如正确提取以米为单位的人身高，即使它是以英尺提供的！
:::

我们可以在这里看到LangSmith跟踪： https://smith.langchain.com/public/44b69a63-3b3b-47b8-8a6d-61b46533f015/r

## 多个实体

在**大多数情况下**，您应该提取实体列表而不是单个实体。

这可以通过在模型内部嵌套模型，使用 pydantic 容易实现。


```python
from typing import List, Optional

from langchain_core.pydantic_v1 import BaseModel, Field


class Person(BaseModel):
    """关于一个人的信息。"""

    # ^ 实体 Person 的文档字符串。
    # 此文档字符串作为模式 Person 的描述发送给 LLM，
    # 并且可以帮助改善提取结果。

    # 注意：
    # 1. 每个字段都是 `optional` -- 这允许模型拒绝提取它！
    # 2. 每个字段都有一个 `description` -- 这个描述被 LLM 使用。
    # 拥有良好的描述可以帮助改善提取结果。
    name: Optional[str] = Field(default=None, description="这个人的名字")
    hair_color: Optional[str] = Field(
        default=None, description="如果已知，这个人的头发颜色"
    )
    height_in_meters: Optional[str] = Field(
        default=None, description="以米为单位测量的身高"
    )


class Data(BaseModel):
    """关于人们的提取数据。"""

    # 创建一个模型，以便我们可以提取多个实体。
    people: List[Person]
```

:::important
提取结果可能并不完美。请继续查看如何使用**参考示例**来提高提取质量，并查看**指南**部分！
:::


```python
runnable = prompt | llm.with_structured_output(schema=Data)
text = "My name is Jeff, my hair is black and i am 6 feet tall. Anna has the same color hair as me."
runnable.invoke({"text": text})
```



```output
Data(people=[Person(name='Jeff', hair_color=None, height_in_meters=None), Person(name='Anna', hair_color=None, height_in_meters=None)])
```


:::tip
当模式适应提取**多个实体**时，它还允许模型在文本中没有相关信息时提取**无实体**，通过提供一个空列表。

这通常是一个**好**事！它允许在实体上指定**必需**属性，而不必强迫模型检测该实体。
:::

我们可以在这里看到 LangSmith 跟踪： https://smith.langchain.com/public/7173764d-5e76-45fe-8496-84460bd9cdef/r

## 下一步

现在您已经了解了使用 LangChain 进行提取的基础知识，您可以继续阅读其余的操作指南：

- [添加示例](/docs/how_to/extraction_examples): 了解如何使用 **参考示例** 来提高性能。
- [处理长文本](/docs/how_to/extraction_long_text): 如果文本无法适应 LLM 的上下文窗口，您应该怎么做？
- [使用解析方法](/docs/how_to/extraction_parse): 使用基于提示的方法来提取不支持 **工具/函数调用** 的模型。
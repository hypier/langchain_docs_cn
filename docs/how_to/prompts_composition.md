---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/prompts_composition.ipynb
sidebar_position: 5
---

# 如何组合提示

:::info 前提条件

本指南假设您熟悉以下概念：
- [提示模板](/docs/concepts/#prompt-templates)

:::

LangChain 提供了一个用户友好的界面，用于将提示的不同部分组合在一起。您可以使用字符串提示或聊天提示来实现这一点。以这种方式构建提示可以方便地重用组件。

## 字符串提示组成

在处理字符串提示时，每个模板会连接在一起。您可以直接使用提示或字符串（列表中的第一个元素需要是提示）。

```python
from langchain_core.prompts import PromptTemplate

prompt = (
    PromptTemplate.from_template("Tell me a joke about {topic}")
    + ", make it funny"
    + "\n\nand in {language}"
)

prompt
```

```output
PromptTemplate(input_variables=['language', 'topic'], template='Tell me a joke about {topic}, make it funny\n\nand in {language}')
```

```python
prompt.format(topic="sports", language="spanish")
```

```output
'Tell me a joke about sports, make it funny\n\nand in spanish'
```

## 聊天提示构成

聊天提示由一系列消息组成。与上述示例类似，我们可以连接聊天提示模板。每个新元素都是最终提示中的一条新消息。

首先，让我们初始化一个 [`ChatPromptTemplate`](https://api.python.langchain.com/en/latest/prompts/langchain_core.prompts.chat.ChatPromptTemplate.html) 和一个 [`SystemMessage`](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.system.SystemMessage.html)。

```python
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage

prompt = SystemMessage(content="You are a nice pirate")
```

然后，您可以轻松地创建一个管道，将其与其他消息 *或* 消息模板结合使用。当没有变量需要格式化时使用 `Message`，当有变量需要格式化时使用 `MessageTemplate`。您也可以仅使用字符串（注意：这将自动推断为一个 [`HumanMessagePromptTemplate`](https://api.python.langchain.com/en/latest/prompts/langchain_core.prompts.chat.HumanMessagePromptTemplate.html)）。

```python
new_prompt = (
    prompt + HumanMessage(content="hi") + AIMessage(content="what?") + "{input}"
)
```

在底层，这会创建一个 ChatPromptTemplate 类的实例，因此您可以像以前一样使用它！

```python
new_prompt.format_messages(input="i said hi")
```



```output
[SystemMessage(content='You are a nice pirate'),
 HumanMessage(content='hi'),
 AIMessage(content='what?'),
 HumanMessage(content='i said hi')]
```

## 使用 PipelinePrompt

LangChain 包含一个名为 [`PipelinePromptTemplate`](https://api.python.langchain.com/en/latest/prompts/langchain_core.prompts.pipeline.PipelinePromptTemplate.html) 的类，当您想重用提示的部分时，它非常有用。PipelinePrompt 由两个主要部分组成：

- 最终提示：返回的最终提示
- 管道提示：由字符串名称和提示模板组成的元组列表。每个提示模板将被格式化，然后作为具有相同名称的变量传递给未来的提示模板。

```python
from langchain_core.prompts import PipelinePromptTemplate, PromptTemplate

full_template = """{introduction}

{example}

{start}"""
full_prompt = PromptTemplate.from_template(full_template)

introduction_template = """You are impersonating {person}."""
introduction_prompt = PromptTemplate.from_template(introduction_template)

example_template = """Here's an example of an interaction:

Q: {example_q}
A: {example_a}"""
example_prompt = PromptTemplate.from_template(example_template)

start_template = """Now, do this for real!

Q: {input}
A:"""
start_prompt = PromptTemplate.from_template(start_template)

input_prompts = [
    ("introduction", introduction_prompt),
    ("example", example_prompt),
    ("start", start_prompt),
]
pipeline_prompt = PipelinePromptTemplate(
    final_prompt=full_prompt, pipeline_prompts=input_prompts
)

pipeline_prompt.input_variables
```

```output
['person', 'example_a', 'example_q', 'input']
```

```python
print(
    pipeline_prompt.format(
        person="Elon Musk",
        example_q="What's your favorite car?",
        example_a="Tesla",
        input="What's your favorite social media site?",
    )
)
```
```output
You are impersonating Elon Musk.

Here's an example of an interaction:

Q: What's your favorite car?
A: Tesla

Now, do this for real!

Q: What's your favorite social media site?
A:
```

## 下一步

您现在已经学习了如何组合提示。

接下来，请查看本节中有关提示模板的其他操作指南，例如 [向您的提示模板添加少量示例](/docs/how_to/few_shot_examples_chat)。
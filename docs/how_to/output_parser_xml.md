---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/output_parser_xml.ipynb
---

# 如何解析 XML 输出

:::info 前提条件

本指南假设您熟悉以下概念：
- [聊天模型](/docs/concepts/#chat-models)
- [输出解析器](/docs/concepts/#output-parsers)
- [提示模板](/docs/concepts/#prompt-templates)
- [结构化输出](/docs/how_to/structured_output)
- [将可运行项串联在一起](/docs/how_to/sequence/)

:::

不同供应商的 LLM 在特定数据上训练的情况下，通常具有不同的优势。这也意味着某些模型在生成 JSON 以外格式的输出时可能“更好”且更可靠。

本指南将向您展示如何使用 [`XMLOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.xml.XMLOutputParser.html) 来提示模型生成 XML 输出，然后将该输出解析为可用格式。

:::note
请记住，大型语言模型是泄漏的抽象！您必须使用具有足够能力的 LLM 来生成格式良好的 XML。
:::

在以下示例中，我们使用 Anthropic 的 Claude-2 模型 (https://docs.anthropic.com/claude/docs)，这是一个针对 XML 标签进行了优化的模型。


```python
%pip install -qU langchain langchain-anthropic

import os
from getpass import getpass

os.environ["ANTHROPIC_API_KEY"] = getpass()
```

让我们开始向模型发出简单的请求。


```python
from langchain_anthropic import ChatAnthropic
from langchain_core.output_parsers import XMLOutputParser
from langchain_core.prompts import PromptTemplate

model = ChatAnthropic(model="claude-2.1", max_tokens_to_sample=512, temperature=0.1)

actor_query = "生成汤姆·汉克斯的简短电影作品表。"

output = model.invoke(
    f"""{actor_query}
请将电影用 <movie></movie> 标签括起来"""
)

print(output.content)
```
```output
这是汤姆·汉克斯的简短电影作品表，电影用 XML 标签括起来：

<movie>Splash</movie>
<movie>Big</movie>
<movie>A League of Their Own</movie>
<movie>Sleepless in Seattle</movie>
<movie>Forrest Gump</movie>
<movie>Toy Story</movie>
<movie>Apollo 13</movie>
<movie>Saving Private Ryan</movie>
<movie>Cast Away</movie>
<movie>The Da Vinci Code</movie>
```
这实际上效果很好！但将 XML 解析为更易于使用的格式会更好。我们可以使用 `XMLOutputParser` 来为提示添加默认格式说明，并将输出的 XML 解析为字典：


```python
parser = XMLOutputParser()

# 我们将在下面的提示中添加这些说明
parser.get_format_instructions()
```



```output
'输出应格式化为 XML 文件。\n1. 输出应符合以下标签。 \n2. 如果没有给出标签，请自行创建。\n3. 请记住始终打开和关闭所有标签。\n\n例如，对于标签 ["foo", "bar", "baz"]:\n1. 字符串 "<foo>\n   <bar>\n      <baz></baz>\n   </bar>\n</foo>" 是该模式的格式良好的实例。 \n2. 字符串 "<foo>\n   <bar>\n   </foo>" 是格式不良的实例。\n3. 字符串 "<foo>\n   <tag>\n   </tag>\n</foo>" 是格式不良的实例。\n\n以下是输出标签：\n```\nNone\n```'
```



```python
prompt = PromptTemplate(
    template="""{query}\n{format_instructions}""",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)

chain = prompt | model | parser

output = chain.invoke({"query": actor_query})
print(output)
```
```output
{'filmography': [{'movie': [{'title': 'Big'}, {'year': '1988'}]}, {'movie': [{'title': 'Forrest Gump'}, {'year': '1994'}]}, {'movie': [{'title': 'Toy Story'}, {'year': '1995'}]}, {'movie': [{'title': 'Saving Private Ryan'}, {'year': '1998'}]}, {'movie': [{'title': 'Cast Away'}, {'year': '2000'}]}]}
```
我们还可以添加一些标签以满足我们的需求。您可以并且应该尝试在提示的其他部分添加您自己的格式提示，以增强或替换默认说明：


```python
parser = XMLOutputParser(tags=["movies", "actor", "film", "name", "genre"])

# 我们将在下面的提示中添加这些说明
parser.get_format_instructions()
```



```output
'输出应格式化为 XML 文件。\n1. 输出应符合以下标签。 \n2. 如果没有给出标签，请自行创建。\n3. 请记住始终打开和关闭所有标签。\n\n例如，对于标签 ["foo", "bar", "baz"]:\n1. 字符串 "<foo>\n   <bar>\n      <baz></baz>\n   </bar>\n</foo>" 是该模式的格式良好的实例。 \n2. 字符串 "<foo>\n   <bar>\n   </foo>" 是格式不良的实例。\n3. 字符串 "<foo>\n   <tag>\n   </tag>\n</foo>" 是格式不良的实例。\n\n以下是输出标签：\n```\n[\'movies\', \'actor\', \'film\', \'name\', \'genre\']\n```'
```



```python
prompt = PromptTemplate(
    template="""{query}\n{format_instructions}""",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)


chain = prompt | model | parser

output = chain.invoke({"query": actor_query})

print(output)
```
```output
{'movies': [{'actor': [{'name': '汤姆·汉克斯'}, {'film': [{'name': 'Forrest Gump'}, {'genre': '剧情'}]}, {'film': [{'name': 'Cast Away'}, {'genre': '冒险'}]}, {'film': [{'name': 'Saving Private Ryan'}, {'genre': '战争'}]}]}]}
```
该输出解析器还支持部分块的流式传输。以下是一个示例：


```python
for s in chain.stream({"query": actor_query}):
    print(s)
```
```output
{'movies': [{'actor': [{'name': '汤姆·汉克斯'}]}]}
{'movies': [{'actor': [{'film': [{'name': 'Forrest Gump'}]}]}]}
{'movies': [{'actor': [{'film': [{'genre': '剧情'}]}]}]}
{'movies': [{'actor': [{'film': [{'name': 'Cast Away'}]}]}]}
{'movies': [{'actor': [{'film': [{'genre': '冒险'}]}]}]}
{'movies': [{'actor': [{'film': [{'name': 'Saving Private Ryan'}]}]}]}
{'movies': [{'actor': [{'film': [{'genre': '战争'}]}]}]}
```

## 下一步

您现在已经学习了如何提示模型返回 XML。接下来，请查看有关获取结构化输出的[更广泛指南](/docs/how_to/structured_output)，以了解其他相关技术。
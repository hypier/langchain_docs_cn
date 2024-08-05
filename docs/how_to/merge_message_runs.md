---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/merge_message_runs.ipynb
---

# 如何合并连续的相同类型消息

某些模型不支持传递连续的相同类型消息（即“相同消息类型的运行”）。

`merge_message_runs` 实用程序使合并连续的相同类型消息变得简单。

## 基本用法


```python
from langchain_core.messages import (
    AIMessage,
    HumanMessage,
    SystemMessage,
    merge_message_runs,
)

messages = [
    SystemMessage("you're a good assistant."),
    SystemMessage("you always respond with a joke."),
    HumanMessage([{"type": "text", "text": "i wonder why it's called langchain"}]),
    HumanMessage("and who is harrison chasing anyways"),
    AIMessage(
        'Well, I guess they thought "WordRope" and "SentenceString" just didn\'t have the same ring to it!'
    ),
    AIMessage("Why, he's probably chasing after the last cup of coffee in the office!"),
]

merged = merge_message_runs(messages)
print("\n\n".join([repr(x) for x in merged]))
```
```output
SystemMessage(content="you're a good assistant.\nyou always respond with a joke.")

HumanMessage(content=[{'type': 'text', 'text': "i wonder why it's called langchain"}, 'and who is harrison chasing anyways'])

AIMessage(content='Well, I guess they thought "WordRope" and "SentenceString" just didn\'t have the same ring to it!\nWhy, he\'s probably chasing after the last cup of coffee in the office!')
```
注意，如果要合并的消息内容之一是内容块的列表，则合并后的消息将具有内容块的列表。如果要合并的两个消息都是字符串内容，则这些内容将用换行符连接。

`merge_message_runs` 工具也可以与使用重载的 `+` 操作组合在一起的消息一起使用：


```python
messages = (
    SystemMessage("you're a good assistant.")
    + SystemMessage("you always respond with a joke.")
    + HumanMessage([{"type": "text", "text": "i wonder why it's called langchain"}])
    + HumanMessage("and who is harrison chasing anyways")
    + AIMessage(
        'Well, I guess they thought "WordRope" and "SentenceString" just didn\'t have the same ring to it!'
    )
    + AIMessage(
        "Why, he's probably chasing after the last cup of coffee in the office!"
    )
)

merged = merge_message_runs(messages)
print("\n\n".join([repr(x) for x in merged]))
```

## 链接

`merge_message_runs` 可以以命令式（如上所示）或声明式使用，使其能够轻松与其他组件组合成链：

```python
# pip install -U langchain-anthropic
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(model="claude-3-sonnet-20240229", temperature=0)
# 注意我们没有传入消息。这会创建
# 一个接收消息作为输入的 RunnableLambda
merger = merge_message_runs()
chain = merger | llm
chain.invoke(messages)
```

```output
AIMessage(content=[], response_metadata={'id': 'msg_01D6R8Naum57q8qBau9vLBUX', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 84, 'output_tokens': 3}}, id='run-ac0c465b-b54f-4b8b-9295-e5951250d653-0', usage_metadata={'input_tokens': 84, 'output_tokens': 3, 'total_tokens': 87})
```

查看 LangSmith 跟踪，我们可以看到在消息传递给模型之前，它们被合并： https://smith.langchain.com/public/ab558677-cac9-4c59-9066-1ecce5bcd87c/r

仅查看合并器，我们可以看到它是一个可调用的 Runnable 对象，可以像所有 Runnables 一样被调用：

```python
merger.invoke(messages)
```

```output
[SystemMessage(content="you're a good assistant.\nyou always respond with a joke."),
 HumanMessage(content=[{'type': 'text', 'text': "i wonder why it's called langchain"}, 'and who is harrison chasing anyways']),
 AIMessage(content='Well, I guess they thought "WordRope" and "SentenceString" just didn\'t have the same ring to it!\nWhy, he\'s probably chasing after the last cup of coffee in the office!')]
```

## API 参考

有关所有参数的完整描述，请访问 API 参考： https://api.python.langchain.com/en/latest/messages/langchain_core.messages.utils.merge_message_runs.html
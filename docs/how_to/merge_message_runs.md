---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/merge_message_runs.ipynb
---
# How to merge consecutive messages of the same type

Certain models do not support passing in consecutive messages of the same type (a.k.a. "runs" of the same message type).

The `merge_message_runs` utility makes it easy to merge consecutive messages of the same type.

## Basic usage


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
Notice that if the contents of one of the messages to merge is a list of content blocks then the merged message will have a list of content blocks. And if both messages to merge have string contents then those are concatenated with a newline character.

The `merge_message_runs` utility also works with messages composed together using the overloaded `+` operation:


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

## Chaining

`merge_message_runs` can be used in an imperatively (like above) or declaratively, making it easy to compose with other components in a chain:


```python
# pip install -U langchain-anthropic
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(model="claude-3-sonnet-20240229", temperature=0)
# Notice we don't pass in messages. This creates
# a RunnableLambda that takes messages as input
merger = merge_message_runs()
chain = merger | llm
chain.invoke(messages)
```



```output
AIMessage(content=[], response_metadata={'id': 'msg_01D6R8Naum57q8qBau9vLBUX', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 84, 'output_tokens': 3}}, id='run-ac0c465b-b54f-4b8b-9295-e5951250d653-0', usage_metadata={'input_tokens': 84, 'output_tokens': 3, 'total_tokens': 87})
```


Looking at the LangSmith trace we can see that before the messages are passed to the model they are merged: https://smith.langchain.com/public/ab558677-cac9-4c59-9066-1ecce5bcd87c/r

Looking at just the merger, we can see that it's a Runnable object that can be invoked like all Runnables:


```python
merger.invoke(messages)
```



```output
[SystemMessage(content="you're a good assistant.\nyou always respond with a joke."),
 HumanMessage(content=[{'type': 'text', 'text': "i wonder why it's called langchain"}, 'and who is harrison chasing anyways']),
 AIMessage(content='Well, I guess they thought "WordRope" and "SentenceString" just didn\'t have the same ring to it!\nWhy, he\'s probably chasing after the last cup of coffee in the office!')]
```


## API reference

For a complete description of all arguments head to the API reference: https://api.python.langchain.com/en/latest/messages/langchain_core.messages.utils.merge_message_runs.html

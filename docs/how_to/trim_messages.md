---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/trim_messages.ipynb
---

# 如何修剪消息

:::info 前提条件

本指南假设您熟悉以下概念：

- [消息](/docs/concepts/#messages)
- [聊天模型](/docs/concepts/#chat-models)
- [链式调用](/docs/how_to/sequence/)
- [聊天历史](/docs/concepts/#chat-history)

本指南中的方法还要求 `langchain-core>=0.2.9`。

:::

所有模型都有有限的上下文窗口，这意味着它们可以作为输入的令牌数量是有限的。如果您有非常长的消息或一个累积了长消息历史的链/代理，您需要管理传递给模型的消息长度。

`trim_messages` 工具提供了一些基本策略，用于将消息列表修剪到特定的令牌长度。

## 获取最后的 `max_tokens` 令牌

要获取消息列表中的最后 `max_tokens`，我们可以设置 `strategy="last"`。请注意，对于我们的 `token_counter`，我们可以传入一个函数（更多内容见下文）或一个语言模型（因为语言模型具有消息令牌计数方法）。在修剪消息以适应特定模型的上下文窗口时，传入模型是有意义的：

```python
# pip install -U langchain-openai
from langchain_core.messages import (
    AIMessage,
    HumanMessage,
    SystemMessage,
    trim_messages,
)
from langchain_openai import ChatOpenAI

messages = [
    SystemMessage("you're a good assistant, you always respond with a joke."),
    HumanMessage("i wonder why it's called langchain"),
    AIMessage(
        'Well, I guess they thought "WordRope" and "SentenceString" just didn\'t have the same ring to it!'
    ),
    HumanMessage("and who is harrison chasing anyways"),
    AIMessage(
        "Hmmm let me think.\n\nWhy, he's probably chasing after the last cup of coffee in the office!"
    ),
    HumanMessage("what do you call a speechless parrot"),
]

trim_messages(
    messages,
    max_tokens=45,
    strategy="last",
    token_counter=ChatOpenAI(model="gpt-4o"),
)
```



```output
[AIMessage(content="Hmmm let me think.\n\nWhy, he's probably chasing after the last cup of coffee in the office!"),
 HumanMessage(content='what do you call a speechless parrot')]
```


如果我们想始终保留初始的系统消息，可以指定 `include_system=True`：

```python
trim_messages(
    messages,
    max_tokens=45,
    strategy="last",
    token_counter=ChatOpenAI(model="gpt-4o"),
    include_system=True,
)
```



```output
[SystemMessage(content="you're a good assistant, you always respond with a joke."),
 HumanMessage(content='what do you call a speechless parrot')]
```


如果我们想允许分割消息的内容，可以指定 `allow_partial=True`：

```python
trim_messages(
    messages,
    max_tokens=56,
    strategy="last",
    token_counter=ChatOpenAI(model="gpt-4o"),
    include_system=True,
    allow_partial=True,
)
```



```output
[SystemMessage(content="you're a good assistant, you always respond with a joke."),
 AIMessage(content="\nWhy, he's probably chasing after the last cup of coffee in the office!"),
 HumanMessage(content='what do you call a speechless parrot')]
```


如果我们需要确保我们的第一条消息（不包括系统消息）始终是特定类型，可以指定 `start_on`：

```python
trim_messages(
    messages,
    max_tokens=60,
    strategy="last",
    token_counter=ChatOpenAI(model="gpt-4o"),
    include_system=True,
    start_on="human",
)
```



```output
[SystemMessage(content="you're a good assistant, you always respond with a joke."),
 HumanMessage(content='what do you call a speechless parrot')]
```

## 获取前 `max_tokens` 个标记

我们可以通过指定 `strategy="first"` 来执行获取 *前* `max_tokens` 的反向操作：


```python
trim_messages(
    messages,
    max_tokens=45,
    strategy="first",
    token_counter=ChatOpenAI(model="gpt-4o"),
)
```



```output
[SystemMessage(content="you're a good assistant, you always respond with a joke."),
 HumanMessage(content="i wonder why it's called langchain")]
```

## 编写自定义令牌计数器

我们可以编写一个自定义令牌计数器函数，该函数接受消息列表并返回一个整数。


```python
from typing import List

# pip install tiktoken
import tiktoken
from langchain_core.messages import BaseMessage, ToolMessage


def str_token_counter(text: str) -> int:
    enc = tiktoken.get_encoding("o200k_base")
    return len(enc.encode(text))


def tiktoken_counter(messages: List[BaseMessage]) -> int:
    """大致重现 https://github.com/openai/openai-cookbook/blob/main/examples/How_to_count_tokens_with_tiktoken.ipynb

    为了简单起见，只支持 str Message.contents。
    """
    num_tokens = 3  # 每个回复都以 <|start|>assistant<|message|> 开头
    tokens_per_message = 3
    tokens_per_name = 1
    for msg in messages:
        if isinstance(msg, HumanMessage):
            role = "user"
        elif isinstance(msg, AIMessage):
            role = "assistant"
        elif isinstance(msg, ToolMessage):
            role = "tool"
        elif isinstance(msg, SystemMessage):
            role = "system"
        else:
            raise ValueError(f"不支持的消息类型 {msg.__class__}")
        num_tokens += (
            tokens_per_message
            + str_token_counter(role)
            + str_token_counter(msg.content)
        )
        if msg.name:
            num_tokens += tokens_per_name + str_token_counter(msg.name)
    return num_tokens


trim_messages(
    messages,
    max_tokens=45,
    strategy="last",
    token_counter=tiktoken_counter,
)
```



```output
[AIMessage(content="Hmmm let me think.\n\nWhy, he's probably chasing after the last cup of coffee in the office!"),
 HumanMessage(content='what do you call a speechless parrot')]
```

## 链接

`trim_messages` 可以以命令式（如上所示）或声明式使用，使其易于与链中的其他组件组合。

```python
llm = ChatOpenAI(model="gpt-4o")

# 注意我们没有传入消息。这会创建
# 一个 RunnableLambda，它将消息作为输入
trimmer = trim_messages(
    max_tokens=45,
    strategy="last",
    token_counter=llm,
    include_system=True,
)

chain = trimmer | llm
chain.invoke(messages)
```

```output
AIMessage(content='A: A "Polly-gone"!', response_metadata={'token_usage': {'completion_tokens': 9, 'prompt_tokens': 32, 'total_tokens': 41}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_66b29dffce', 'finish_reason': 'stop', 'logprobs': None}, id='run-83e96ddf-bcaa-4f63-824c-98b0f8a0d474-0', usage_metadata={'input_tokens': 32, 'output_tokens': 9, 'total_tokens': 41})
```

查看 LangSmith 跟踪，我们可以看到在消息传递给模型之前，它们首先被修剪： https://smith.langchain.com/public/65af12c4-c24d-4824-90f0-6547566e59bb/r

单独查看修剪器，我们可以看到它是一个可以像所有 Runnable 一样被调用的 Runnable 对象：

```python
trimmer.invoke(messages)
```

```output
[SystemMessage(content="you're a good assistant, you always respond with a joke."),
 HumanMessage(content='what do you call a speechless parrot')]
```

## 使用 ChatMessageHistory

修剪消息在[处理聊天记录](/docs/how_to/message_history/)时尤其有用，因为聊天记录可能会变得非常长：

```python
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

chat_history = InMemoryChatMessageHistory(messages=messages[:-1])


def dummy_get_session_history(session_id):
    if session_id != "1":
        return InMemoryChatMessageHistory()
    return chat_history


llm = ChatOpenAI(model="gpt-4o")

trimmer = trim_messages(
    max_tokens=45,
    strategy="last",
    token_counter=llm,
    include_system=True,
)

chain = trimmer | llm
chain_with_history = RunnableWithMessageHistory(chain, dummy_get_session_history)
chain_with_history.invoke(
    [HumanMessage("what do you call a speechless parrot")],
    config={"configurable": {"session_id": "1"}},
)
```

```output
AIMessage(content='A "polly-no-wanna-cracker"!', response_metadata={'token_usage': {'completion_tokens': 10, 'prompt_tokens': 32, 'total_tokens': 42}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_5bf7397cd3', 'finish_reason': 'stop', 'logprobs': None}, id='run-054dd309-3497-4e7b-b22a-c1859f11d32e-0', usage_metadata={'input_tokens': 32, 'output_tokens': 10, 'total_tokens': 42})
```

查看 LangSmith 跟踪，我们可以看到我们检索了所有消息，但在消息传递给模型之前，它们被修剪为仅包含系统消息和最后一条人类消息： https://smith.langchain.com/public/17dd700b-9994-44ca-930c-116e00997315/r

## API 参考

有关所有参数的完整描述，请访问 API 参考： https://api.python.langchain.com/en/latest/messages/langchain_core.messages.utils.trim_messages.html
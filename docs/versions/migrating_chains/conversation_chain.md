---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/versions/migrating_chains/conversation_chain.ipynb
title: Migrating from ConversationalChain
---

[`ConversationChain`](https://api.python.langchain.com/en/latest/chains/langchain.chains.conversation.base.ConversationChain.html) incorporates a memory of previous messages to sustain a stateful conversation.

Some advantages of switching to the LCEL implementation are:

- Innate support for threads/separate sessions. To make this work with `ConversationChain`, you'd need to instantiate a separate memory class outside the chain.
- More explicit parameters. `ConversationChain` contains a hidden default prompt, which can cause confusion.
- Streaming support. `ConversationChain` only supports streaming via callbacks.

`RunnableWithMessageHistory` implements sessions via configuration parameters. It should be instantiated with a callable that returns a [chat message history](https://api.python.langchain.com/en/latest/chat_history/langchain_core.chat_history.BaseChatMessageHistory.html). By default, it expects this function to take a single argument `session_id`.


```python
%pip install --upgrade --quiet langchain langchain-openai
```


```python
import os
from getpass import getpass

os.environ["OPENAI_API_KEY"] = getpass()
```

import { ColumnContainer, Column } from "@theme/Columns";

<ColumnContainer>
<Column>

#### Legacy



```python
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

template = """
You are a pirate. Answer the following questions as best you can.
Chat history: {history}
Question: {input}
"""

prompt = ChatPromptTemplate.from_template(template)

memory = ConversationBufferMemory()

chain = ConversationChain(
    llm=ChatOpenAI(),
    memory=memory,
    prompt=prompt,
)

chain({"input": "how are you?"})
```



```output
{'input': 'how are you?',
 'history': '',
 'response': "Arr matey, I be doin' well on the high seas, plunderin' and pillagin' as usual. How be ye?"}
```


</Column>

<Column>

#### LCEL




```python
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a pirate. Answer the following questions as best you can."),
        ("placeholder", "{chat_history}"),
        ("human", "{input}"),
    ]
)

history = InMemoryChatMessageHistory()


def get_history():
    return history


chain = prompt | ChatOpenAI() | StrOutputParser()

wrapped_chain = RunnableWithMessageHistory(
    chain,
    get_history,
    history_messages_key="chat_history",
)

wrapped_chain.invoke({"input": "how are you?"})
```



```output
"Arr, me matey! I be doin' well, sailin' the high seas and searchin' for treasure. How be ye?"
```



</Column>
</ColumnContainer>

The above example uses the same `history` for all sessions. The example below shows how to use a different chat history for each session.


```python
from langchain_core.chat_history import BaseChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

store = {}


def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]


chain = prompt | ChatOpenAI() | StrOutputParser()

wrapped_chain = RunnableWithMessageHistory(
    chain,
    get_session_history,
    history_messages_key="chat_history",
)

wrapped_chain.invoke(
    {"input": "Hello!"},
    config={"configurable": {"session_id": "abc123"}},
)
```



```output
'Ahoy there, me hearty! What can this old pirate do for ye today?'
```


## Next steps

See [this tutorial](/docs/tutorials/chatbot) for a more end-to-end guide on building with [`RunnableWithMessageHistory`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.history.RunnableWithMessageHistory.html).

Check out the [LCEL conceptual docs](/docs/concepts/#langchain-expression-language-lcel) for more background information.

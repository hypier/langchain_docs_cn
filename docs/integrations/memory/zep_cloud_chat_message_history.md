---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/zep_cloud_chat_message_history.ipynb
---

# ZepCloudChatMessageHistory
> 回忆、理解并提取聊天记录中的数据。增强个性化的AI体验。

>[Zep](https://www.getzep.com) 是一个为AI助手应用提供长期记忆服务的工具。
> 使用Zep，您可以让AI助手回忆起过去的对话，无论距离多远，
> 同时减少幻觉、延迟和成本。

> 请参阅 [Zep Cloud安装指南](https://help.getzep.com/sdks) 和更多 [Zep Cloud Langchain示例](https://github.com/getzep/zep-python/tree/main/examples)

## 示例

本笔记本演示了如何使用 [Zep](https://www.getzep.com/) 来持久化聊天记录，并在您的链中使用 Zep Memory。



```python
from uuid import uuid4

from langchain_community.chat_message_histories import ZepCloudChatMessageHistory
from langchain_community.memory.zep_cloud_memory import ZepCloudMemory
from langchain_core.messages import AIMessage, HumanMessage
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables import (
    RunnableParallel,
)
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI

session_id = str(uuid4())  # This is a unique identifier for the session
```

提供您的 OpenAI 密钥


```python
import getpass

openai_key = getpass.getpass()
```

提供您的 Zep API 密钥。请参见 https://help.getzep.com/projects#api-keys



```python
zep_api_key = getpass.getpass()
```

将一些消息预加载到内存中。默认消息窗口为 4 条消息。我们希望超出此范围以演示自动总结。


```python
test_history = [
    {"role": "human", "content": "谁是奥克塔维亚·巴特勒？"},
    {
        "role": "ai",
        "content": (
            "奥克塔维亚·埃斯特尔·巴特勒（1947年6月22日 - 2006年2月24日）是一位美国"
            " 科幻小说作家。"
        ),
    },
    {"role": "human", "content": "她的哪本书被改编成电影？"},
    {
        "role": "ai",
        "content": (
            "奥克塔维亚·巴特勒作品中最著名的改编是 FX 系列"
            " 《亲缘关系》，该系列改编自她同名小说。"
        ),
    },
    {"role": "human", "content": "她的同时代人是谁？"},
    {
        "role": "ai",
        "content": (
            "奥克塔维亚·巴特勒的同时代人包括厄休拉·K·勒古恩、塞缪尔·R·"
            " 德拉尼和乔安娜·拉斯。"
        ),
    },
    {"role": "human", "content": "她获得了哪些奖项？"},
    {
        "role": "ai",
        "content": (
            "奥克塔维亚·巴特勒获得了雨果奖、星云奖和麦克阿瑟"
            " 奖学金。"
        ),
    },
    {
        "role": "human",
        "content": "还有哪些女性科幻作家值得我阅读？",
    },
    {
        "role": "ai",
        "content": "您可能想阅读厄休拉·K·勒古恩或乔安娜·拉斯。"
    },
    {
        "role": "human",
        "content": (
            "写一段关于巴特勒的书《播种者的寓言》的简短简介。它讲述了什么？"
        ),
    },
    {
        "role": "ai",
        "content": (
            "《播种者的寓言》是奥克塔维亚·巴特勒于1993年出版的科幻小说。"
            " 故事讲述了年轻女性劳伦·奥拉米娜的故事，她生活在一个反乌托邦的未来，"
            " 在这个未来中，由于环境灾难、贫困和暴力，社会已经崩溃。"
        ),
        "metadata": {"foo": "bar"},
    },
]

zep_memory = ZepCloudMemory(
    session_id=session_id,
    api_key=zep_api_key,
)

for msg in test_history:
    zep_memory.chat_memory.add_message(
        HumanMessage(content=msg["content"])
        if msg["role"] == "human"
        else AIMessage(content=msg["content"])
    )

import time

time.sleep(
    10
)  # Wait for the messages to be embedded and summarized, this happens asynchronously.
```

**MessagesPlaceholder** - 我们在这里使用变量名 chat_history。这将把聊天记录纳入提示中。
确保该变量名与 RunnableWithMessageHistory 链中的 history_messages_key 对齐，以实现无缝集成。

**question** 必须与 `RunnableWithMessageHistory` 链中的 input_messages_key 匹配。


```python
template = """请提供帮助并使用提供的上下文回答下面的问题：
    """
answer_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", template),
        MessagesPlaceholder(variable_name="chat_history"),
        ("user", "{question}"),
    ]
)
```

我们使用 RunnableWithMessageHistory 将 Zep 的聊天记录纳入我们的链中。激活链时，此类需要一个 session_id 作为参数。


```python
inputs = RunnableParallel(
    {
        "question": lambda x: x["question"],
        "chat_history": lambda x: x["chat_history"],
    },
)
chain = RunnableWithMessageHistory(
    inputs | answer_prompt | ChatOpenAI(openai_api_key=openai_key) | StrOutputParser(),
    lambda s_id: ZepCloudChatMessageHistory(
        session_id=s_id,  # This uniquely identifies the conversation, note that we are getting session id as chain configurable field
        api_key=zep_api_key,
        memory_type="perpetual",
    ),
    input_messages_key="question",
    history_messages_key="chat_history",
)
```


```python
chain.invoke(
    {
        "question": "这本书与当代社会面临的挑战有什么关系？"
    },
    config={"configurable": {"session_id": session_id}},
)
```
```output
父运行 622c6f75-3e4a-413d-ba20-558c1fea0d50 未找到运行 af12a4b1-e882-432d-834f-e9147465faf6。将其视为根运行。
```


```output
'"播种者的寓言" 与当代社会面临的挑战相关，因为它探讨了环境退化、经济不平等、社会动荡以及在混乱面前寻找希望和社区的主题。小说描绘了一个反乌托邦的未来，社会因环境和经济危机而崩溃，作为对我们当前社会和环境挑战潜在后果的警示故事。通过讨论气候变化、社会不公和技术对人类的影响等问题，奥克塔维亚·巴特勒的作品促使读者反思我们时代的紧迫问题，以及在建设更美好未来中韧性、同情心和集体行动的重要性。'
```
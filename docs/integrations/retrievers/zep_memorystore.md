---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/zep_memorystore.ipynb
---

# Zep 开源

## Retriever 示例 [Zep](https://docs.getzep.com/)

> 回忆、理解并提取聊天记录中的数据。为个性化的 AI 体验提供动力。

> [Zep](https://www.getzep.com) 是一个用于 AI 助手应用的长期记忆服务。
> 通过 Zep，您可以为 AI 助手提供回忆过去对话的能力，无论这些对话有多遥远，
> 同时还可以减少幻觉、延迟和成本。

> 对 Zep Cloud 感兴趣？请查看 [Zep Cloud 安装指南](https://help.getzep.com/sdks) 和 [Zep Cloud Retriever 示例](https://help.getzep.com/langchain/examples/rag-message-history-example)

## 开源安装与设置

> Zep 开源项目: [https://github.com/getzep/zep](https://github.com/getzep/zep)
> Zep 开源文档: [https://docs.getzep.com/](https://docs.getzep.com/)

## Retriever 示例

本笔记本演示如何使用 [Zep 长期记忆存储](https://getzep.github.io/) 搜索历史聊天消息记录。

我们将演示：

1. 将对话历史添加到 Zep 记忆存储中。
2. 对话历史的向量搜索：
    1. 对聊天消息进行相似性搜索
    2. 使用最大边际相关性重新排序聊天消息搜索
    3. 使用元数据过滤器过滤搜索
    4. 对聊天消息摘要进行相似性搜索
    5. 使用最大边际相关性重新排序摘要搜索

```python
import getpass
import time
from uuid import uuid4

from langchain.memory import ZepMemory
from langchain_core.messages import AIMessage, HumanMessage

# Set this to your Zep server URL
ZEP_API_URL = "http://localhost:8000"
```

### 初始化 Zep 聊天消息历史类并将聊天消息历史添加到内存存储中

**注意：** 与其他检索器不同，Zep 检索器返回的内容是会话/用户特定的。在实例化检索器时需要提供 `session_id`。

```python
# 提供您的 Zep API 密钥。请注意，这不是必需的。请参阅 https://docs.getzep.com/deployment/auth
AUTHENTICATE = False

zep_api_key = None
if AUTHENTICATE:
    zep_api_key = getpass.getpass()
```

```python
session_id = str(uuid4())  # 这是用户/会话的唯一标识符

# 初始化 Zep 内存类
zep_memory = ZepMemory(session_id=session_id, url=ZEP_API_URL, api_key=zep_api_key)
```

```python
# 预加载一些消息到内存中。默认消息窗口为 12 条消息。我们希望超过这个限制以演示自动摘要。
test_history = [
    {"role": "human", "content": "Octavia Butler 是谁？"},
    {
        "role": "ai",
        "content": (
            "Octavia Estelle Butler (1947年6月22日 – 2006年2月24日) 是一位美国"
            " 科幻小说作家。"
        ),
    },
    {"role": "human", "content": "她的哪些书被改编成了电影？"},
    {
        "role": "ai",
        "content": (
            "Octavia Butler 的作品中最著名的改编是 FX 系列"
            " 《亲属》，基于她同名小说改编。"
        ),
    },
    {"role": "human", "content": "她的同时代人是谁？"},
    {
        "role": "ai",
        "content": (
            "Octavia Butler 的同时代人包括 Ursula K. Le Guin、Samuel R."
            " Delany 和 Joanna Russ。"
        ),
    },
    {"role": "human", "content": "她获得了哪些奖项？"},
    {
        "role": "ai",
        "content": (
            "Octavia Butler 获得了雨果奖、星云奖和麦克阿瑟"
            " 奖学金。"
        ),
    },
    {
        "role": "human",
        "content": "还有哪些女性科幻作家我可能想读？",
    },
    {
        "role": "ai",
        "content": "你可能想读 Ursula K. Le Guin 或 Joanna Russ。",
    },
    {
        "role": "human",
        "content": (
            "写一段关于 Butler 的书《播种者的寓言》的简短摘要。它讲述了什么？"
        ),
    },
    {
        "role": "ai",
        "content": (
            "《播种者的寓言》是 Octavia Butler 于1993年出版的科幻小说。"
            " 它讲述了年轻女性 Lauren Olamina 的故事，她生活在一个由于"
            " 环境灾难、贫困和暴力而崩溃的反乌托邦未来。"
        ),
    },
    {"role": "human", "content": "这本书的背景是什么？"},
    {
        "role": "ai",
        "content": (
            "这本书的背景设定在2020年代的反乌托邦未来，社会因气候变化和"
            " 经济危机而崩溃。"
        ),
    },
    {"role": "human", "content": "谁是主角？"},
    {
        "role": "ai",
        "content": (
            "这本书的主角是 Lauren Olamina，一个拥有 '超同理心' 的年轻女性，"
            " 她能够感受到她目睹的痛苦和其他感受。"
        ),
    },
    {"role": "human", "content": "这本书的主要主题是什么？"},
    {
        "role": "ai",
        "content": (
            "这本书的主要主题是在剧烈的社会变革和崩溃面前的生存。"
            " 它还探讨了适应性、社区和人类改变的能力等主题。"
        ),
    },
    {"role": "human", "content": "什么是《播种者的寓言》？"},
    {
        "role": "ai",
        "content": (
            "《播种者的寓言》是一个圣经寓言，Butler 在书中将其作为"
            " 比喻。在这个寓言中，一个播种者撒下种子，其中一些落在肥沃的土壤上并生长，"
            " 而另一些则落在岩石地或荆棘中，无法生长。这个寓言用来说明在变化面前"
            " 接受能力和准备的重要性。"
        ),
    },
    {"role": "human", "content": "Butler 的写作风格如何？"},
    {
        "role": "ai",
        "content": (
            "Butler 的写作风格以清晰、直接和心理洞察力著称。"
            " 她的叙述通常涉及复杂、多样的角色，并探讨种族、性别和权力等主题。"
        ),
    },
    {"role": "human", "content": "她还写了哪些书？"},
    {
        "role": "ai",
        "content": (
            "除了《播种者的寓言》，Butler 还写了几部其他著名作品，包括《亲属》、"
            "《黎明》和《播种者的才能》。"
        ),
    },
]

for msg in test_history:
    zep_memory.chat_memory.add_message(
        HumanMessage(content=msg["content"])
        if msg["role"] == "human"
        else AIMessage(content=msg["content"])
    )

time.sleep(
    10
)  # 等待消息嵌入和摘要。速度取决于 OpenAI API 的延迟和您的速率限制。
```

### 使用 Zep Retriever 对 Zep 内存进行向量搜索

Zep 提供对历史对话内存的原生向量搜索。嵌入过程是自动进行的。

注意：消息的嵌入是异步发生的，因此第一次查询可能不会返回结果。后续查询将在嵌入生成后返回结果。

```python
from langchain_community.retrievers.zep import SearchScope, SearchType, ZepRetriever

zep_retriever = ZepRetriever(
    session_id=session_id,  # 确保在实例化 Retriever 时提供 session_id
    url=ZEP_API_URL,
    top_k=5,
    api_key=zep_api_key,
)

await zep_retriever.ainvoke("Who wrote Parable of the Sower?")
```

```output
[Document(page_content="What is the 'Parable of the Sower'?", metadata={'score': 0.9250216484069824, 'uuid': '4cbfb1c0-6027-4678-af43-1e18acb224bb', 'created_at': '2023-11-01T00:32:40.224256Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 34, 'Start': 13, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}]}}, 'token_count': 13}),
 Document(page_content='Parable of the Sower is a science fiction novel by Octavia Butler, published in 1993. It follows the story of Lauren Olamina, a young woman living in a dystopian future where society has collapsed due to environmental disasters, poverty, and violence.', metadata={'score': 0.8897348046302795, 'uuid': '3dd9f5ed-9dc9-4427-9da6-aba1b8278a5c', 'created_at': '2023-11-01T00:32:40.192527Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'GPE', 'Matches': [{'End': 20, 'Start': 15, 'Text': 'Sower'}], 'Name': 'Sower'}, {'Label': 'PERSON', 'Matches': [{'End': 65, 'Start': 51, 'Text': 'Octavia Butler'}], 'Name': 'Octavia Butler'}, {'Label': 'DATE', 'Matches': [{'End': 84, 'Start': 80, 'Text': '1993'}], 'Name': '1993'}, {'Label': 'PERSON', 'Matches': [{'End': 124, 'Start': 110, 'Text': 'Lauren Olamina'}], 'Name': 'Lauren Olamina'}], 'intent': 'Providing information'}}, 'token_count': 56}),
 Document(page_content="Write a short synopsis of Butler's book, Parable of the Sower. What is it about?", metadata={'score': 0.8856019973754883, 'uuid': '81761dcb-38f3-4686-a4f5-6cb1007eaf29', 'created_at': '2023-11-01T00:32:40.187543Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'metadata': {'system': {'entities': [{'Label': 'ORG', 'Matches': [{'End': 32, 'Start': 26, 'Text': 'Butler'}], 'Name': 'Butler'}, {'Label': 'WORK_OF_ART', 'Matches': [{'End': 61, 'Start': 41, 'Text': 'Parable of the Sower'}], 'Name': 'Parable of the Sower'}], 'intent': "The subject is asking for a brief summary of Butler's book, Parable of the Sower, and what it is about."}}, 'token_count': 23}),
 Document(page_content="The 'Parable of the Sower' is a biblical parable that Butler uses as a metaphor in the book. In the parable, a sower scatters seeds, some of which fall on fertile ground and grow, while others fall on rocky ground or among thorns and fail to grow. The parable is used to illustrate the importance of receptivity and preparedness in the face of change.", metadata={'score': 0.8781436681747437, 'uuid': '1a8c5f99-2fec-425d-bc37-176ab91e7080', 'created_at': '2023-11-01T00:32:40.22836Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 26, 'Start': 5, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}, {'Label': 'ORG', 'Matches': [{'End': 60, 'Start': 54, 'Text': 'Butler'}], 'Name': 'Butler'}]}}, 'token_count': 84}),
 Document(page_content="In addition to 'Parable of the Sower', Butler has written several other notable works, including 'Kindred', 'Dawn', and 'Parable of the Talents'.", metadata={'score': 0.8745182752609253, 'uuid': '45d8aa08-85ab-432f-8902-81712fe363b9', 'created_at': '2023-11-01T00:32:40.245081Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 37, 'Start': 16, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}, {'Label': 'ORG', 'Matches': [{'End': 45, 'Start': 39, 'Text': 'Butler'}], 'Name': 'Butler'}, {'Label': 'GPE', 'Matches': [{'End': 105, 'Start': 98, 'Text': 'Kindred'}], 'Name': 'Kindred'}, {'Label': 'WORK_OF_ART', 'Matches': [{'End': 144, 'Start': 121, 'Text': "Parable of the Talents'"}], 'Name': "Parable of the Talents'"}]}}, 'token_count': 39})]
```

我们还可以使用 Zep 同步 API 来检索结果：

```python
zep_retriever.invoke("Who wrote Parable of the Sower?")
```

```output
[Document(page_content="What is the 'Parable of the Sower'?", metadata={'score': 0.9250596761703491, 'uuid': '4cbfb1c0-6027-4678-af43-1e18acb224bb', 'created_at': '2023-11-01T00:32:40.224256Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 34, 'Start': 13, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}]}}, 'token_count': 13}),
 Document(page_content='Parable of the Sower is a science fiction novel by Octavia Butler, published in 1993. It follows the story of Lauren Olamina, a young woman living in a dystopian future where society has collapsed due to environmental disasters, poverty, and violence.', metadata={'score': 0.8897120952606201, 'uuid': '3dd9f5ed-9dc9-4427-9da6-aba1b8278a5c', 'created_at': '2023-11-01T00:32:40.192527Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'GPE', 'Matches': [{'End': 20, 'Start': 15, 'Text': 'Sower'}], 'Name': 'Sower'}, {'Label': 'PERSON', 'Matches': [{'End': 65, 'Start': 51, 'Text': 'Octavia Butler'}], 'Name': 'Octavia Butler'}, {'Label': 'DATE', 'Matches': [{'End': 84, 'Start': 80, 'Text': '1993'}], 'Name': '1993'}, {'Label': 'PERSON', 'Matches': [{'End': 124, 'Start': 110, 'Text': 'Lauren Olamina'}], 'Name': 'Lauren Olamina'}], 'intent': 'Providing information'}}, 'token_count': 56}),
 Document(page_content="Write a short synopsis of Butler's book, Parable of the Sower. What is it about?", metadata={'score': 0.885666012763977, 'uuid': '81761dcb-38f3-4686-a4f5-6cb1007eaf29', 'created_at': '2023-11-01T00:32:40.187543Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'metadata': {'system': {'entities': [{'Label': 'ORG', 'Matches': [{'End': 32, 'Start': 26, 'Text': 'Butler'}], 'Name': 'Butler'}, {'Label': 'WORK_OF_ART', 'Matches': [{'End': 61, 'Start': 41, 'Text': 'Parable of the Sower'}], 'Name': 'Parable of the Sower'}], 'intent': "The subject is asking for a brief summary of Butler's book, Parable of the Sower, and what it is about."}}, 'token_count': 23}),
 Document(page_content="The 'Parable of the Sower' is a biblical parable that Butler uses as a metaphor in the book. In the parable, a sower scatters seeds, some of which fall on fertile ground and grow, while others fall on rocky ground or among thorns and fail to grow. The parable is used to illustrate the importance of receptivity and preparedness in the face of change.", metadata={'score': 0.878172755241394, 'uuid': '1a8c5f99-2fec-425d-bc37-176ab91e7080', 'created_at': '2023-11-01T00:32:40.22836Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 26, 'Start': 5, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}, {'Label': 'ORG', 'Matches': [{'End': 60, 'Start': 54, 'Text': 'Butler'}], 'Name': 'Butler'}]}}, 'token_count': 84}),
 Document(page_content="In addition to 'Parable of the Sower', Butler has written several other notable works, including 'Kindred', 'Dawn', and 'Parable of the Talents'.", metadata={'score': 0.8745154142379761, 'uuid': '45d8aa08-85ab-432f-8902-81712fe363b9', 'created_at': '2023-11-01T00:32:40.245081Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 37, 'Start': 16, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}, {'Label': 'ORG', 'Matches': [{'End': 45, 'Start': 39, 'Text': 'Butler'}], 'Name': 'Butler'}, {'Label': 'GPE', 'Matches': [{'End': 105, 'Start': 98, 'Text': 'Kindred'}], 'Name': 'Kindred'}, {'Label': 'WORK_OF_ART', 'Matches': [{'End': 144, 'Start': 121, 'Text': "Parable of the Talents'"}], 'Name': "Parable of the Talents'"}]}}, 'token_count': 39})]
```

### 使用MMR（最大边际相关性）进行重新排序

Zep原生支持使用MMR进行结果的重新排序，并且通过SIMD加速。这对于去除结果中的冗余非常有用。

```python
zep_retriever = ZepRetriever(
    session_id=session_id,  # 确保在实例化检索器时提供session_id
    url=ZEP_API_URL,
    top_k=5,
    api_key=zep_api_key,
    search_type=SearchType.mmr,
    mmr_lambda=0.5,
)

await zep_retriever.ainvoke("Who wrote Parable of the Sower?")
```

```output
[Document(page_content="What is the 'Parable of the Sower'?", metadata={'score': 0.9250596761703491, 'uuid': '4cbfb1c0-6027-4678-af43-1e18acb224bb', 'created_at': '2023-11-01T00:32:40.224256Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 34, 'Start': 13, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}]}}, 'token_count': 13}),
 Document(page_content='What other books has she written?', metadata={'score': 0.77488774061203, 'uuid': '1b3c5079-9cab-46f3-beae-fb56c572e0fd', 'created_at': '2023-11-01T00:32:40.240135Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'token_count': 9}),
 Document(page_content="In addition to 'Parable of the Sower', Butler has written several other notable works, including 'Kindred', 'Dawn', and 'Parable of the Talents'.", metadata={'score': 0.8745154142379761, 'uuid': '45d8aa08-85ab-432f-8902-81712fe363b9', 'created_at': '2023-11-01T00:32:40.245081Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 37, 'Start': 16, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}, {'Label': 'ORG', 'Matches': [{'End': 45, 'Start': 39, 'Text': 'Butler'}], 'Name': 'Butler'}, {'Label': 'GPE', 'Matches': [{'End': 105, 'Start': 98, 'Text': 'Kindred'}], 'Name': 'Kindred'}, {'Label': 'WORK_OF_ART', 'Matches': [{'End': 144, 'Start': 121, 'Text': "Parable of the Talents'"}], 'Name': "Parable of the Talents'"}]}}, 'token_count': 39}),
 Document(page_content='Parable of the Sower is a science fiction novel by Octavia Butler, published in 1993. It follows the story of Lauren Olamina, a young woman living in a dystopian future where society has collapsed due to environmental disasters, poverty, and violence.', metadata={'score': 0.8897120952606201, 'uuid': '3dd9f5ed-9dc9-4427-9da6-aba1b8278a5c', 'created_at': '2023-11-01T00:32:40.192527Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'GPE', 'Matches': [{'End': 20, 'Start': 15, 'Text': 'Sower'}], 'Name': 'Sower'}, {'Label': 'PERSON', 'Matches': [{'End': 65, 'Start': 51, 'Text': 'Octavia Butler'}], 'Name': 'Octavia Butler'}, {'Label': 'DATE', 'Matches': [{'End': 84, 'Start': 80, 'Text': '1993'}], 'Name': '1993'}, {'Label': 'PERSON', 'Matches': [{'End': 124, 'Start': 110, 'Text': 'Lauren Olamina'}], 'Name': 'Lauren Olamina'}], 'intent': 'Providing information'}}, 'token_count': 56}),
 Document(page_content='Who is the protagonist?', metadata={'score': 0.7858647704124451, 'uuid': 'ee514b37-a0b0-4d24-b0c9-3e9f8ad9d52d', 'created_at': '2023-11-01T00:32:40.203891Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'metadata': {'system': {'intent': 'The subject is asking about the identity of the protagonist in a specific context, such as a story, movie, or game.'}}, 'token_count': 7})]
```

### 使用元数据过滤器来细化搜索结果

Zep 支持通过元数据过滤结果。这对于按实体类型或其他元数据过滤结果非常有用。

更多信息请见： https://docs.getzep.com/sdk/search_query/


```python
filter = {"where": {"jsonpath": '$[*] ? (@.Label == "WORK_OF_ART")'}}

await zep_retriever.ainvoke("Who wrote Parable of the Sower?", metadata=filter)
```



```output
[Document(page_content="What is the 'Parable of the Sower'?", metadata={'score': 0.9251098036766052, 'uuid': '4cbfb1c0-6027-4678-af43-1e18acb224bb', 'created_at': '2023-11-01T00:32:40.224256Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 34, 'Start': 13, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}]}}, 'token_count': 13}),
 Document(page_content='What other books has she written?', metadata={'score': 0.7747920155525208, 'uuid': '1b3c5079-9cab-46f3-beae-fb56c572e0fd', 'created_at': '2023-11-01T00:32:40.240135Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'token_count': 9}),
 Document(page_content="In addition to 'Parable of the Sower', Butler has written several other notable works, including 'Kindred', 'Dawn', and 'Parable of the Talents'.", metadata={'score': 0.8745266795158386, 'uuid': '45d8aa08-85ab-432f-8902-81712fe363b9', 'created_at': '2023-11-01T00:32:40.245081Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 37, 'Start': 16, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}, {'Label': 'ORG', 'Matches': [{'End': 45, 'Start': 39, 'Text': 'Butler'}], 'Name': 'Butler'}, {'Label': 'GPE', 'Matches': [{'End': 105, 'Start': 98, 'Text': 'Kindred'}], 'Name': 'Kindred'}, {'Label': 'WORK_OF_ART', 'Matches': [{'End': 144, 'Start': 121, 'Text': "Parable of the Talents'"}], 'Name': "Parable of the Talents'"}]}}, 'token_count': 39}),
 Document(page_content='Parable of the Sower is a science fiction novel by Octavia Butler, published in 1993. It follows the story of Lauren Olamina, a young woman living in a dystopian future where society has collapsed due to environmental disasters, poverty, and violence.', metadata={'score': 0.8897372484207153, 'uuid': '3dd9f5ed-9dc9-4427-9da6-aba1b8278a5c', 'created_at': '2023-11-01T00:32:40.192527Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'GPE', 'Matches': [{'End': 20, 'Start': 15, 'Text': 'Sower'}], 'Name': 'Sower'}, {'Label': 'PERSON', 'Matches': [{'End': 65, 'Start': 51, 'Text': 'Octavia Butler'}], 'Name': 'Octavia Butler'}, {'Label': 'DATE', 'Matches': [{'End': 84, 'Start': 80, 'Text': '1993'}], 'Name': '1993'}, {'Label': 'PERSON', 'Matches': [{'End': 124, 'Start': 110, 'Text': 'Lauren Olamina'}], 'Name': 'Lauren Olamina'}], 'intent': 'Providing information'}}, 'token_count': 56}),
 Document(page_content='Who is the protagonist?', metadata={'score': 0.7858127355575562, 'uuid': 'ee514b37-a0b0-4d24-b0c9-3e9f8ad9d52d', 'created_at': '2023-11-01T00:32:40.203891Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'metadata': {'system': {'intent': 'The subject is asking about the identity of the protagonist in a specific context, such as a story, movie, or game.'}}, 'token_count': 7})]
```

### 使用MMR重新排序进行摘要搜索

Zep自动生成聊天消息的摘要。这些摘要可以通过Zep检索器进行搜索。由于摘要是对对话的提炼，它们更有可能与您的搜索查询匹配，并为LLM提供丰富而简洁的上下文。

连续的摘要可能包含相似的内容，Zep的相似性搜索返回最高匹配的结果，但多样性较少。MMR重新排序结果，以确保您填充到提示中的摘要既相关且每个摘要都为LLM提供额外的信息。

```python
zep_retriever = ZepRetriever(
    session_id=session_id,  # Ensure that you provide the session_id when instantiating the Retriever
    url=ZEP_API_URL,
    top_k=3,
    api_key=zep_api_key,
    search_scope=SearchScope.summary,
    search_type=SearchType.mmr,
    mmr_lambda=0.5,
)

await zep_retriever.ainvoke("Who wrote Parable of the Sower?")
```

```output
[Document(page_content='The human asks about Octavia Butler and the AI informs them that she was an American science fiction author. The human\nasks which of her books were made into movies and the AI mentions the FX series Kindred. The human then asks about her\ncontemporaries and the AI lists Ursula K. Le Guin, Samuel R. Delany, and Joanna Russ. The human also asks about the awards\nshe won and the AI mentions the Hugo Award, the Nebula Award, and the MacArthur Fellowship. The human asks about other women sci-fi writers to read and the AI suggests Ursula K. Le Guin and Joanna Russ. The human then asks for a synopsis of Butler\'s book "Parable of the Sower" and the AI describes it.', metadata={'score': 0.7882999777793884, 'uuid': '3c95a29a-52dc-4112-b8a7-e6b1dc414d45', 'created_at': '2023-11-01T00:32:47.76449Z', 'token_count': 155}),
 Document(page_content='The human asks about Octavia Butler. The AI informs the human that Octavia Estelle Butler was an American science \nfiction author. The human then asks which books of hers were made into movies and the AI mentions the FX series Kindred, \nbased on her novel of the same name.', metadata={'score': 0.7407922744750977, 'uuid': '0e027f4d-d71f-42ae-977f-696b8948b8bf', 'created_at': '2023-11-01T00:32:41.637098Z', 'token_count': 59}),
 Document(page_content='The human asks about Octavia Butler and the AI informs them that she was an American science fiction author. The human\nasks which of her books were made into movies and the AI mentions the FX series Kindred. The human then asks about her\ncontemporaries and the AI lists Ursula K. Le Guin, Samuel R. Delany, and Joanna Russ. The human also asks about the awards\nshe won and the AI mentions the Hugo Award, the Nebula Award, and the MacArthur Fellowship.', metadata={'score': 0.7436535358428955, 'uuid': 'b3500d1b-1a78-4aef-9e24-6b196cfa83cb', 'created_at': '2023-11-01T00:32:44.24744Z', 'token_count': 104})]
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)
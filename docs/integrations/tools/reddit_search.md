---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/reddit_search.ipynb
---

# Reddit 搜索

在这个笔记本中，我们学习 Reddit 搜索工具的工作原理。  
首先确保你已经通过以下命令安装了 praw：  

```python
%pip install --upgrade --quiet  praw
```

然后你需要设置合适的 API 密钥和环境变量。你需要创建一个 Reddit 用户账户并获取凭证。因此，请访问 https://www.reddit.com 并注册一个 Reddit 用户账户。  
然后通过访问 https://www.reddit.com/prefs/apps 创建一个应用来获取你的凭证。  
你应该从创建应用中获得你的 client_id 和 secret。现在，你可以将这些字符串粘贴到 client_id 和 client_secret 变量中。  
注意：你可以为 user_agent 输入任何字符串  

```python
client_id = ""
client_secret = ""
user_agent = ""
```

```python
from langchain_community.tools.reddit_search.tool import RedditSearchRun
from langchain_community.utilities.reddit_search import RedditSearchAPIWrapper

search = RedditSearchRun(
    api_wrapper=RedditSearchAPIWrapper(
        reddit_client_id=client_id,
        reddit_client_secret=client_secret,
        reddit_user_agent=user_agent,
    )
)
```

然后你可以设置你的查询，例如你想查询哪个 subreddit，想返回多少个帖子，结果的排序方式等等。  

```python
from langchain_community.tools.reddit_search.tool import RedditSearchSchema

search_params = RedditSearchSchema(
    query="beginner", sort="new", time_filter="week", subreddit="python", limit="2"
)
```

最后运行搜索并获取结果  

```python
result = search.run(tool_input=search_params.dict())
```

```python
print(result)
```

这是打印结果的一个示例。  
注意：根据 subreddit 中最新帖子的不同，你可能会得到不同的输出，但格式应该相似。  

> Searching r/python found 2 posts:  
> Post Title: 'Setup Github Copilot in Visual Studio Code'  
> User: Feisty-Recording-715  
> Subreddit: r/Python:  
>                     Text body: 🛠️ This tutorial is perfect for beginners looking to strengthen their understanding of version control or for experienced developers seeking a quick reference for GitHub setup in Visual Studio Code.  
>  
> 🎓 By the end of this video, you'll be equipped with the skills to confidently manage your codebase, collaborate with others, and contribute to open-source projects on GitHub.  
>  
>  
> Video link: https://youtu.be/IdT1BhrSfdo?si=mV7xVpiyuhlD8Zrw  
>  
> Your feedback is welcome  
>                     Post URL: https://www.reddit.com/r/Python/comments/1823wr7/setup_github_copilot_in_visual_studio_code/  
>                     Post Category: N/A.  
>                     Score: 0  
>  
> Post Title: 'A Chinese Checkers game made with pygame and PySide6, with custom bots support'  
> User: HenryChess  
> Subreddit: r/Python:  
>                     Text body: GitHub link: https://github.com/henrychess/pygame-chinese-checkers  
>  
> I'm not sure if this counts as beginner or intermediate. I think I'm still in the beginner zone, so I flair it as beginner.  
>  
> This is a Chinese Checkers (aka Sternhalma) game for 2 to 3 players. The bots I wrote are easy to beat, as they're mainly for debugging the game logic part of the code. However, you can write up your own custom bots. There is a guide at the github page.  
>                     Post URL: https://www.reddit.com/r/Python/comments/181xq0u/a_chinese_checkers_game_made_with_pygame_and/  
>                     Post Category: N/A.  
>                     Score: 1

## 使用带有代理链的工具

Reddit 搜索功能也作为一个多输入工具提供。在这个例子中，我们适应了 [文档中的现有代码](https://python.langchain.com/v0.1/docs/modules/memory/agent_with_memory/)，并使用 ChatOpenAI 创建一个带有记忆的代理链。这个代理链能够从 Reddit 中提取信息，并利用这些帖子来回应后续输入。

要运行此示例，请添加您的 Reddit API 访问信息，并从 [OpenAI API](https://help.openai.com/en/articles/4936850-where-do-i-find-my-api-key) 获取一个 OpenAI 密钥。

```python
# Adapted code from /docs/modules/agents/how_to/sharedmemory_for_tools

from langchain.agents import AgentExecutor, StructuredChatAgent
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory, ReadOnlySharedMemory
from langchain_community.tools.reddit_search.tool import RedditSearchRun
from langchain_community.utilities.reddit_search import RedditSearchAPIWrapper
from langchain_core.prompts import PromptTemplate
from langchain_core.tools import Tool
from langchain_openai import ChatOpenAI

# Provide keys for Reddit
client_id = ""
client_secret = ""
user_agent = ""
# Provide key for OpenAI
openai_api_key = ""

template = """This is a conversation between a human and a bot:

{chat_history}

Write a summary of the conversation for {input}:
"""

prompt = PromptTemplate(input_variables=["input", "chat_history"], template=template)
memory = ConversationBufferMemory(memory_key="chat_history")

prefix = """Have a conversation with a human, answering the following questions as best you can. You have access to the following tools:"""
suffix = """Begin!"

{chat_history}
Question: {input}
{agent_scratchpad}"""

tools = [
    RedditSearchRun(
        api_wrapper=RedditSearchAPIWrapper(
            reddit_client_id=client_id,
            reddit_client_secret=client_secret,
            reddit_user_agent=user_agent,
        )
    )
]

prompt = StructuredChatAgent.create_prompt(
    prefix=prefix,
    tools=tools,
    suffix=suffix,
    input_variables=["input", "chat_history", "agent_scratchpad"],
)

llm = ChatOpenAI(temperature=0, openai_api_key=openai_api_key)

llm_chain = LLMChain(llm=llm, prompt=prompt)
agent = StructuredChatAgent(llm_chain=llm_chain, verbose=True, tools=tools)
agent_chain = AgentExecutor.from_agent_and_tools(
    agent=agent, verbose=True, memory=memory, tools=tools
)

# Answering the first prompt requires usage of the Reddit search tool.
agent_chain.run(input="What is the newest post on r/langchain for the week?")
# Answering the subsequent prompt uses memory.
agent_chain.run(input="Who is the author of the post?")
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)
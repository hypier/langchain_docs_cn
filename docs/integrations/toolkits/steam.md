---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/steam.ipynb
---

# Steam 游戏推荐与游戏详情

>[Steam (维基百科)](https://en.wikipedia.org/wiki/Steam_(service)) 是由 `Valve Corporation` 开发的视频游戏数字发行服务和商店。它为 Valve 的游戏自动提供更新，并扩展到分发第三方游戏。`Steam` 提供各种功能，如与 Valve 反作弊措施的游戏服务器匹配、社交网络和游戏流媒体服务。

>[Steam](https://store.steampowered.com/about/) 是玩、讨论和创造游戏的终极目的地。

Steam 工具包有两个工具：
- `游戏详情`
- `推荐游戏`

本笔记本提供了使用 Steam API 与 LangChain 的操作指南，以根据您当前的 Steam 游戏库存检索 Steam 游戏推荐，或收集您提供的一些 Steam 游戏的信息。

## 设置

我们需要安装两个 Python 库。

## 导入


```python
%pip install --upgrade --quiet  python-steam-api python-decouple
```

## 分配环境变量
要使用此工具包，请准备好您的 OpenAI API 密钥、Steam API 密钥（从 [这里](https://steamcommunity.com/dev/apikey)）和您自己的 SteamID。一旦您获得了 Steam API 密钥，您可以将其作为环境变量输入到下面。
该工具包将读取 "STEAM_KEY" API 密钥作为环境变量以进行身份验证，因此请在此处设置它们。您还需要设置您的 "OPENAI_API_KEY" 和 "STEAM_ID"。


```python
import os

os.environ["STEAM_KEY"] = "xyz"
os.environ["STEAM_ID"] = "123"
os.environ["OPENAI_API_KEY"] = "abc"
```

## 初始化：
初始化 LLM、SteamWebAPIWrapper、SteamToolkit，以及最重要的 langchain 代理来处理您的查询！

## 示例


```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits.steam.toolkit import SteamToolkit
from langchain_community.utilities.steam import SteamWebAPIWrapper
from langchain_openai import OpenAI
```


```python
llm = OpenAI(temperature=0)
Steam = SteamWebAPIWrapper()
toolkit = SteamToolkit.from_steam_api_wrapper(Steam)
agent = initialize_agent(
    toolkit.get_tools(), llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```


```python
out = agent("can you give the information about the game Terraria")
print(out)
```
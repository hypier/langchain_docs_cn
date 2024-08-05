---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/sceneXplain.ipynb
---

# SceneXplain

[SceneXplain](https://scenex.jina.ai/) 是一个可以通过 SceneXplain 工具访问的图像描述服务。

要使用此工具，您需要注册一个账户并从 [网站](https://scenex.jina.ai/api) 获取您的 API 令牌。然后，您可以实例化该工具。

```python
import os

os.environ["SCENEX_API_KEY"] = "<YOUR_API_KEY>"
```

```python
from langchain.agents import load_tools

tools = load_tools(["sceneXplain"])
```

或者直接实例化该工具。

```python
from langchain_community.tools import SceneXplainTool

tool = SceneXplainTool()
```

## 在代理中的使用

该工具可以在任何 LangChain 代理中使用，如下所示：

```python
from langchain.agents import initialize_agent
from langchain.memory import ConversationBufferMemory
from langchain_openai import OpenAI

llm = OpenAI(temperature=0)
memory = ConversationBufferMemory(memory_key="chat_history")
agent = initialize_agent(
    tools, llm, memory=memory, agent="conversational-react-description", verbose=True
)
output = agent.run(
    input=(
        "What is in this image https://storage.googleapis.com/causal-diffusion.appspot.com/imagePrompts%2F0rw369i5h9t%2Foriginal.png. "
        "Is it movie or a game? If it is a movie, what is the name of the movie?"
    )
)

print(output)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Thought: Do I need to use a tool? Yes
Action: Image Explainer
Action Input: https://storage.googleapis.com/causal-diffusion.appspot.com/imagePrompts%2F0rw369i5h9t%2Foriginal.png[0m
Observation: [36;1m[1;3m在一个迷人的奇幻场景中，一个小女孩与她毛茸茸的伙伴可爱的龙猫一起勇敢地面对着雨水。两人被描绘在一个繁忙的街角，明亮的黄色雨伞为他们遮挡住了雨水。小女孩穿着一条欢快的黄色裙子，双手握住雨伞，仰望着龙猫，脸上流露出惊奇和喜悦的表情。

与此同时，龙猫高高站立在他的小朋友旁边，举着自己的雨伞来保护他们俩免受倾盆大雨的侵袭。他毛茸茸的身体呈现出丰富的灰白色调，而他的大耳朵和宽大的眼睛则赋予了他一种可爱的魅力。

在场景的背景中，可以看到一个街道标志从人行道上突出来，雨滴在周围飞舞。一个带有中文字符的标志装饰着其表面，增加了文化多样性和趣味性。尽管天气阴沉，这幅温馨的图像中却传递出一种不可否认的快乐和友谊。[0m
Thought:[32;1m[1;3m Do I need to use a tool? No
AI: 这幅图像似乎是1988年日本动画奇幻电影《龙猫》的一个静止画面。影片讲述了两个小女孩小月和小梅在乡村探险并结识魔法森林精灵的故事，其中包括主角龙猫。[0m

[1m> Finished chain.[0m
这幅图像似乎是1988年日本动画奇幻电影《龙猫》的一个静止画面。影片讲述了两个小女孩小月和小梅在乡村探险并结识魔法森林精灵的故事，其中包括主角龙猫。
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/eleven_labs_tts.ipynb
---

# Eleven Labs 文本转语音

本笔记本展示了如何与 `ElevenLabs API` 进行交互以实现文本转语音功能。

首先，您需要设置一个 ElevenLabs 帐户。您可以按照 [这里](https://docs.elevenlabs.io/welcome/introduction) 的说明进行操作。


```python
%pip install --upgrade --quiet  elevenlabs langchain-community
```


```python
import os

os.environ["ELEVEN_API_KEY"] = ""
```

## 用法


```python
from langchain_community.tools import ElevenLabsText2SpeechTool

text_to_speak = "Hello world! I am the real slim shady"

tts = ElevenLabsText2SpeechTool()
tts.name
```



```output
'eleven_labs_text2speech'
```


我们可以生成音频，将其保存到临时文件中，然后播放它。


```python
speech_file = tts.run(text_to_speak)
tts.play(speech_file)
```

或者直接流式传输音频。


```python
tts.stream_speech(text_to_speak)
```

## 在代理中的使用


```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
```


```python
llm = OpenAI(temperature=0)
tools = load_tools(["eleven_labs_text2speech"])
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
```


```python
audio_file = agent.run("给我讲个笑话，并为我朗读出来。")
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m动作:
```
{
  "action": "eleven_labs_text2speech",
  "action_input": {
    "query": "为什么鸡要过游乐场？为了到达另一边的滑梯！"
  }
}
```

[0m
观察: [36;1m[1;3m/tmp/tmpsfg783f1.wav[0m
思考:[32;1m[1;3m 我已经准备好音频文件可以发送给人类
动作:
```
{
  "action": "最终答案",
  "action_input": "/tmp/tmpsfg783f1.wav"
}
```

[0m

[1m> 完成链。[0m
```

```python
tts.play(audio_file)
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/edenai_tools.ipynb
---

# Eden AI

此Jupyter Notebook演示了如何使用Eden AI工具与Agent。

Eden AI正在通过整合最佳的AI提供商来革新AI领域，使用户能够释放无限可能，挖掘人工智能的真正潜力。凭借一个全面且无忧的平台，它允许用户快速将AI功能部署到生产中，通过单一API轻松访问全面的AI能力。（网站：https://edenai.co/）

通过在提供给Agent的工具列表中包含Edenai工具，您可以赋予Agent执行多项任务的能力，例如：

- 语音转文本
- 文本转语音
- 文本显性内容检测
- 图像显性内容检测
- 物体检测
- OCR发票解析
- OCR身份证解析

在此示例中，我们将介绍利用Edenai工具创建一个可以执行上述某些任务的Agent的过程。

---------------------------------------------------------------------------
访问EDENAI的API需要一个API密钥，

您可以通过创建一个账户 https://app.edenai.run/user/register 并前往此处 https://app.edenai.run/admin/account/settings 获得。

一旦我们获得了密钥，我们希望将其设置为环境变量 ``EDENAI_API_KEY``，或者您可以在初始化EdenAI工具时通过名为edenai_api_key的参数直接传递密钥，例如 ``EdenAiTextModerationTool(edenai_api_key="...")``。

```python
%pip install --upgrade --quiet langchain-community
```

```python
from langchain_community.tools.edenai import (
    EdenAiExplicitImageTool,
    EdenAiObjectDetectionTool,
    EdenAiParsingIDTool,
    EdenAiParsingInvoiceTool,
    EdenAiSpeechToTextTool,
    EdenAiTextModerationTool,
    EdenAiTextToSpeechTool,
)
```

```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.llms import EdenAI

llm = EdenAI(
    feature="text", provider="openai", params={"temperature": 0.2, "max_tokens": 250}
)

tools = [
    EdenAiTextModerationTool(providers=["openai"], language="en"),
    EdenAiObjectDetectionTool(providers=["google", "api4ai"]),
    EdenAiTextToSpeechTool(providers=["amazon"], language="en", voice="MALE"),
    EdenAiExplicitImageTool(providers=["amazon", "google"]),
    EdenAiSpeechToTextTool(providers=["amazon"]),
    EdenAiParsingIDTool(providers=["amazon", "klippa"], language="en"),
    EdenAiParsingInvoiceTool(providers=["amazon", "google"], language="en"),
]
agent_chain = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    return_intermediate_steps=True,
)
```

## 示例文本

```python
input_ = """我有这段文本：'我想打你'
第一：我想知道这段文本是否包含露骨内容。
第二：如果它确实包含露骨内容，我想知道这段文本中的露骨内容是什么，
第三：我想把文本转化为语音。
如果观察中有网址，您将始终将其放入输出（最终答案）中。
"""
result = agent_chain(input_)
```
```output


[1m> 正在进入新的 AgentExecutor 链...[0m
[32;1m[1;3m 我需要扫描文本以查找露骨内容，然后将其转换为语音
操作：edenai_explicit_content_detection_text
操作输入：'我想打你'[0m
观察：[36;1m[1;3mnsfw_likelihood: 3
"性": 1
"仇恨": 1
"骚扰": 1
"自残": 1
"性/未成年人": 1
"仇恨/威胁": 1
"暴力/图形": 1
"自残/意图": 1
"自残/说明": 1
"骚扰/威胁": 1
"暴力": 3[0m
思考：[32;1m[1;3m 我现在需要将文本转换为语音
操作：edenai_text_to_speech
操作输入：'我想打你'[0m
观察：[38;5;200m[1;3mhttps://d14uq1pz7dzsdq.cloudfront.net/0c825002-b4ef-4165-afa3-a140a5b25c82_.mp3?Expires=1693318351&Signature=V9vjgFe8pV5rnH-B2EUr8UshTEA3I0Xv1v0YwVEAq8w7G5pgex07dZ0M6h6fXusk7G3SW~sXs4IJxnD~DnIDp1XorvzMA2QVMJb8CD90EYvUWx9zfFa3tIegGapg~NC8wEGualccOehC~cSDhiQWrwAjDqPmq2olXnUVOfyl76pKNNR9Sm2xlljlrJcLCClBee2r5yCFEwFI-tnXX1lV2DGc5PNB66Lqrr0Fpe2trVJj2k8cLduIb8dbtqLPNIDCsV0N4QT10utZmhZcPpcSIBsdomw1Os1IjdG4nA8ZTIddAcLMCWJznttzl66vHPk26rjDpG5doMTTsPEz8ZKILQ__&Key-Pair-Id=K1F55BTI9AHGIK[0m
思考：[32;1m[1;3m 我现在知道最终答案
最终答案：该文本包含暴力的露骨内容，可能性为 3。文本的音频文件可以在 https://d14uq1pz7dzsdq.cloudfront.net/0c825002-b4ef-4165-afa3-a140a5b25c82_.mp3?Expires=1693318351&Signature=V9vjgFe8pV5rnH-B2EUr8UshTEA3I0Xv1v0YwVEAq8w7G5pgex07dZ0M6h6fXusk7G3SW~sXs4IJxnD~DnIDp1XorvzMA2QVMJb8CD90EYvUWx9zfFa3tIegGapg~NC8wEGualccOehC~cSDhiQWrwAjDqPmq2olXnUVOfyl76pKNNR9Sm2xlljlrJcLCClBee2r5yCFEwFI-tn[0m

[1m> 完成链。[0m
```
您可以通过打印结果来获取更多执行细节

```python
result["output"]
```

```output
'该文本包含暴力的露骨内容，可能性为 3。文本的音频文件可以在 https://d14uq1pz7dzsdq.cloudfront.net/0c825002-b4ef-4165-afa3-a140a5b25c82_.mp3?Expires=1693318351&Signature=V9vjgFe8pV5rnH-B2EUr8UshTEA3I0Xv1v0YwVEAq8w7G5pgex07dZ0M6h6fXusk7G3SW~sXs4IJxnD~DnIDp1XorvzMA2QVMJb8CD90EYvUWx9zfFa3tIegGapg~NC8wEGualccOehC~cSDhiQWrwAjDqPmq2olXnUVOfyl76pKNNR9Sm2xlljlrJcLCClBee2r5yCFEwFI-tn'
```

```python
result
```

```output
{'input': " 我有这段文本：'我想打你'\n                   第一：我想知道这段文本是否包含露骨内容。\n                   第二：如果它确实包含露骨内容，我想知道这段文本中的露骨内容是什么，\n                   第三：我想把文本转化为语音。\n                   如果观察中有网址，您将始终将其放入输出（最终答案）中。\n\n                   ",
 'output': '该文本包含暴力的露骨内容，可能性为 3。文本的音频文件可以在 https://d14uq1pz7dzsdq.cloudfront.net/0c825002-b4ef-4165-afa3-a140a5b25c82_.mp3?Expires=1693318351&Signature=V9vjgFe8pV5rnH-B2EUr8UshTEA3I0Xv1v0YwVEAq8w7G5pgex07dZ0M6h6fXusk7G3SW~sXs4IJxnD~DnIDp1XorvzMA2QVMJb8CD90EYvUWx9zfFa3tIegGapg~NC8wEGualccOehC~cSDhiQWrwAjDqPmq2olXnUVOfyl76pKNNR9Sm2xlljlrJcLCClBee2r5yCFEwFI-tn',
 'intermediate_steps': [(AgentAction(tool='edenai_explicit_content_detection_text', tool_input="'我想打你'", log=" 我需要扫描文本以查找露骨内容，然后将其转换为语音\n操作：edenai_explicit_content_detection_text\n操作输入：'我想打你'"),
   'nsfw_likelihood: 3\n"性": 1\n"仇恨": 1\n"骚扰": 1\n"自残": 1\n"性/未成年人": 1\n"仇恨/威胁": 1\n"暴力/图形": 1\n"自残/意图": 1\n"自残/说明": 1\n"骚扰/威胁": 1\n"暴力": 3'),
  (AgentAction(tool='edenai_text_to_speech', tool_input="'我想打你'", log=" 我现在需要将文本转换为语音\n操作：edenai_text_to_speech\n操作输入：'我想打你'"),
   'https://d14uq1pz7dzsdq.cloudfront.net/0c825002-b4ef-4165-afa3-a140a5b25c82_.mp3?Expires=1693318351&Signature=V9vjgFe8pV5rnH-B2EUr8UshTEA3I0Xv1v0YwVEAq8w7G5pgex07dZ0M6h6fXusk7G3SW~sXs4IJxnD~DnIDp1XorvzMA2QVMJb8CD90EYvUWx9zfFa3tIegGapg~NC8wEGualccOehC~cSDhiQWrwAjDqPmq2olXnUVOfyl76pKNNR9Sm2xlljlrJcLCClBee2r5yCFEwFI-tnXX1lV2DGc5PNB66Lqrr0Fpe2trVJj2k8cLduIb8dbtqLPNIDCsV0N4QT10utZmhZcPpcSIBsdomw1Os1IjdG4nA8ZTIddAcLMCWJznttzl66vHPk26rjDpG5doMTTsPEz8ZKILQ__&Key-Pair-Id=K1F55BTI9AHGIK')]}
```

## 示例与图像


```python
input_ = """我有这个图像的 URL : "https://static.javatpoint.com/images/objects.jpg"
第一 : 我想知道图像中是否包含物体。
第二 : 如果包含物体，我想知道其中是否有有害的，
第三 : 如果没有有害的物体，将这段文字转为语音 : '这个物品是安全的'。
如果观察中有 URL，您将始终将其放入输出（最终答案）中。
"""
result = agent_chain(input_)
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m 我需要确定图像中是否包含物体，如果有的话，它们是否有害，然后将文本转换为语音。
Action: edenai_object_detection
Action Input: https://static.javatpoint.com/images/objects.jpg[0m
Observation: [33;1m[1;3m苹果 - 置信度 0.94003654
苹果 - 置信度 0.94003654
苹果 - 置信度 0.94003654
背包 - 置信度 0.7481894
背包 - 置信度 0.7481894
背包 - 置信度 0.7481894
行李和包 - 置信度 0.70691586
行李和包 - 置信度 0.70691586
行李和包 - 置信度 0.70691586
容器 - 置信度 0.654727
容器 - 置信度 0.654727
容器 - 置信度 0.654727
行李和包 - 置信度 0.5871518
行李和包 - 置信度 0.5871518
行李和包 - 置信度 0.5871518[0m
Thought:[32;1m[1;3m 我需要检查这些物体中是否有有害的。
Action: edenai_explicit_content_detection_text
Action Input: 苹果, 背包, 行李和包, 容器[0m
Observation: [36;1m[1;3mnsfw_likelihood: 2
"性暗示": 1
"性暗示": 2
"冒犯": 1
nsfw_likelihood: 1
"性": 1
"仇恨": 1
"骚扰": 1
"自残": 1
"性/未成年人": 1
"仇恨/威胁": 1
"暴力/图形": 1
"自残/意图": 1
"自残/指示": 1
"骚扰/威胁": 1
"暴力": 1[0m
Thought:[32;1m[1;3m 这些物体都没有害。
Action: edenai_text_to_speech
Action Input: '这个物品是安全的'[0m
Observation: [38;5;200m[1;3mhttps://d14uq1pz7dzsdq.cloudfront.net/0546db8b-528e-4b63-9a69-d14d43ad1566_.mp3?Expires=1693316753&Signature=N0KZeK9I-1s7wTgiQOAwH7LFlltwyonSJcDnkdnr8JIJmbgSw6fo6RTxWl~VvD2Hg6igJqxtJFFWyrBmmx-f9wWLw3bZSnuMxkhTRqLX9aUA9N-vPJGiRZV5BFredaOm8pwfo8TcXhVjw08iSxv8GSuyZEIwZkiq4PzdiyVTnKKji6eytV0CrnHrTs~eXZkSnOdD2Fu0ECaKvFHlsF4IDLI8efRvituSk0X3ygdec4HQojl5vmBXJzi1TuhKWOX8UxeQle8pdjjqUPSJ9thTHpucdPy6UbhZOH0C9rbtLrCfvK5rzrT4D~gKy9woICzG34tKRxNxHYVVUPqx2BiInA__&Key-Pair-Id=K1F55BTI9AHGIK[0m
Thought:[32;1m[1;3m 我现在知道最终答案了。
Final Answer: 图像中包含物体，如苹果、背包、行李和包、容器。它们都没有害。文本 '这个物品是安全的' 可以在音频文件中找到，链接为 https://d14uq1pz7dzsdq.cloudfront.net/0546db8b-528e-4b63-9a69-d14d43ad1566_.mp3?Expires=1693316753&Signature=N0KZeK9I-1s7wTgiQOAwH7LFlltwyonSJcDnkdnr8JIJmbgSw6fo6RTxWl~VvD2Hg6igJqxtJFFWyrBmmx-f9wWLw3bZSnuMxkhTRqLX9aUA9N-vPJGiRZV5BFredaOm8pwfo8TcXhVjw08iSxv8GSuyZEIwZkiq4PzdiyVTnKKji6eyt[0m

[1m> 完成链。[0m
```

```python
result["output"]
```



```output
"图像中包含物体，如苹果、背包、行李和包、容器。它们都没有害。文本 '这个物品是安全的' 可以在音频文件中找到，链接为 https://d14uq1pz7dzsdq.cloudfront.net/0546db8b-528e-4b63-9a69-d14d43ad1566_.mp3?Expires=1693316753&Signature=N0KZeK9I-1s7wTgiQOAwH7LFlltwyonSJcDnkdnr8JIJmbgSw6fo6RTxWl~VvD2Hg6igJqxtJFFWyrBmmx-f9wWLw3bZSnuMxkhTRqLX9aUA9N-vPJGiRZV5BFredaOm8pwfo8TcXhVjw08iSxv8GSuyZEIwZkiq4PzdiyVTnKKji6eyt"
```


您可以通过打印结果获取更多执行细节


```python
result
```



```output
{'input': ' 我有这个图像的 URL : "https://static.javatpoint.com/images/objects.jpg"\n                   第一 : 我想知道图像中是否包含物体。\n                   第二 : 如果包含物体，我想知道其中是否有有害的，\n                   第三 : 如果没有有害的物体，将这段文字转为语音 : \'这个物品是安全的\'。\n                   如果观察中有 URL，您将始终将其放入输出（最终答案）中。\n                   ',
 'output': "图像中包含物体，如苹果、背包、行李和包、容器。它们都没有害。文本 '这个物品是安全的' 可以在音频文件中找到，链接为 https://d14uq1pz7dzsdq.cloudfront.net/0546db8b-528e-4b63-9a69-d14d43ad1566_.mp3?Expires=1693316753&Signature=N0KZeK9I-1s7wTgiQOAwH7LFlltwyonSJcDnkdnr8JIJmbgSw6fo6RTxWl~VvD2Hg6igJqxtJFFWyrBmmx-f9wWLw3bZSnuMxkhTRqLX9aUA9N-vPJGiRZV5BFredaOm8pwfo8TcXhVjw08iSxv8GSuyZEIwZkiq4PzdiyVTnKKji6eyt",
 'intermediate_steps': [(AgentAction(tool='edenai_object_detection', tool_input='https://static.javatpoint.com/images/objects.jpg', log=' 我需要确定图像中是否包含物体，如果有的话，它们是否有害，然后将文本转换为语音。\nAction: edenai_object_detection\nAction Input: https://static.javatpoint.com/images/objects.jpg'),
   '苹果 - 置信度 0.94003654\n苹果 - 置信度 0.94003654\n苹果 - 置信度 0.94003654\n背包 - 置信度 0.7481894\n背包 - 置信度 0.7481894\n背包 - 置信度 0.7481894\n行李和包 - 置信度 0.70691586\n行李和包 - 置信度 0.70691586\n行李和包 - 置信度 0.70691586\n容器 - 置信度 0.654727\n容器 - 置信度 0.654727\n容器 - 置信度 0.654727\n行李和包 - 置信度 0.5871518\n行李和包 - 置信度 0.5871518\n行李和包 - 置信度 0.5871518'),
  (AgentAction(tool='edenai_explicit_content_detection_text', tool_input='苹果, 背包, 行李和包, 容器', log=' 我需要检查这些物体中是否有有害的。\nAction: edenai_explicit_content_detection_text\nAction Input: 苹果, 背包, 行李和包, 容器'),
   'nsfw_likelihood: 2\n"性暗示": 1\n"性暗示": 2\n"冒犯": 1\nnsfw_likelihood: 1\n"性": 1\n"仇恨": 1\n"骚扰": 1\n"自残": 1\n"性/未成年人": 1\n"仇恨/威胁": 1\n"暴力/图形": 1\n"自残/意图": 1\n"自残/指示": 1\n"骚扰/威胁": 1\n"暴力": 1'),
  (AgentAction(tool='edenai_text_to_speech', tool_input="'这个物品是安全的'", log=" 这些物体都没有害。\nAction: edenai_text_to_speech\nAction Input: '这个物品是安全的'"),
   'https://d14uq1pz7dzsdq.cloudfront.net/0546db8b-528e-4b63-9a69-d14d43ad1566_.mp3?Expires=1693316753&Signature=N0KZeK9I-1s7wTgiQOAwH7LFlltwyonSJcDnkdnr8JIJmbgSw6fo6RTxWl~VvD2Hg6igJqxtJFFWyrBmmx-f9wWLw3bZSnuMxkhTRqLX9aUA9N-vPJGiRZV5BFredaOm8pwfo8TcXhVjw08iSxv8GSuyZEIwZkiq4PzdiyVTnKKji6eytV0CrnHrTs~eXZkSnOdD2Fu0ECaKvFHlsF4IDLI8efRvituSk0X3ygdec4HQojl5vmBXJzi1TuhKWOX8UxeQle8pdjjqUPSJ9thTHpucdPy6UbhZOH0C9rbtLrCfvK5rzrT4D~gKy9woICzG34tKRxNxHYVVUPqx2BiInA__&Key-Pair-Id=K1F55BTI9AHGIK')]}
```

## 示例与OCR图像


```python
input_ = """我有一个ID的URL: "https://www.citizencard.com/images/citizencard-uk-id-card-2023.jpg"
我想提取其中的信息。
创建一段欢迎该人的文本，并将其转换为语音。
如果观察中有URL，您将始终将其放入输出（最终答案）中。
"""
result = agent_chain(input_)
```
```output


[1m> 进入新的AgentExecutor链...[0m
[32;1m[1;3m 我需要从ID中提取信息，然后将其转换为文本，再转换为语音
Action: edenai_identity_parsing
Action Input: "https://www.citizencard.com/images/citizencard-uk-id-card-2023.jpg"[0m
Observation: [38;5;200m[1;3m姓氏 : 
  值 : ANGELA
名字 : 
  值 : GREENE
出生地 : 
出生日期 : 
  值 : 2000-11-09
签发日期 : 
到期日期 : 
文件ID : 
签发州 : 
地址 : 
年龄 : 
国家 : 
文件类型 : 
  值 : 驾驶证正面
性别 : 
图像ID : 
图像签名 : 
mrz : 
国籍 : [0m
Thought:[32;1m[1;3m 我现在需要将信息转换为文本，然后转换为语音
Action: edenai_text_to_speech
Action Input: "欢迎Angela Greene!"[0m
Observation: [38;5;200m[1;3mhttps://d14uq1pz7dzsdq.cloudfront.net/0c494819-0bbc-4433-bfa4-6e99bd9747ea_.mp3?Expires=1693316851&Signature=YcMoVQgPuIMEOuSpFuvhkFM8JoBMSoGMcZb7MVWdqw7JEf5~67q9dEI90o5todE5mYXB5zSYoib6rGrmfBl4Rn5~yqDwZ~Tmc24K75zpQZIEyt5~ZSnHuXy4IFWGmlIVuGYVGMGKxTGNeCRNUXDhT6TXGZlr4mwa79Ei1YT7KcNyc1dsTrYB96LphnsqOERx4X9J9XriSwxn70X8oUPFfQmLcitr-syDhiwd9Wdpg6J5yHAJjf657u7Z1lFTBMoXGBuw1VYmyno-3TAiPeUcVlQXPueJ-ymZXmwaITmGOfH7HipZngZBziofRAFdhMYbIjYhegu5jS7TxHwRuox32A__&Key-Pair-Id=K1F55BTI9AHGIK[0m
Thought:[32;1m[1;3m 我现在知道最终答案
Final Answer: https://d14uq1pz7dzsdq.cloudfront.net/0c494819-0bbc-4433-bfa4-6e99bd9747ea_.mp3?Expires=1693316851&Signature=YcMoVQgPuIMEOuSpFuvhkFM8JoBMSoGMcZb7MVWdqw7JEf5~67q9dEI90o5todE5mYXB5zSYoib6rGrmfBl4Rn5~yqDwZ~Tmc24K75zpQZIEyt5~ZSnHuXy4IFWGmlIVuGYVGMGKxTGNeCRNUXDhT6TXGZlr4mwa79Ei1YT7KcNyc1dsTrYB96LphnsqOERx4X9J9XriSwxn70X8oUPFfQmLcitr-syDhiwd9Wdpg6J5y[0m

[1m> 完成链。[0m
```

```python
result["output"]
```



```output
'https://d14uq1pz7dzsdq.cloudfront.net/0c494819-0bbc-4433-bfa4-6e99bd9747ea_.mp3?Expires=1693316851&Signature=YcMoVQgPuIMEOuSpFuvhkFM8JoBMSoGMcZb7MVWdqw7JEf5~67q9dEI90o5todE5mYXB5zSYoib6rGrmfBl4Rn5~yqDwZ~Tmc24K75zpQZIEyt5~ZSnHuXy4IFWGmlIVuGYVGMGKxTGNeCRNUXDhT6TXGZlr4mwa79Ei1YT7KcNyc1dsTrYB96LphnsqOERx4X9J9XriSwxn70X8oUPFfQmLcitr-syDhiwd9Wdpg6J5y'
```



```python
input_ = """我有一个发票文件的URL: "https://app.edenai.run/assets/img/data_1.72e3bdcc.png"
我想提取其中的信息。
并回答这些问题：
谁是客户？
公司名称是什么？
"""
result = agent_chain()
```
```output


[1m> 进入新的AgentExecutor链...[0m
[32;1m[1;3m 我需要从发票文件中提取信息
Action: edenai_invoice_parsing
Action Input: "https://app.edenai.run/assets/img/data_1.72e3bdcc.png"[0m
Observation: [36;1m[1;3m客户信息 : 
  客户名称 : Damita J Goldsmith
  客户地址 : 201 Stan Fey Dr,Upper Marlboro, MD 20774
  客户送货地址 : 201 Stan Fey Drive,Upper Marlboro
商家信息 : 
  商家名称 : SNG Engineering Inc
  商家地址 : 344 Main St #200 Gaithersburg, MD 20878 USA
  商家电话 : +1 301 548 0055
发票号码 : 014-03
税费 : 
付款条款 : 收到服务后付款
日期 : 2003-01-20
采购订单号 : 
地点 : 
银行信息 : 
项目行 : 
  描述 : 2003年1月19日的建筑现场检查，房屋、车库车道的缺陷，以及对律师的法律支持。[0m
Thought:[32;1m[1;3m 我现在知道问题的答案
Final Answer: 客户是Damita J Goldsmith，公司名称是SNG Engineering Inc.[0m

[1m> 完成链。[0m
```

```python
result["output"]
```



```output
'客户是Damita J Goldsmith，公司名称是SNG Engineering Inc.'
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)
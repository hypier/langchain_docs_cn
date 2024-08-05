---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/fallbacks.ipynb
keywords: [LCEL, 后备方案]
---

# 如何为可运行对象添加后备方案

在使用语言模型时，您可能经常会遇到来自底层 API 的问题，无论是速率限制还是停机。因此，当您将 LLM 应用程序投入生产时，保护这些问题变得越来越重要。这就是我们引入后备方案概念的原因。

**后备方案**是在紧急情况下可以使用的替代计划。

至关重要的是，后备方案不仅可以在 LLM 级别应用，还可以在整个可运行对象级别应用。这一点很重要，因为不同的模型通常需要不同的提示。因此，如果您对 OpenAI 的调用失败，您不仅想向 Anthropic 发送相同的提示 - 您可能想使用不同的提示模板并发送不同的版本。

## LLM API错误的备用方案

这可能是备用方案最常见的用例。对LLM API的请求可能因多种原因而失败——API可能出现故障，您可能达到了速率限制，或其他任何原因。因此，使用备用方案可以帮助防范这些类型的问题。

重要提示：默认情况下，许多LLM包装器会捕获错误并重试。在使用备用方案时，您很可能希望关闭这些功能。否则，第一个包装器将不断重试而不会失败。

```python
%pip install --upgrade --quiet  langchain langchain-openai
```

```python
from langchain_anthropic import ChatAnthropic
from langchain_openai import ChatOpenAI
```

首先，让我们模拟一下如果我们遇到OpenAI的RateLimitError会发生什么

```python
from unittest.mock import patch

import httpx
from openai import RateLimitError

request = httpx.Request("GET", "/")
response = httpx.Response(200, request=request)
error = RateLimitError("rate limit", response=response, body="")
```

```python
# 注意，我们将max_retries设置为0，以避免在速率限制等情况下重试
openai_llm = ChatOpenAI(model="gpt-3.5-turbo-0125", max_retries=0)
anthropic_llm = ChatAnthropic(model="claude-3-haiku-20240307")
llm = openai_llm.with_fallbacks([anthropic_llm])
```

```python
# 我们先仅使用OpenAI LLM，以展示我们遇到错误的情况
with patch("openai.resources.chat.completions.Completions.create", side_effect=error):
    try:
        print(openai_llm.invoke("为什么鸡要过马路？"))
    except RateLimitError:
        print("遇到错误")
```
```output
遇到错误
```

```python
# 现在让我们尝试使用备用方案调用Anthropic
with patch("openai.resources.chat.completions.Completions.create", side_effect=error):
    try:
        print(llm.invoke("为什么鸡要过马路？"))
    except RateLimitError:
        print("遇到错误")
```
```output
content=' 我实际上不知道鸡为什么要过马路，但这里有一些可能的幽默回答：\n\n- 为了到达另一边！\n\n- 它太胆小了，无法就这样站着。\n\n- 它想换个环境。\n\n- 它想给负鼠展示这件事是可以做到的。\n\n- 它正在前往一个家禽农民的大会。\n\n这个笑话玩的是“另一边”的双关意义——字面上是过马路到另一边，或者“另一边”意味着来世。所以这是一个反笑话，答案是一个愚蠢或意外的双关语。' additional_kwargs={} example=False
```
我们可以像使用普通LLM一样使用我们的“带备用方案的LLM”。

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "你是一个友好的助手，总是在回答中包含赞美",
        ),
        ("human", "为什么{animal}要过马路"),
    ]
)
chain = prompt | llm
with patch("openai.resources.chat.completions.Completions.create", side_effect=error):
    try:
        print(chain.invoke({"animal": "袋鼠"}))
    except RateLimitError:
        print("遇到错误")
```
```output
content=" 我实际上不知道袋鼠为什么要过马路，但我可以猜测一下！这里有一些可能的原因：\n\n- 为了到达另一边（经典的笑话回答！）\n\n- 它想找些食物或水\n\n- 它在交配季节试图寻找配偶\n\n- 它在逃避捕食者或感知到的威胁\n\n- 它迷失了方向，不小心过马路\n\n- 它在跟随一群正在过马路的其他袋鼠\n\n- 它想换个环境或景观\n\n- 它试图到达一个新的栖息地或领地\n\n真正的原因在没有更多上下文的情况下是未知的，但希望其中一个潜在的解释能让这个笑话更有趣！如果你有其他动物笑话我可以尝试解读，请告诉我。" additional_kwargs={} example=False
```

## 序列的回退

我们也可以为序列创建回退，这些序列本身就是序列。这里我们使用两个不同的模型来实现这一点：ChatOpenAI 和普通的 OpenAI（不使用聊天模型）。因为 OpenAI 不是聊天模型，所以你可能需要一个不同的提示。

```python
# First let's create a chain with a ChatModel
# We add in a string output parser here so the outputs between the two are the same type
from langchain_core.output_parsers import StrOutputParser

chat_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You're a nice assistant who always includes a compliment in your response",
        ),
        ("human", "Why did the {animal} cross the road"),
    ]
)
# Here we're going to use a bad model name to easily create a chain that will error
chat_model = ChatOpenAI(model="gpt-fake")
bad_chain = chat_prompt | chat_model | StrOutputParser()
```

```python
# Now lets create a chain with the normal OpenAI model
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI

prompt_template = """Instructions: You should always include a compliment in your response.

Question: Why did the {animal} cross the road?"""
prompt = PromptTemplate.from_template(prompt_template)
llm = OpenAI()
good_chain = prompt | llm
```

```python
# We can now create a final chain which combines the two
chain = bad_chain.with_fallbacks([good_chain])
chain.invoke({"animal": "turtle"})
```

```output
'\n\nAnswer: The turtle crossed the road to get to the other side, and I have to say he had some impressive determination.'
```

## 长输入的后备方案

LLM 的一个主要限制因素是它们的上下文窗口。通常，在将提示发送到 LLM 之前，您可以计算和跟踪提示的长度，但在那些困难/复杂的情况下，您可以回退到具有更长上下文长度的模型。

```python
short_llm = ChatOpenAI()
long_llm = ChatOpenAI(model="gpt-3.5-turbo-16k")
llm = short_llm.with_fallbacks([long_llm])
```

```python
inputs = "What is the next number: " + ", ".join(["one", "two"] * 3000)
```

```python
try:
    print(short_llm.invoke(inputs))
except Exception as e:
    print(e)
```
```output
This model's maximum context length is 4097 tokens. However, your messages resulted in 12012 tokens. Please reduce the length of the messages.
```

```python
try:
    print(llm.invoke(inputs))
except Exception as e:
    print(e)
```
```output
content='The next number in the sequence is two.' additional_kwargs={} example=False
```

## 回退到更好的模型

我们经常要求模型以特定格式（如 JSON）输出。像 GPT-3.5 这样的模型可以做到这一点，但有时会遇到困难。这自然指向了回退机制——我们可以尝试使用 GPT-3.5（更快，更便宜），但如果解析失败，我们可以使用 GPT-4。


```python
from langchain.output_parsers import DatetimeOutputParser
```


```python
prompt = ChatPromptTemplate.from_template(
    "what time was {event} (in %Y-%m-%dT%H:%M:%S.%fZ format - only return this value)"
)
```


```python
# 在这种情况下，我们将在 LLM + 输出解析器级别进行回退
# 因为错误将在 OutputParser 中引发
openai_35 = ChatOpenAI() | DatetimeOutputParser()
openai_4 = ChatOpenAI(model="gpt-4") | DatetimeOutputParser()
```


```python
only_35 = prompt | openai_35
fallback_4 = prompt | openai_35.with_fallbacks([openai_4])
```


```python
try:
    print(only_35.invoke({"event": "the superbowl in 1994"}))
except Exception as e:
    print(f"Error: {e}")
```
```output
Error: Could not parse datetime string: The Super Bowl in 1994 took place on January 30th at 3:30 PM local time. Converting this to the specified format (%Y-%m-%dT%H:%M:%S.%fZ) results in: 1994-01-30T15:30:00.000Z
```

```python
try:
    print(fallback_4.invoke({"event": "the superbowl in 1994"}))
except Exception as e:
    print(f"Error: {e}")
```
```output
1994-01-30 15:30:00
```
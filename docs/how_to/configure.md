---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/configure.ipynb
sidebar_position: 7
keywords: [ConfigurableField, configurable_fields, ConfigurableAlternatives, configurable_alternatives, LCEL]
---

# 如何配置运行时链内部

:::info 前提条件

本指南假设您熟悉以下概念：
- [LangChain 表达式语言 (LCEL)](/docs/concepts/#langchain-expression-language)
- [链接可运行对象](/docs/how_to/sequence/)
- [绑定运行时参数](/docs/how_to/binding/)

:::

有时您可能希望在链中尝试多种不同的方式，甚至向最终用户展示这些方式。这可以包括调整温度等参数，甚至将一个模型替换为另一个模型。为了使这一体验尽可能简单，我们定义了两种方法。

- `configurable_fields` 方法。此方法允许您配置可运行对象的特定字段。
  - 这与可运行对象上的 [`.bind`](/docs/how_to/binding) 方法相关，但允许您在运行时为链中的特定步骤指定参数，而不是事先指定。
- `configurable_alternatives` 方法。通过此方法，您可以列出在运行时可以设置的任何特定可运行对象的替代方案，并将其替换为那些指定的替代方案。

## 可配置字段

让我们通过一个示例来演示如何在运行时配置聊天模型字段，例如温度：


```python
%pip install --upgrade --quiet langchain langchain-openai

import os
from getpass import getpass

os.environ["OPENAI_API_KEY"] = getpass()
```


```python
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import ConfigurableField
from langchain_openai import ChatOpenAI

model = ChatOpenAI(temperature=0).configurable_fields(
    temperature=ConfigurableField(
        id="llm_temperature",
        name="LLM 温度",
        description="LLM 的温度",
    )
)

model.invoke("pick a random number")
```



```output
AIMessage(content='17', response_metadata={'token_usage': {'completion_tokens': 1, 'prompt_tokens': 11, 'total_tokens': 12}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-ba26a0da-0a69-4533-ab7f-21178a73d303-0')
```


在上面，我们将 `temperature` 定义为一个 [`ConfigurableField`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.utils.ConfigurableField.html#langchain_core.runnables.utils.ConfigurableField)，可以在运行时进行设置。为此，我们使用 [`with_config`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.with_config) 方法，如下所示：


```python
model.with_config(configurable={"llm_temperature": 0.9}).invoke("pick a random number")
```



```output
AIMessage(content='12', response_metadata={'token_usage': {'completion_tokens': 1, 'prompt_tokens': 11, 'total_tokens': 12}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-ba8422ad-be77-4cb1-ac45-ad0aae74e3d9-0')
```


请注意，字典中传递的 `llm_temperature` 条目的键与 `ConfigurableField` 的 `id` 相同。

我们还可以这样做，只影响链中某一步：


```python
prompt = PromptTemplate.from_template("Pick a random number above {x}")
chain = prompt | model

chain.invoke({"x": 0})
```



```output
AIMessage(content='27', response_metadata={'token_usage': {'completion_tokens': 1, 'prompt_tokens': 14, 'total_tokens': 15}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-ecd4cadd-1b72-4f92-b9a0-15e08091f537-0')
```



```python
chain.with_config(configurable={"llm_temperature": 0.9}).invoke({"x": 0})
```



```output
AIMessage(content='35', response_metadata={'token_usage': {'completion_tokens': 1, 'prompt_tokens': 14, 'total_tokens': 15}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-a916602b-3460-46d3-a4a8-7c926ec747c0-0')
```

### 使用 HubRunnables

这对于切换提示非常有用


```python
from langchain.runnables.hub import HubRunnable

prompt = HubRunnable("rlm/rag-prompt").configurable_fields(
    owner_repo_commit=ConfigurableField(
        id="hub_commit",
        name="Hub Commit",
        description="要拉取的 Hub 提交",
    )
)

prompt.invoke({"question": "foo", "context": "bar"})
```



```output
ChatPromptValue(messages=[HumanMessage(content="你是一个用于问答任务的助手。使用以下检索到的上下文片段来回答问题。如果你不知道答案，就说你不知道。最多使用三句话并保持答案简洁。\n问题: foo \n上下文: bar \n答案:")])
```



```python
prompt.with_config(configurable={"hub_commit": "rlm/rag-prompt-llama"}).invoke(
    {"question": "foo", "context": "bar"}
)
```



```output
ChatPromptValue(messages=[HumanMessage(content="[INST]<<SYS>> 你是一个用于问答任务的助手。使用以下检索到的上下文片段来回答问题。如果你不知道答案，就说你不知道。最多使用三句话并保持答案简洁。<</SYS>> \n问题: foo \n上下文: bar \n答案: [/INST]")])
```

## 可配置的替代方案



`configurable_alternatives()` 方法允许我们在链中用替代步骤进行替换。下面，我们将一个聊天模型替换为另一个：


```python
%pip install --upgrade --quiet langchain-anthropic

import os
from getpass import getpass

os.environ["ANTHROPIC_API_KEY"] = getpass()
```
```output
[33mWARNING: You are using pip version 22.0.4; however, version 24.0 is available.
You should consider upgrading via the '/Users/jacoblee/.pyenv/versions/3.10.5/bin/python -m pip install --upgrade pip' command.[0m[33m
[0mNote: you may need to restart the kernel to use updated packages.
```

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import ConfigurableField
from langchain_openai import ChatOpenAI

llm = ChatAnthropic(
    model="claude-3-haiku-20240307", temperature=0
).configurable_alternatives(
    # This gives this field an id
    # When configuring the end runnable, we can then use this id to configure this field
    ConfigurableField(id="llm"),
    # This sets a default_key.
    # If we specify this key, the default LLM (ChatAnthropic initialized above) will be used
    default_key="anthropic",
    # This adds a new option, with name `openai` that is equal to `ChatOpenAI()`
    openai=ChatOpenAI(),
    # This adds a new option, with name `gpt4` that is equal to `ChatOpenAI(model="gpt-4")`
    gpt4=ChatOpenAI(model="gpt-4"),
    # You can add more configuration options here
)
prompt = PromptTemplate.from_template("Tell me a joke about {topic}")
chain = prompt | llm

# By default it will call Anthropic
chain.invoke({"topic": "bears"})
```



```output
AIMessage(content="Here's a bear joke for you:\n\nWhy don't bears wear socks? \nBecause they have bear feet!\n\nHow's that? I tried to come up with a simple, silly pun-based joke about bears. Puns and wordplay are a common way to create humorous bear jokes. Let me know if you'd like to hear another one!", response_metadata={'id': 'msg_018edUHh5fUbWdiimhrC3dZD', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 13, 'output_tokens': 80}}, id='run-775bc58c-28d7-4e6b-a268-48fa6661f02f-0')
```



```python
# We can use `.with_config(configurable={"llm": "openai"})` to specify an llm to use
chain.with_config(configurable={"llm": "openai"}).invoke({"topic": "bears"})
```



```output
AIMessage(content="Why don't bears like fast food?\n\nBecause they can't catch it!", response_metadata={'token_usage': {'completion_tokens': 15, 'prompt_tokens': 13, 'total_tokens': 28}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-7bdaa992-19c9-4f0d-9a0c-1f326bc992d4-0')
```



```python
# If we use the `default_key` then it uses the default
chain.with_config(configurable={"llm": "anthropic"}).invoke({"topic": "bears"})
```



```output
AIMessage(content="Here's a bear joke for you:\n\nWhy don't bears wear socks? \nBecause they have bear feet!\n\nHow's that? I tried to come up with a simple, silly pun-based joke about bears. Puns and wordplay are a common way to create humorous bear jokes. Let me know if you'd like to hear another one!", response_metadata={'id': 'msg_01BZvbmnEPGBtcxRWETCHkct', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 13, 'output_tokens': 80}}, id='run-59b6ee44-a1cd-41b8-a026-28ee67cdd718-0')
```

### 使用提示

我们可以做类似的事情，但在提示之间交替

```python
llm = ChatAnthropic(model="claude-3-haiku-20240307", temperature=0)
prompt = PromptTemplate.from_template(
    "告诉我一个关于 {topic} 的笑话"
).configurable_alternatives(
    # 这给这个字段一个 id
    # 当配置最终可运行项时，我们可以使用这个 id 来配置这个字段
    ConfigurableField(id="prompt"),
    # 这设置了一个 default_key。
    # 如果我们指定这个键，将使用默认的 LLM（上述初始化的 ChatAnthropic）
    default_key="joke",
    # 这添加了一个新选项，名称为 `poem`
    poem=PromptTemplate.from_template("写一首关于 {topic} 的短诗"),
    # 你可以在这里添加更多配置选项
)
chain = prompt | llm

# 默认情况下，它将写一个笑话
chain.invoke({"topic": "bears"})
```

```output
AIMessage(content="这是一个关于熊的笑话：\n\n为什么熊不穿袜子？ \n因为它们有熊脚！", response_metadata={'id': 'msg_01DtM1cssjNFZYgeS3gMZ49H', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 13, 'output_tokens': 28}}, id='run-8199af7d-ea31-443d-b064-483693f2e0a1-0')
```

```python
# 我们可以配置它写一首诗
chain.with_config(configurable={"prompt": "poem"}).invoke({"topic": "bears"})
```

```output
AIMessage(content="这是关于熊的短诗：\n\n雄伟的熊，强壮而真实，\n在森林中游荡，自由而狂野。\n强大的爪子，柔软的棕色毛发，\n威严的尊重，自然的王冠。\n\n觅食浆果，捕鱼溪流，\n保护幼崽，凶猛而敏锐。\n强大的熊，令人瞩目的景象，\n荒野的守护者，未曾诉说。\n\n在野外它们统治至高无上，\n体现自然的宏伟主题。\n熊，力量与优雅的象征，\n吸引着所有看到它们面孔的人。", response_metadata={'id': 'msg_01Wck3qPxrjURtutvtodaJFn', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 13, 'output_tokens': 134}}, id='run-69414a1e-51d7-4bec-a307-b34b7d61025e-0')
```

### 使用提示和LLMs

我们也可以配置多个内容！
这是一个同时使用提示和LLMs的示例。

```python
llm = ChatAnthropic(
    model="claude-3-haiku-20240307", temperature=0
).configurable_alternatives(
    # 这给这个字段一个id
    # 在配置最终可运行的内容时，我们可以使用这个id来配置这个字段
    ConfigurableField(id="llm"),
    # 这设置一个default_key。
    # 如果我们指定这个key，将使用默认的LLM（上述初始化的ChatAnthropic）
    default_key="anthropic",
    # 这添加一个新选项，名称为`openai`，等于`ChatOpenAI()`
    openai=ChatOpenAI(),
    # 这添加一个新选项，名称为`gpt4`，等于`ChatOpenAI(model="gpt-4")`
    gpt4=ChatOpenAI(model="gpt-4"),
    # 你可以在这里添加更多配置选项
)
prompt = PromptTemplate.from_template(
    "告诉我一个关于{topic}的笑话"
).configurable_alternatives(
    # 这给这个字段一个id
    # 在配置最终可运行的内容时，我们可以使用这个id来配置这个字段
    ConfigurableField(id="prompt"),
    # 这设置一个default_key。
    # 如果我们指定这个key，将使用默认的LLM（上述初始化的ChatAnthropic）
    default_key="joke",
    # 这添加一个新选项，名称为`poem`
    poem=PromptTemplate.from_template("写一首关于{topic}的短诗"),
    # 你可以在这里添加更多配置选项
)
chain = prompt | llm

# 我们可以配置它写一首用OpenAI的诗
chain.with_config(configurable={"prompt": "poem", "llm": "openai"}).invoke(
    {"topic": "bears"}
)
```



```output
AIMessage(content="在广阔而深邃的森林中，\n熊以优雅和骄傲漫游。\n毛发如夜色般黑暗，\n它们以全部的力量统治这片土地。\n\n在冬天的寒冷中，它们冬眠，\n春天它们苏醒，饥饿而伟大。\n锋利的爪子和敏锐的眼睛，\n它们寻找食物，凶猛而瘦削。\n\n但在它们坚硬的外表下，\n藏着一颗温暖而高贵的心。\n它们用尽全力爱护幼崽，\n在白天和黑夜中保护它们。\n\n所以让我们欣赏这些雄伟的生物，\n对它们的力量和特征感到敬畏。\n因为在野外，它们是至高无上的，\n强大的熊，永恒的梦想。", response_metadata={'token_usage': {'completion_tokens': 133, 'prompt_tokens': 13, 'total_tokens': 146}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-5eec0b96-d580-49fd-ac4e-e32a0803b49b-0')
```



```python
# 如果我们只想配置一个，也可以
chain.with_config(configurable={"llm": "openai"}).invoke({"topic": "bears"})
```



```output
AIMessage(content="为什么熊不穿鞋子？\n\n因为它们有熊掌！", response_metadata={'token_usage': {'completion_tokens': 13, 'prompt_tokens': 13, 'total_tokens': 26}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-c1b14c9c-4988-49b8-9363-15bfd479973a-0')
```

### 保存配置

我们还可以轻松地将配置好的链保存为它们自己的对象


```python
openai_joke = chain.with_config(configurable={"llm": "openai"})

openai_joke.invoke({"topic": "bears"})
```



```output
AIMessage(content="Why did the bear break up with his girlfriend? \nBecause he couldn't bear the relationship anymore!", response_metadata={'token_usage': {'completion_tokens': 20, 'prompt_tokens': 13, 'total_tokens': 33}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-391ebd55-9137-458b-9a11-97acaff6a892-0')
```

## 下一步

您现在知道如何在运行时配置链的内部步骤。

要了解更多信息，请参阅本节中关于可运行对象的其他操作指南，包括：

- 使用 [.bind()](/docs/how_to/binding) 作为设置可运行对象运行时参数的更简单方法
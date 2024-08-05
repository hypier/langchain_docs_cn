---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/arthur_tracking.ipynb
---

# Arthur

>[Arthur](https://arthur.ai) 是一个模型监控和可观测性平台。

以下指南展示了如何使用 Arthur 回调处理程序运行注册的聊天 LLM，以自动记录模型推理到 Arthur。

如果您当前没有将模型接入 Arthur，请访问我们的 [生成文本模型接入指南](https://docs.arthur.ai/user-guide/walkthroughs/model-onboarding/generative_text_onboarding.html)。有关如何使用 `Arthur SDK` 的更多信息，请访问我们的 [文档](https://docs.arthur.ai/)。

## 安装与设置

在此处放置Arthur凭证


```python
arthur_url = "https://app.arthur.ai"
arthur_login = "your-arthur-login-username-here"
arthur_model_id = "your-arthur-model-id-here"
```

## 回调处理程序


```python
from langchain_community.callbacks import ArthurCallbackHandler
from langchain_core.callbacks import StreamingStdOutCallbackHandler
from langchain_core.messages import HumanMessage
from langchain_openai import ChatOpenAI
```

使用 Arthur 回调处理程序创建 Langchain LLM


```python
def make_langchain_chat_llm():
    return ChatOpenAI(
        streaming=True,
        temperature=0.1,
        callbacks=[
            StreamingStdOutCallbackHandler(),
            ArthurCallbackHandler.from_credentials(
                arthur_model_id, arthur_url=arthur_url, arthur_login=arthur_login
            ),
        ],
    )
```


```python
chatgpt = make_langchain_chat_llm()
```
```output
请输入管理员密码: ········
```
使用此 `run` 函数运行聊天 LLM 将会将聊天历史保存在一个持续的列表中，以便对话可以引用早期消息，并将每个响应记录到 Arthur 平台。您可以在您的 [模型仪表板页面](https://app.arthur.ai/) 查看此模型的推断历史。

输入 `q` 退出运行循环


```python
def run(llm):
    history = []
    while True:
        user_input = input("\n>>> 输入 >>>\n>>>: ")
        if user_input == "q":
            break
        history.append(HumanMessage(content=user_input))
        history.append(llm(history))
```


```python
run(chatgpt)
```
```output

>>> 输入 >>>
>>>: 什么是回调处理程序？
回调处理程序，也称为回调函数或回调方法，是一段在特定事件或条件发生时执行的代码。它通常用于支持事件驱动或异步编程范式的编程语言中。

回调处理程序的目的是为开发人员提供一种定义自定义行为的方式，这些行为应在特定事件发生时执行。程序不会等待结果或阻塞执行，而是注册一个回调函数并继续执行其他任务。当事件被触发时，回调函数被调用，从而使程序能够做出相应的反应。

回调处理程序通常用于各种场景，例如处理用户输入、响应网络请求、处理异步操作和实现事件驱动架构。它们提供了一种灵活和模块化的方式来处理事件，并解耦系统的不同组件。
>>> 输入 >>>
>>>: 我需要做什么才能充分利用这个？
要充分利用回调处理程序，您应考虑以下几点：

1. 理解事件或条件：确定您希望通过回调处理程序响应的特定事件或条件。这可以是用户输入、网络请求或任何其他异步操作。

2. 定义回调函数：创建一个将在事件或条件发生时执行的函数。该函数应包含您希望针对事件采取的所需行为或操作。

3. 注册回调函数：根据您使用的编程语言或框架，您可能需要将回调函数注册或附加到适当的事件或条件。这确保回调函数在事件发生时被调用。

4. 处理回调：在回调函数中实现必要的逻辑来处理事件或条件。这可能涉及更新用户界面、处理数据、进行进一步请求或触发其他操作。

5. 考虑错误处理：重要的是处理回调函数内可能发生的任何潜在错误或异常。这确保您的程序能够优雅地处理意外情况，并防止崩溃或不良行为。

6. 保持代码可读性和模块化：随着代码库的增长，保持回调处理程序的组织性和可维护性至关重要。考虑使用设计模式或架构原则以模块化和可扩展的方式构建代码。

通过遵循这些步骤，您可以利用回调处理程序的好处，例如异步和事件驱动编程、提高响应能力和模块化代码设计。
>>> 输入 >>>
>>>: q
```
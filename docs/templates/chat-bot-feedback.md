# 聊天机器人反馈模板

此模板展示了如何在没有明确用户反馈的情况下评估您的聊天机器人。它在 [chain.py](https://github.com/langchain-ai/langchain/blob/master/templates/chat-bot-feedback/chat_bot_feedback/chain.py) 中定义了一个简单的聊天机器人和一个自定义评估器，该评估器根据后续用户响应来评分机器人的响应有效性。您可以通过在聊天机器人上调用 `with_config` 来应用此运行评估器，以便在服务之前进行配置。您还可以使用此模板直接部署您的聊天应用。

[聊天机器人](https://python.langchain.com/docs/use_cases/chatbots) 是部署 LLM 的最常见接口之一。聊天机器人的质量各异，使得持续开发变得重要。但用户往往不愿通过点赞或点踩按钮等机制留下明确的反馈。此外，传统分析如“会话时长”或“对话时长”往往缺乏清晰度。然而，与聊天机器人的多轮对话可以提供大量信息，我们可以将其转化为用于微调、评估和产品分析的指标。

以 [Chat Langchain](https://chat.langchain.com/) 为案例研究，所有查询中仅约 0.04% 收到明确反馈。然而，约 70% 的查询是对先前问题的后续提问。这些后续查询中的相当一部分继续提供我们可以用来推断先前 AI 响应质量的有用信息。

此模板有助于解决“反馈稀缺”问题。以下是此聊天机器人的一个示例调用：

[](https://smith.langchain.com/public/3378daea-133c-4fe8-b4da-0a3044c5dbe8/r?runtab=1)

当用户对此作出响应时 ([link](https://smith.langchain.com/public/a7e2df54-4194-455d-9978-cecd8be0df1e/r))，响应评估器被调用，产生以下评估运行：

[](https://smith.langchain.com/public/534184ee-db8f-4831-a386-3f578145114c/r)

如上所示，评估器看到用户越来越沮丧，表明先前的响应并不有效。

## LangSmith 反馈

[LangSmith](https://smith.langchain.com/) 是一个用于构建生产级 LLM 应用的平台。除了调试和离线评估功能，LangSmith 还帮助您捕获用户和模型辅助的反馈，以优化您的 LLM 应用。此模板使用 LLM 为您的应用生成反馈，您可以利用这些反馈不断改进您的服务。有关使用 LangSmith 收集反馈的更多示例，请参阅 [文档](https://docs.smith.langchain.com/cookbook/feedback-examples)。

## 评估器实现

用户反馈是通过自定义 `RunEvaluator` 推断出来的。这个评估器是通过 `EvaluatorCallbackHandler` 调用的，它在一个单独的线程中运行，以避免干扰聊天机器人的运行时。您可以通过在您的 LangChain 对象上调用以下函数，在任何兼容的聊天机器人上使用这个自定义评估器：

```python
my_chain.with_config(
    callbacks=[
        EvaluatorCallbackHandler(
            evaluators=[
                ResponseEffectivenessEvaluator(evaluate_response_effectiveness)
            ]
        )
    ],
)
```

评估器指示 LLM，特别是 `gpt-3.5-turbo`，根据用户的后续响应评估 AI 最近的聊天消息。它生成一个分数和相应的推理，这些内容被转换为 LangSmith 中的反馈，并应用于作为 `last_run_id` 提供的值。

在 LLM 中使用的提示 [可以在中心找到](https://smith.langchain.com/hub/wfh/response-effectiveness)。您可以根据需要自定义它，例如添加应用上下文（如应用的目标或它应该响应的问题类型）或您希望 LLM 关注的“症状”。这个评估器还利用了 OpenAI 的函数调用 API，以确保评分的输出更加一致和结构化。

## 环境变量

确保 `OPENAI_API_KEY` 已设置以使用 OpenAI 模型。同时，通过设置 `LANGSMITH_API_KEY` 来配置 LangSmith。

```bash
export OPENAI_API_KEY=sk-...
export LANGSMITH_API_KEY=...
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_PROJECT=my-project # Set to the project you want to save to
```

## 使用

如果通过 `LangServe` 部署，我们建议配置服务器以返回回调事件。这将确保后端追踪包含您使用 `RemoteRunnable` 生成的任何追踪。

```python
from chat_bot_feedback.chain import chain

add_routes(app, chain, path="/chat-bot-feedback", include_callback_events=True)
```

在服务器运行时，您可以使用以下代码片段来流式传输 2 次对话的聊天机器人响应。

```python
from functools import partial
from typing import Dict, Optional, Callable, List
from langserve import RemoteRunnable
from langchain.callbacks.manager import tracing_v2_enabled
from langchain_core.messages import BaseMessage, AIMessage, HumanMessage

# 用您 LangServe 服务器提供的 URL 更新
chain = RemoteRunnable("http://127.0.0.1:8031/chat-bot-feedback")

def stream_content(
    text: str,
    chat_history: Optional[List[BaseMessage]] = None,
    last_run_id: Optional[str] = None,
    on_chunk: Callable = None,
):
    results = []
    with tracing_v2_enabled() as cb:
        for chunk in chain.stream(
            {"text": text, "chat_history": chat_history, "last_run_id": last_run_id},
        ):
            on_chunk(chunk)
            results.append(chunk)
        last_run_id = cb.latest_run.id if cb.latest_run else None
    return last_run_id, "".join(results)

chat_history = []
text = "我的钥匙在哪里？"
last_run_id, response_message = stream_content(text, on_chunk=partial(print, end=""))
print()
chat_history.extend([HumanMessage(content=text), AIMessage(content=response_message)])
text = "我到处都找不到它们"  # 之前的响应可能会收到低分，
# 因为用户的挫败感似乎在加剧。
last_run_id, response_message = stream_content(
    text,
    chat_history=chat_history,
    last_run_id=str(last_run_id),
    on_chunk=partial(print, end=""),
)
print()
chat_history.extend([HumanMessage(content=text), AIMessage(content=response_message)])
```

这使用 `tracing_v2_enabled` 回调管理器获取调用的运行 ID，我们在同一聊天线程中的后续调用中提供该 ID，以便评估者可以将反馈分配给适当的追踪。

## 结论

此模板提供了一个简单的聊天机器人定义，您可以直接使用 LangServe 部署。它定义了一个自定义评估器，用于记录聊天机器人的评估反馈，而无需任何显式的用户评分。这是一种有效的方法，可以增强您的分析，并更好地选择用于微调和评估的数据点。
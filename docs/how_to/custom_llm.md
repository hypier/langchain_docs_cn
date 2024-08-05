---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/custom_llm.ipynb
---

# 如何创建自定义 LLM 类

本笔记本介绍了如何创建自定义 LLM 包装器，以便您可以使用自己的 LLM 或与 LangChain 支持的包装器不同的包装器。

使用标准 `LLM` 接口包装您的 LLM，可以让您在现有的 LangChain 程序中以最小的代码修改使用您的 LLM！

作为额外的好处，您的 LLM 将自动成为 LangChain `Runnable`，并将享受开箱即用的一些优化、异步支持、`astream_events` API 等。

## 实现

自定义 LLM 需要实现的两个必需内容：

| 方法          | 描述                                                                  |
|---------------|-----------------------------------------------------------------------|
| `_call`       | 接受一个字符串和一些可选的停止词，并返回一个字符串。由 `invoke` 使用。 |
| `_llm_type`   | 返回一个字符串的属性，仅用于日志记录目的。                           |

可选实现：

| 方法                  | 描述                                                                                                  |
|----------------------|-------------------------------------------------------------------------------------------------------|
| `_identifying_params` | 用于帮助识别模型并打印 LLM；应返回一个字典。这是一个 **@property**。                                 |
| `_acall`              | 提供 `_call` 的异步本地实现，由 `ainvoke` 使用。                                                     |
| `_stream`             | 方法逐个输出流式生成的令牌。                                                                          |
| `_astream`            | 提供 `_stream` 的异步本地实现；在较新的 LangChain 版本中，默认为 `_stream`。                        |

让我们实现一个简单的自定义 LLM，仅返回输入的前 n 个字符。

```python
from typing import Any, Dict, Iterator, List, Mapping, Optional

from langchain_core.callbacks.manager import CallbackManagerForLLMRun
from langchain_core.language_models.llms import LLM
from langchain_core.outputs import GenerationChunk


class CustomLLM(LLM):
    """一个自定义聊天模型，回显输入的前 `n` 个字符。

    在向 LangChain 提交实现时，请仔细记录模型，包括初始化参数，
    包含如何初始化模型的示例，并包含任何相关的
    底层模型文档或 API 的链接。

    示例：

        .. code-block:: python

            model = CustomChatModel(n=2)
            result = model.invoke([HumanMessage(content="hello")])
            result = model.batch([[HumanMessage(content="hello")],
                                 [HumanMessage(content="world")]])
    """

    n: int
    """从提示的最后一条消息中回显的字符数。"""

    def _call(
        self,
        prompt: str,
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> str:
        """在给定输入上运行 LLM。

        重写此方法以实现 LLM 逻辑。

        参数：
            prompt: 要生成的提示。
            stop: 生成时使用的停止词。模型输出在任何停止子字符串的第一次出现处被截断。
                如果不支持停止令牌，请考虑引发 NotImplementedError。
            run_manager: 运行的回调管理器。
            **kwargs: 任意其他关键字参数。这些通常传递给模型提供者 API 调用。

        返回：
            模型输出作为字符串。实际的完成不应包含提示。
        """
        if stop is not None:
            raise ValueError("不允许使用 stop kwargs。")
        return prompt[: self.n]

    def _stream(
        self,
        prompt: str,
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> Iterator[GenerationChunk]:
        """在给定提示上流式传输 LLM。

        支持流式传输的子类应重写此方法。

        如果未实现，对流的调用的默认行为将回退到模型的非流式版本，并将
        输出作为一个单独的块返回。

        参数：
            prompt: 要生成的提示。
            stop: 生成时使用的停止词。模型输出在这些子字符串的第一次出现处被截断。
            run_manager: 运行的回调管理器。
            **kwargs: 任意其他关键字参数。这些通常传递给模型提供者 API 调用。

        返回：
            GenerationChunks 的迭代器。
        """
        for char in prompt[: self.n]:
            chunk = GenerationChunk(text=char)
            if run_manager:
                run_manager.on_llm_new_token(chunk.text, chunk=chunk)

            yield chunk

    @property
    def _identifying_params(self) -> Dict[str, Any]:
        """返回一个识别参数的字典。"""
        return {
            # 模型名称允许用户在 LLM 监控应用中指定自定义令牌计数
            # 规则（例如，在 LangSmith 中，用户可以为其模型提供每个令牌定价并监控
            # 给定 LLM 的成本。）
            "model_name": "CustomChatModel",
        }

    @property
    def _llm_type(self) -> str:
        """获取此聊天模型使用的语言模型的类型。仅用于日志记录目的。"""
        return "custom"
```

### 让我们测试一下 🧪

这个 LLM 将实现 LangChain 的标准 `Runnable` 接口，许多 LangChain 抽象都支持它！


```python
llm = CustomLLM(n=5)
print(llm)
```
```output
[1mCustomLLM[0m
Params: {'model_name': 'CustomChatModel'}
```

```python
llm.invoke("This is a foobar thing")
```



```output
'This '
```



```python
await llm.ainvoke("world")
```



```output
'world'
```



```python
llm.batch(["woof woof woof", "meow meow meow"])
```



```output
['woof ', 'meow ']
```



```python
await llm.abatch(["woof woof woof", "meow meow meow"])
```



```output
['woof ', 'meow ']
```



```python
async for token in llm.astream("hello"):
    print(token, end="|", flush=True)
```
```output
h|e|l|l|o|
```
让我们确认它与其他 `LangChain` API 的良好集成。


```python
from langchain_core.prompts import ChatPromptTemplate
```


```python
prompt = ChatPromptTemplate.from_messages(
    [("system", "you are a bot"), ("human", "{input}")]
)
```


```python
llm = CustomLLM(n=7)
chain = prompt | llm
```


```python
idx = 0
async for event in chain.astream_events({"input": "hello there!"}, version="v1"):
    print(event)
    idx += 1
    if idx > 7:
        # Truncate
        break
```
```output
{'event': 'on_chain_start', 'run_id': '05f24b4f-7ea3-4fb6-8417-3aa21633462f', 'name': 'RunnableSequence', 'tags': [], 'metadata': {}, 'data': {'input': {'input': 'hello there!'}}}
{'event': 'on_prompt_start', 'name': 'ChatPromptTemplate', 'run_id': '7e996251-a926-4344-809e-c425a9846d21', 'tags': ['seq:step:1'], 'metadata': {}, 'data': {'input': {'input': 'hello there!'}}}
{'event': 'on_prompt_end', 'name': 'ChatPromptTemplate', 'run_id': '7e996251-a926-4344-809e-c425a9846d21', 'tags': ['seq:step:1'], 'metadata': {}, 'data': {'input': {'input': 'hello there!'}, 'output': ChatPromptValue(messages=[SystemMessage(content='you are a bot'), HumanMessage(content='hello there!')])}}
{'event': 'on_llm_start', 'name': 'CustomLLM', 'run_id': 'a8766beb-10f4-41de-8750-3ea7cf0ca7e2', 'tags': ['seq:step:2'], 'metadata': {}, 'data': {'input': {'prompts': ['System: you are a bot\nHuman: hello there!']}}}
{'event': 'on_llm_stream', 'name': 'CustomLLM', 'run_id': 'a8766beb-10f4-41de-8750-3ea7cf0ca7e2', 'tags': ['seq:step:2'], 'metadata': {}, 'data': {'chunk': 'S'}}
{'event': 'on_chain_stream', 'run_id': '05f24b4f-7ea3-4fb6-8417-3aa21633462f', 'tags': [], 'metadata': {}, 'name': 'RunnableSequence', 'data': {'chunk': 'S'}}
{'event': 'on_llm_stream', 'name': 'CustomLLM', 'run_id': 'a8766beb-10f4-41de-8750-3ea7cf0ca7e2', 'tags': ['seq:step:2'], 'metadata': {}, 'data': {'chunk': 'y'}}
{'event': 'on_chain_stream', 'run_id': '05f24b4f-7ea3-4fb6-8417-3aa21633462f', 'tags': [], 'metadata': {}, 'name': 'RunnableSequence', 'data': {'chunk': 'y'}}
```

## 贡献

我们感谢所有聊天模型集成的贡献。

以下是一个检查清单，以帮助确保您的贡献被添加到 LangChain：

文档：

* 模型包含所有初始化参数的文档字符串，因为这些将在 [APIReference](https://api.python.langchain.com/en/stable/langchain_api_reference.html) 中显示。
* 如果模型由服务提供支持，模型的类文档字符串中应包含指向模型 API 的链接。

测试：

* [ ] 为重写的方法添加单元或集成测试。如果您重写了相应的代码，请验证 `invoke`、`ainvoke`、`batch`、`stream` 是否正常工作。

流式处理（如果您正在实现）：

* [ ] 确保调用 `on_llm_new_token` 回调
* [ ] `on_llm_new_token` 在生成块之前被调用

停止令牌行为：

* [ ] 应尊重停止令牌
* [ ] 停止令牌应作为响应的一部分包含在内

秘密 API 密钥：

* [ ] 如果您的模型连接到 API，它可能会在初始化时接受 API 密钥。使用 Pydantic 的 `SecretStr` 类型来处理秘密，以便在打印模型时不会意外打印出来。
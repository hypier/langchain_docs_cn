---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/mlflow_tracking.ipynb
---

# MLflow

>[MLflow](https://mlflow.org/) 是一个多功能的开源平台，用于管理机器学习生命周期中的工作流和工件。它与许多流行的机器学习库内置集成，但可以与任何库、算法或部署工具一起使用。它旨在具有可扩展性，因此您可以编写插件以支持新的工作流、库和工具。

在 LangChain 集成的上下文中，MLflow 提供以下功能：

- **实验跟踪**：MLflow 跟踪并存储您 LangChain 实验的工件，包括模型、代码、提示、指标等。
- **依赖管理**：MLflow 自动记录模型依赖，确保开发和生产环境之间的一致性。
- **模型评估**：MLflow 提供原生功能来评估 LangChain 应用程序。
- **追踪**：MLflow 允许您直观地追踪数据在您的 LangChain 链、代理、检索器或其他组件中的流动。

**注意**：追踪功能仅在 MLflow 版本 2.14.0 及更高版本中可用。

本笔记本演示如何使用 MLflow 跟踪您的 LangChain 实验。有关此功能的更多信息，以及探索使用 LangChain 与 MLflow 的教程和示例，请参考 [MLflow 文档中的 LangChain 集成](https://mlflow.org/docs/latest/llms/langchain/index.html)。

## 设置

安装 MLflow Python 包：

```python
%pip install google-search-results num
```

```python
%pip install mlflow -qU
```

本示例利用了 OpenAI LLM。如果需要，可以跳过下面的命令，使用其他 LLM。

```python
%pip install langchain-openai -qU
```

```python
import os

# 如果您有运行 MLflow Tracking Server，请设置 MLflow 跟踪 URI
os.environ["MLFLOW_TRACKING_URI"] = ""
os.environ["OPENAI_API_KEY"] = ""
```

首先，让我们创建一个专用的 MLflow 实验，以便跟踪我们的模型和工件。虽然您可以选择跳过此步骤并使用默认实验，但我们强烈建议将您的运行和工件组织到单独的实验中，以避免混乱并保持清晰、结构化的工作流程。

```python
import mlflow

mlflow.set_experiment("LangChain MLflow Integration")
```

## 概述

通过以下方法将 MLflow 集成到您的 LangChain 应用程序中：

1. **自动记录**：使用 `mlflow.langchain.autolog()` 命令启用无缝跟踪，这是我们推荐的首选选项，用于利用 LangChain 和 MLflow 的集成。
2. **手动记录**：使用 MLflow API 记录 LangChain 链和代理，提供对实验中需要跟踪内容的细粒度控制。
3. **自定义回调**：在调用链时手动传递 MLflow 回调，允许对您的工作负载进行半自动化的定制，例如跟踪特定的调用。

## 场景 1：MLFlow 自动日志记录

要开始使用自动日志记录，只需调用 `mlflow.langchain.autolog()`。在此示例中，我们将 `log_models` 参数设置为 `True`，这允许将链定义及其依赖库记录为 MLflow 模型，从而提供全面的跟踪体验。

```python
import mlflow

mlflow.langchain.autolog(
    # These are optional configurations to control what information should be logged automatically (default: False)
    # For the full list of the arguments, refer to https://mlflow.org/docs/latest/llms/langchain/autologging.html#id1
    log_models=True,
    log_input_examples=True,
    log_model_signatures=True,
)
```

### 定义一个链


```python
from langchain.schema.output_parser import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model_name="gpt-4o", temperature=0)
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant that translates {input_language} to {output_language}.",
        ),
        ("human", "{input}"),
    ]
)
parser = StrOutputParser()

chain = prompt | llm | parser
```

### 调用链

请注意，这一步骤可能比平常多花几秒钟，因为 MLflow 在后台运行多个任务，以便将模型、跟踪和工件记录到跟踪服务器。

```python
test_input = {
    "input_language": "English",
    "output_language": "German",
    "input": "I love programming.",
}

chain.invoke(test_input)
```



```output
'Ich liebe das Programmieren.'
```


花一点时间探索 MLflow 跟踪 UI，在这里您可以更深入地了解正在记录的信息。
* **跟踪** - 在实验中导航到“跟踪”选项卡，点击第一行的请求 ID 链接。显示的跟踪树可视化了您的链调用的调用栈，为您提供了每个组件在链中执行的深入见解。
* **MLflow 模型** - 由于我们设置了 `log_model=True`，MLflow 会自动创建一个 MLflow 运行以跟踪您的链定义。导航到最新的运行页面并打开“工件”选项卡，其中列出了作为 MLflow 模型记录的文件工件，包括依赖项、输入示例、模型签名等。

### 调用已记录的链

接下来，让我们加载模型并验证我们是否可以重现相同的预测，确保一致性和可靠性。

有两种加载模型的方法
1. `mlflow.langchain.load_model(MODEL_URI)` - 这将模型加载为原始的 LangChain 对象。
2. `mlflow.pyfunc.load_model(MODEL_URI)` - 这将在 `PythonModel` 包装器中加载模型，并通过 `predict()` API 封装预测逻辑，包含诸如模式强制等额外逻辑。

```python
# Replace YOUR_RUN_ID with the Run ID displayed on the MLflow UI
loaded_model = mlflow.langchain.load_model("runs:/{YOUR_RUN_ID}/model")
loaded_model.invoke(test_input)
```



```output
'Ich liebe Programmieren.'
```



```python
pyfunc_model = mlflow.pyfunc.load_model("runs:/{YOUR_RUN_ID}/model")
pyfunc_model.predict(test_input)
```



```output
['Ich liebe das Programmieren.']
```

### 配置自动日志记录

`mlflow.langchain.autolog()` 函数提供了多个参数，可以对记录到 MLflow 的工件进行细致控制。有关可用配置的完整列表，请参阅最新的 [MLflow LangChain 自动日志记录文档](https://mlflow.org/docs/latest/llms/langchain/autologging.html)。

## 场景 2：从代码手动记录代理


#### 先决条件

此示例使用 `SerpAPI`，一个搜索引擎 API，作为代理检索 Google 搜索结果的工具。LangChain 与 `SerpAPI` 原生集成，允许您仅用一行代码为代理配置该工具。

开始之前：

* 通过 pip 安装所需的 Python 包：`pip install google-search-results numexpr`。
* 在 [SerpAPI 官方网站](https://serpapi.com/) 创建一个账户并获取 API 密钥。
* 将 API 密钥设置为环境变量：`os.environ["SERPAPI_API_KEY"] = "YOUR_API_KEY"`

### 定义代理

在这个例子中，我们将代理定义**作为代码**记录，而不是直接提供 Python 对象并将其保存为序列化格式。这种方法提供了几个好处：

1. **无需序列化**：通过将模型保存为代码，我们避免了序列化的需要，这在处理不原生支持的组件时可能会出现问题。这种方法还消除了在不同环境中反序列化模型时可能出现的不兼容问题的风险。
2. **更好的透明度**：通过检查保存的代码文件，您可以获得有关模型功能的宝贵见解。这与像 pickle 这样的序列化格式形成对比，后者在加载回之前，模型的行为保持不透明，可能暴露出诸如远程代码执行等安全风险。

首先，创建一个单独的 `.py` 文件来定义代理实例。

为了节省时间，您可以运行以下单元生成一个包含代理定义代码的 Python 文件 `agent.py`。在实际开发场景中，您会在另一个笔记本或手工编写的 Python 脚本中定义它。

```python
script_content = """
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import ChatOpenAI
import mlflow

llm = ChatOpenAI(model_name="gpt-4o", temperature=0)
tools = load_tools(["serpapi", "llm-math"], llm=llm)
agent = initialize_agent(tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION)

# IMPORTANT: call set_model() to register the instance to be logged.
mlflow.models.set_model(agent)
"""

with open("agent.py", "w") as f:
    f.write(script_content)
```

### 记录代理

返回原始笔记本并运行以下单元，以记录您在 `agent.py` 文件中定义的代理。



```python
question = "How long would it take to drive to the Moon with F1 racing cars?"

with mlflow.start_run(run_name="search-math-agent") as run:
    info = mlflow.langchain.log_model(
        lc_model="agent.py",  # Specify the relative code path to the agent definition
        artifact_path="model",
        input_example=question,
    )

print("The agent is successfully logged to MLflow!")
```
```output
The agent is successfully logged to MLflow!
```
现在，打开 MLflow UI，并在运行详细信息页面中导航到“Artifacts”选项卡。您应该会看到 `agent.py` 文件已成功记录，以及其他模型工件，例如依赖项、输入示例等。

### 调用已登录的代理

现在加载代理并调用它。有两种方法可以加载模型


```python
# Let's turn on the autologging with default configuration, so we can see the trace for the agent invocation.
mlflow.langchain.autolog()

# Load the model back
agent = mlflow.pyfunc.load_model(info.model_uri)

# Invoke
agent.predict(question)
```
```output
Downloading artifacts: 100%|██████████| 10/10 [00:00<00:00, 331.57it/s]
```


```output
['It would take approximately 1194.5 hours to drive to the Moon with an F1 racing car.']
```


导航到实验中的 **"Traces"** 标签，并单击第一行的请求 ID 链接。该跟踪可视化代理在单次预测调用中如何执行多个任务：
1. 确定回答问题所需的子任务。
2. 搜索 F1 赛车的速度。
3. 搜索地球到月球的距离。
4. 使用 LLM 进行除法计算。

## 场景 3. 使用 MLflow 回调

**MLflow 回调** 提供了一种半自动化的方式来跟踪您的 LangChain 应用程序在 MLflow 中。主要有两个可用的回调：

1. **`MlflowLangchainTracer`：** 主要用于生成跟踪，适用于 `mlflow >= 2.14.0`。
2. **`MLflowCallbackHandler`：** 将指标和工件记录到 MLflow 跟踪服务器。

### MlflowLangchainTracer

当使用 `MlflowLangchainTracer` 回调调用链或代理时，MLflow 将自动生成调用堆栈的跟踪并记录到 MLflow 跟踪服务器。 结果与 `mlflow.langchain.autolog()` 完全相同，但这在您只想跟踪特定调用时特别有用。 自动记录适用于同一笔记本/脚本中的所有调用。

```python
from mlflow.langchain.langchain_tracer import MlflowLangchainTracer

mlflow_tracer = MlflowLangchainTracer()

# 此调用生成一个跟踪
chain.invoke(test_input, config={"callbacks": [mlflow_tracer]})

# 此调用不生成跟踪
chain.invoke(test_input)
```

#### 如何传递回调
 LangChain 支持两种传递回调实例的方法：(1) 请求时回调 - 将它们传递给 `invoke` 方法或与 `with_config()` 绑定 (2) 构造函数回调 - 在链构造函数中设置它们。 当使用 `MlflowLangchainTracer` 作为回调时，您 **必须使用请求时回调**。 如果在构造函数中设置，它只会将回调应用于顶层对象，导致无法传播到子组件，从而导致跟踪不完整。 有关此行为的更多信息，请参阅 [回调文档](https://python.langchain.com/v0.2/docs/concepts/#callbacks) 以获取更多详细信息。

```python
# 正确
chain.invoke(test_input, config={"callbacks": [mlflow_tracer]})
chain.with_config(callbacks=[mlflow_tracer])
# 错误
chain = TheNameOfSomeChain(callbacks=[mlflow_tracer])
```

#### 支持的方法

`MlflowLangchainTracer` 支持来自 [Runnable Interfaces](https://python.langchain.com/v0.1/docs/expression_language/interface/) 的以下调用方法。
- 标准接口：`invoke`、`stream`、`batch`
- 异步接口：`astream`、`ainvoke`、`abatch`、`astream_log`、`astream_events`

其他方法不保证完全兼容。

### MlflowCallbackHandler

`MlflowCallbackHandler` 是一个回调处理器，位于 LangChain Community 代码库中。

此回调可以用于链/代理调用，但必须通过调用 `flush_tracker()` 方法显式结束。

当使用回调调用链时，它执行以下操作：

1. 创建一个新的 MLflow 运行，或者如果在活动的 MLflow 实验中可用，则检索一个活动的运行。
2. 记录指标，例如 LLM 调用的数量、令牌使用情况和其他相关指标。如果链/代理包含 LLM 调用，并且您已安装 `spacy` 库，则记录文本复杂度指标，例如 `flesch_kincaid_grade`。
3. 作为 JSON 文件记录内部步骤（这是跟踪的遗留版本）。
4. 将链的输入和输出记录为 Pandas Dataframe。
5. 使用链/代理实例调用 `flush_tracker()` 方法，将链/代理记录为 MLflow 模型。



```python
from langchain_community.callbacks import MlflowCallbackHandler

mlflow_callback = MlflowCallbackHandler()

chain.invoke("What is LangChain callback?", config={"callbacks": [mlflow_callback]})

mlflow_callback.flush_tracker()
```

## 参考文献
要了解更多关于该功能的信息，并访问使用 LangChain 与 MLflow 的教程和示例，请参考 [MLflow 文档中的 LangChain 集成](https://mlflow.org/docs/latest/llms/langchain/index.html)。

`MLflow` 还提供了几个关于 `LangChain` 集成的 [教程](https://mlflow.org/docs/latest/llms/langchain/index.html#getting-started-with-the-mlflow-langchain-flavor-tutorials-and-guides) 和 [示例](https://github.com/mlflow/mlflow/tree/master/examples/langchain)：
- [快速入门](https://mlflow.org/docs/latest/llms/langchain/notebooks/langchain-quickstart.html)
- [RAG 教程](https://mlflow.org/docs/latest/llms/langchain/notebooks/langchain-retriever.html)
- [代理示例](https://github.com/mlflow/mlflow/blob/master/examples/langchain/simple_agent.py)
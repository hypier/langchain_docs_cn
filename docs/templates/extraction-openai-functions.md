# extraction-openai-functions

此模板使用 [OpenAI 函数调用](https://python.langchain.com/docs/modules/chains/how_to/openai_functions) 从非结构化输入文本中提取结构化输出。

提取输出模式可以在 `chain.py` 中设置。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 用法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package extraction-openai-functions
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add extraction-openai-functions
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from extraction_openai_functions import chain as extraction_openai_functions_chain

add_routes(app, extraction_openai_functions_chain, path="/extraction-openai-functions")
```

（可选）现在让我们配置 LangSmith。
LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。
您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。
如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接通过以下方式启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/extraction-openai-functions/playground](http://127.0.0.1:8000/extraction-openai-functions/playground) 访问游乐场  

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/extraction-openai-functions")
```
默认情况下，此包设置为提取论文的标题和作者，如 `chain.py` 文件中所指定。

默认情况下，LLM 由 OpenAI 函数提供支持。
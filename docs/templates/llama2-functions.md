# llama2-functions

此模板使用支持指定 JSON 输出架构的 [LLaMA2 模型](https://github.com/ggerganov/llama.cpp/blob/master/grammars/README.md) 从非结构化数据中提取结构化数据。

提取架构可以在 `chain.py` 中设置。

## 环境设置

这将使用一个由 Replicate 托管的 [LLaMA2-13b 模型](https://replicate.com/andreasjansson/llama-2-13b-chat-gguf/versions)。

确保在您的环境中设置 `REPLICATE_API_TOKEN`。

## 使用方法

要使用此软件包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此软件包作为唯一软件包安装，您可以执行：

```shell
langchain app new my-app --package llama2-functions
```

如果您想将其添加到现有项目中，可以直接运行：

```shell
langchain app add llama2-functions
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from llama2_functions import chain as llama2_functions_chain

add_routes(app, llama2_functions_chain, path="/llama2-functions")
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

如果您在此目录中，那么您可以直接启动一个 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/llama2-functions/playground](http://127.0.0.1:8000/llama2-functions/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/llama2-functions")
```
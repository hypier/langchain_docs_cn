# rag-semi-structured

此模板在半结构化数据上执行RAG，例如包含文本和表格的PDF。

参见[此烹饪书](https://github.com/langchain-ai/langchain/blob/master/cookbook/Semi_Structured_RAG.ipynb)作为参考。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

这使用 [Unstructured](https://unstructured-io.github.io/unstructured/) 进行 PDF 解析，需要一些系统级别的软件包安装。

在 Mac 上，您可以使用以下命令安装必要的软件包：

```shell
brew install tesseract poppler
```

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此包作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package rag-semi-structured
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-semi-structured
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_semi_structured import chain as rag_semi_structured_chain

add_routes(app, rag_semi_structured_chain, path="/rag-semi-structured")
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

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行于 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/rag-semi-structured/playground](http://127.0.0.1:8000/rag-semi-structured/playground) 访问游乐场。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-semi-structured")
```

有关如何连接到模板的更多详细信息，请参阅 Jupyter notebook `rag_semi_structured`。
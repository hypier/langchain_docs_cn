# propositional-retrieval

此模板演示了Chen等人提出的多向量索引策略，详见[Dense X Retrieval: What Retrieval Granularity Should We Use?](https://arxiv.org/abs/2312.06648)。该提示可以在[hub上试用](https://smith.langchain.com/hub/wfh/proposal-indexing)，引导LLM生成去上下文化的“命题”，这些命题可以被向量化以提高检索准确性。您可以在`proposal_chain.py`中查看完整定义。

## 存储

在这个演示中，我们使用 RecursiveUrlLoader 对一篇简单的学术论文进行索引，并将所有检索器信息存储在本地（使用 chroma 和存储在本地文件系统上的 bytestore）。您可以在 `storage.py` 中修改存储层。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 `gpt-3.5` 和 OpenAI Embeddings 类。

## 索引

通过运行以下命令创建索引：

```python
poetry install
poetry run python propositional_retrieval/ingest.py
```

## 使用方法

要使用此包，您应该首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package propositional-retrieval
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add propositional-retrieval
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from propositional_retrieval import chain

add_routes(app, chain, path="/propositional-retrieval")
```

（可选）现在让我们配置 LangSmith。
LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。
您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。
如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接启动一个 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/propositional-retrieval/playground](http://127.0.0.1:8000/propositional-retrieval/playground) 访问游乐场。

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/propositional-retrieval")
```
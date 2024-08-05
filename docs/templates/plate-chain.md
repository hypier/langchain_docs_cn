# plate-chain

此模板可以解析实验室板中的数据。

在生物化学或分子生物学的背景下，实验室板是用于以网格格式保存样本的常用工具。

这可以将结果数据解析为标准化格式（例如，JSON），以便进一步处理。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 使用

要使用 plate-chain，您必须安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

创建一个新的 LangChain 项目并将 plate-chain 作为唯一包安装，可以使用以下命令：

```shell
langchain app new my-app --package plate-chain
```

如果您希望将其添加到现有项目中，只需运行：

```shell
langchain app add plate-chain
```

然后将以下代码添加到您的 `server.py` 文件中：

```python
from plate_chain import chain as plate_chain

add_routes(app, plate_chain, path="/plate-chain")
```

（可选）要配置 LangSmith，以帮助跟踪、监视和调试 LangChain 应用程序，请使用以下代码：

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，可以直接通过以下命令启动 LangServe 实例：

```shell
langchain serve
```

这将在本地启动 FastAPI 应用，服务器运行在 
[http://localhost:8000](http://localhost:8000)

所有模板可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看
访问游乐场在 [http://127.0.0.1:8000/plate-chain/playground](http://127.0.0.1:8000/plate-chain/playground)  

您可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/plate-chain")
```
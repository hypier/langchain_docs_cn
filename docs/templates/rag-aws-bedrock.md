# rag-aws-bedrock

此模板旨在与 AWS Bedrock 服务连接，这是一个提供一组基础模型的托管服务器。

它主要使用 `Anthropic Claude` 进行文本生成，使用 `Amazon Titan` 进行文本嵌入，并利用 FAISS 作为向量存储。

有关 RAG 管道的更多信息，请参阅 [此笔记本](https://github.com/aws-samples/amazon-bedrock-workshop/blob/main/03_QuestionAnswering/01_qa_w_rag_claude.ipynb)。

## 环境设置

在使用此包之前，请确保您已配置 `boto3` 以与您的 AWS 账户配合使用。

有关如何设置和配置 `boto3` 的详细信息，请访问 [此页面](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration)。

此外，您需要安装 `faiss-cpu` 包以使用 FAISS 向量存储：

```bash
pip install faiss-cpu
```

您还应设置以下环境变量，以反映您的 AWS 配置文件和区域（如果您不是在使用 `default` AWS 配置文件和 `us-east-1` 区域）：

* `AWS_DEFAULT_REGION`
* `AWS_PROFILE`

## 使用方法

首先，安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装：

```shell
langchain app new my-app --package rag-aws-bedrock
```

要将此包添加到现有项目中：

```shell
langchain app add rag-aws-bedrock
```

然后将以下代码添加到您的 `server.py` 文件中：
```python
from rag_aws_bedrock import chain as rag_aws_bedrock_chain

add_routes(app, rag_aws_bedrock_chain, path="/rag-aws-bedrock")
```

（可选）如果您可以访问 LangSmith，您可以将其配置为跟踪、监控和调试 LangChain 应用程序。如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，可以直接通过以下方式启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 [http://localhost:8000](http://localhost:8000)

您可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板，并在 [http://127.0.0.1:8000/rag-aws-bedrock/playground](http://127.0.0.1:8000/rag-aws-bedrock/playground) 访问游乐场。

您可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-aws-bedrock")
```
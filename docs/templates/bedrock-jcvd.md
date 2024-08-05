# Bedrock JCVD 🕺🥋

## 概述

LangChain 模板，使用 [Anthropic 的 Claude 在 Amazon Bedrock 上](https://aws.amazon.com/bedrock/claude/) 使其表现得像 JCVD。

> 我是聊天机器人的弗雷德·阿斯泰尔！ 🕺

## 环境设置

### AWS 凭证

此模板使用 [Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)，即 AWS 的 Python SDK，来调用 [Amazon Bedrock](https://aws.amazon.com/bedrock/)。您 **必须** 配置 AWS 凭证 *和* AWS 区域才能发出请求。

> 有关如何执行此操作的信息，请参见 [AWS Boto3 文档](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html)（开发者指南 > 凭证）。

### 基础模型

默认情况下，此模板使用 [Anthropic's Claude v2](https://aws.amazon.com/about-aws/whats-new/2023/08/claude-2-foundation-model-anthropic-amazon-bedrock/) (`anthropic.claude-v2`)。

> 要请求访问特定模型，请查看 [Amazon Bedrock 用户指南](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html)（模型访问）

要使用不同的模型，请设置环境变量 `BEDROCK_JCVD_MODEL_ID`。基本模型的列表可在 [Amazon Bedrock 用户指南](https://docs.aws.amazon.com/bedrock/latest/userguide/model-ids-arns.html) 中找到（使用 API > API 操作 > 运行推理 > 基本模型 ID）。

> 可用模型的完整列表（包括基本模型和 [自定义模型](https://docs.aws.amazon.com/bedrock/latest/userguide/custom-models.html)）可在 [Amazon Bedrock 控制台](https://docs.aws.amazon.com/bedrock/latest/userguide/using-console.html) 的 **基础模型** 下找到，或通过调用 [`aws bedrock list-foundation-models`](https://docs.aws.amazon.com/cli/latest/reference/bedrock/list-foundation-models.html) 获取。

## 使用方法

要使用此软件包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的软件包安装，您可以执行：

```shell
langchain app new my-app --package bedrock-jcvd
```

如果您想将其添加到现有项目中，可以直接运行：

```shell
langchain app add bedrock-jcvd
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from bedrock_jcvd import chain as bedrock_jcvd_chain

add_routes(app, bedrock_jcvd_chain, path="/bedrock-jcvd")
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
[http://localhost:8000](http://localhost:8000)。

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。

我们还可以访问游乐场，地址为 [http://127.0.0.1:8000/bedrock-jcvd/playground](http://127.0.0.1:8000/bedrock-jcvd/playground)。
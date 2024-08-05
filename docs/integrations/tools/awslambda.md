---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/awslambda.ipynb
---

# AWS Lambda

>[`Amazon AWS Lambda`](https://aws.amazon.com/pm/lambda/) 是由 `Amazon Web Services` (`AWS`) 提供的无服务器计算服务。它帮助开发人员构建和运行应用程序和服务，而无需配置或管理服务器。这种无服务器架构使您能够专注于编写和部署代码，同时 AWS 自动处理运行应用程序所需的基础设施的扩展、补丁和管理。

本笔记本介绍了如何使用 `AWS Lambda` 工具。

通过将 `AWS Lambda` 包含在提供给代理的工具列表中，您可以授予您的代理在 AWS 云中调用代码的能力，以满足您的各种需求。

当代理使用 `AWS Lambda` 工具时，它将提供一个字符串类型的参数，该参数将通过事件参数传递给 Lambda 函数。

首先，您需要安装 `boto3` python 包。

```python
%pip install --upgrade --quiet  boto3 > /dev/null
%pip install --upgrade --quiet langchain-community
```

为了让代理使用该工具，您必须提供与您 Lambda 函数逻辑功能匹配的名称和描述。

您还必须提供您的函数名称。

请注意，由于此工具实际上只是 boto3 库的一个包装，您需要运行 `aws configure` 才能使用该工具。有关更多详细信息，请参见 [这里](https://docs.aws.amazon.com/cli/index.html)

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI

llm = OpenAI(temperature=0)

tools = load_tools(
    ["awslambda"],
    awslambda_tool_name="email-sender",
    awslambda_tool_description="sends an email with the specified content to test@testing123.com",
    function_name="testFunction1",
)

agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)

agent.run("Send an email to test@testing123.com saying hello world.")
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)
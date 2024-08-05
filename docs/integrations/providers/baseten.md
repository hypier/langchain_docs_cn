# Baseten

>[Baseten](https://baseten.co) 是一个提供您所需基础设施的供应商，以高效、可扩展和经济的方式部署和服务 ML 模型。

>作为一个模型推理平台，`Baseten` 是 LangChain 生态系统中的一个 `Provider`。 
`Baseten` 集成目前实现了一个 `Component`，即 LLMs，但未来会有更多计划！

>`Baseten` 允许您在专用 GPU 上运行开源模型，如 Llama 2 或 Mistral，以及运行专有或微调模型。如果您习惯于像 OpenAI 这样的供应商，使用 Baseten 会有一些不同之处：

>* 您是按使用的 GPU 时间付费，而不是按每个 token 付费。
>* 每个在 Baseten 上的模型都使用 [Truss](https://truss.baseten.co/welcome)，我们的开源模型打包框架，以实现最大程度的可定制性。
>* 虽然我们有一些 [与 OpenAI ChatCompletions 兼容的模型](https://docs.baseten.co/api-reference/openai)，您可以使用 `Truss` 定义自己的 I/O 规范。

>[了解更多](https://docs.baseten.co/deploy/lifecycle) 关于模型 ID 和部署的信息。

>在 [Baseten 文档](https://docs.baseten.co/) 中了解更多关于 Baseten 的信息。

## 安装与设置

要在 LangChain 中使用 Baseten 模型，您需要两个东西：

- 一个 [Baseten 账户](https://baseten.co)
- 一个 [API 密钥](https://docs.baseten.co/observability/api-keys)

将您的 API 密钥导出为名为 `BASETEN_API_KEY` 的环境变量。

```sh
export BASETEN_API_KEY="paste_your_api_key_here"
```

## LLMs

查看 [使用示例](/docs/integrations/llms/baseten)。

```python
from langchain_community.llms import Baseten
```
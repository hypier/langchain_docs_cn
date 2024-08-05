# 🦜️🔗 LangChain 中文文档

⚡ 构建上下文感知推理应用程序 ⚡

[![Release Notes](https://img.shields.io/github/release/langchain-ai/langchain?style=flat-square)](https://github.com/langchain-ai/langchain/releases)
[![CI](https://github.com/langchain-ai/langchain/actions/workflows/check_diffs.yml/badge.svg)](https://github.com/langchain-ai/langchain/actions/workflows/check_diffs.yml)
[![PyPI - License](https://img.shields.io/pypi/l/langchain-core?style=flat-square)](https://opensource.org/licenses/MIT)
[![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-core?style=flat-square)](https://pypistats.org/packages/langchain-core)
[![GitHub star chart](https://img.shields.io/github/stars/langchain-ai/langchain?style=flat-square)](https://star-history.com/#langchain-ai/langchain)
[![Open Issues](https://img.shields.io/github/issues-raw/langchain-ai/langchain?style=flat-square)](https://github.com/langchain-ai/langchain/issues)
[![Open in Dev Containers](https://img.shields.io/static/v1?label=Dev%20Containers&message=Open&color=blue&logo=visualstudiocode&style=flat-square)](https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/langchain-ai/langchain)
[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/langchain-ai/langchain)
[![Twitter](https://img.shields.io/twitter/url/https/twitter.com/langchainai.svg?style=social&label=Follow%20%40LangChainAI)](https://twitter.com/langchainai)

寻找 JS/TS 库？查看 [LangChain.js](https://github.com/langchain-ai/langchainjs)。

为了帮助您更快地将 LangChain 应用程序推向生产环境，请查看 [LangSmith](https://smith.langchain.com)。 
[LangSmith](https://smith.langchain.com) 是一个统一的开发者平台，用于构建、测试和监控 LLM 应用程序。 
填写 [此表单](https://www.langchain.com/contact-sales) 与我们的销售团队联系。

## 快速安装

使用 pip:
```bash
pip install langchain
```

使用 conda:
```bash
conda install langchain -c conda-forge
```

## 🤔 什么是 LangChain？

**LangChain** 是一个用于开发由大型语言模型（LLMs）驱动的应用程序的框架。

对于这些应用程序，LangChain 简化了整个应用程序生命周期：

- **开源库**：使用 LangChain 的开源 [构建模块](https://python.langchain.com/v0.2/docs/concepts#langchain-expression-language-lcel)、[组件](https://python.langchain.com/v0.2/docs/concepts) 和 [第三方集成](https://python.langchain.com/v0.2/docs/integrations/platforms/) 构建您的应用程序。使用 [LangGraph](/docs/concepts/#langgraph) 构建具有一流流媒体和人机协作支持的有状态代理。
- **生产化**：使用 [LangSmith](https://docs.smith.langchain.com/) 检查、监控和评估您的应用程序，以便您可以不断优化并自信地部署。
- **部署**：使用 [LangGraph Cloud](https://langchain-ai.github.io/langgraph/cloud/) 将您的 LangGraph 应用程序转换为生产就绪的 API 和助手。

### 开源库
- **`langchain-core`**：基础抽象和LangChain表达语言。
- **`langchain-community`**：第三方集成。
  - 一些集成进一步拆分为**合作伙伴包**，仅依赖于**`langchain-core`**。示例包括**`langchain_openai`**和**`langchain_anthropic`**。
- **`langchain`**：构成应用程序认知架构的链、代理和检索策略。
- **[`LangGraph`](https://langchain-ai.github.io/langgraph/)**：一个用于构建强大且有状态的多参与者应用程序的库，通过将步骤建模为图中的边和节点，使用LLMs。与LangChain无缝集成，但也可以独立使用。

### 生产化：
- **[LangSmith](https://docs.smith.langchain.com/)**：一个开发者平台，使您能够调试、测试、评估和监控基于任何 LLM 框架构建的链，并与 LangChain 无缝集成。

### 部署：
- **[LangGraph Cloud](https://langchain-ai.github.io/langgraph/cloud/)**: 将您的 LangGraph 应用程序转变为可投入生产的 API 和助手。

![Diagram outlining the hierarchical organization of the LangChain framework, displaying the interconnected parts across multiple layers.](docs/static/svg/langchain_stack_062024.svg "LangChain Architecture Overview")

## 🧱 使用 LangChain 可以构建什么？

**❓ 使用 RAG 进行问答**

- [文档](https://python.langchain.com/v0.2/docs/tutorials/rag/)
- 端到端示例: [Chat LangChain](https://chat.langchain.com) 和 [repo](https://github.com/langchain-ai/chat-langchain)

**🧱 提取结构化输出**

- [文档](https://python.langchain.com/v0.2/docs/tutorials/extraction/)
- 端到端示例: [SQL Llama2 模板](https://github.com/langchain-ai/langchain-extract/)

**🤖 聊天机器人**

- [文档](https://python.langchain.com/v0.2/docs/tutorials/chatbot/)
- 端到端示例: [Web LangChain (网页研究聊天机器人)](https://weblangchain.vercel.app) 和 [repo](https://github.com/langchain-ai/weblangchain)

还有更多！请访问文档的 [教程](https://python.langchain.com/v0.2/docs/tutorials/) 部分以获取更多信息。

## 🚀 LangChain 如何提供帮助？
LangChain 库的主要价值主张包括：
1. **组件**：用于处理语言模型的可组合构建块、工具和集成。组件是模块化的，易于使用，无论您是否使用 LangChain 框架的其余部分。
2. **现成链**：内置的组件组合，用于完成更高级的任务。

现成链使得入门变得简单。组件使得自定义现有链和构建新链变得容易。

## LangChain 表达语言 (LCEL)

LCEL 是许多 LangChain 组件的基础，是一种声明式的链式组合方式。LCEL 从第一天起就设计为支持将原型投入生产，无需代码更改，从最简单的“提示 + LLM”链到最复杂的链。

- **[概述](https://python.langchain.com/v0.2/docs/concepts/#langchain-expression-language-lcel)**: LCEL 及其优势
- **[接口](https://python.langchain.com/v0.2/docs/concepts/#runnable-interface)**: LCEL 对象的标准 Runnable 接口
- **[原语](https://python.langchain.com/v0.2/docs/how_to/#langchain-expression-language-lcel)**: 关于 LCEL 包含的原语的更多信息
- **[备忘单](https://python.langchain.com/v0.2/docs/how_to/lcel_cheatsheet/)**: 最常见使用模式的快速概述

## 组件

组件分为以下 **模块**：

**📃 模型输入/输出**

这包括 [提示管理](https://python.langchain.com/v0.2/docs/concepts/#prompt-templates)、[提示优化](https://python.langchain.com/v0.2/docs/concepts/#example-selectors)、一个通用接口用于 [聊天模型](https://python.langchain.com/v0.2/docs/concepts/#chat-models) 和 [LLMs](https://python.langchain.com/v0.2/docs/concepts/#llms)，以及处理 [模型输出](https://python.langchain.com/v0.2/docs/concepts/#output-parsers) 的常用工具。

**📚 检索**

检索增强生成涉及从各种来源 [加载数据](https://python.langchain.com/v0.2/docs/concepts/#document-loaders)、[准备数据](https://python.langchain.com/v0.2/docs/concepts/#text-splitters)，然后 [进行搜索（即从中检索）](https://python.langchain.com/v0.2/docs/concepts/#retrievers)，以便在生成步骤中使用。

**🤖 代理**

代理允许 LLM 自主决定如何完成任务。代理会决定采取哪些行动，然后执行该行动，观察结果，并重复该过程直到任务完成。LangChain 提供了 [代理的标准接口](https://python.langchain.com/v0.2/docs/concepts/#agents)，以及用于构建自定义代理的 [LangGraph](https://github.com/langchain-ai/langgraph)。

## 📖 文档

请查看 [这里](https://python.langchain.com) 获取完整文档，其中包括：

- [介绍](https://python.langchain.com/v0.2/docs/introduction/): 框架概述和文档结构。
- [教程](https://python.langchain.com/docs/use_cases/): 如果您想构建特定内容或更喜欢动手学习，请查看我们的教程。这是开始的最佳地方。
- [操作指南](https://python.langchain.com/v0.2/docs/how_to/): 解答“我该如何….？”类型的问题。这些指南以目标为导向，具体明确；旨在帮助您完成特定任务。
- [概念指南](https://python.langchain.com/v0.2/docs/concepts/): 框架关键部分的概念性解释。
- [API 参考](https://api.python.langchain.com): 每个类和方法的详细文档。

## 🌐 生态系统

- [🦜🛠️ LangSmith](https://docs.smith.langchain.com/): 跟踪和评估您的语言模型应用程序和智能代理，帮助您从原型转向生产环境。
- [🦜🕸️ LangGraph](https://langchain-ai.github.io/langgraph/): 创建有状态的多参与者应用程序，使用LLMs。与LangChain无缝集成，但也可以独立使用。
- [🦜🏓 LangServe](https://python.langchain.com/docs/langserve): 将LangChain可运行组件和链部署为REST API。

## 💁 贡献

作为一个快速发展的领域中的开源项目，我们非常欢迎各种形式的贡献，无论是新功能、改善基础设施，还是更好的文档。

有关如何贡献的详细信息，请参见 [这里](https://python.langchain.com/v0.2/docs/contributing/)。

## 🌟 贡献者

[![langchain contributors](https://contrib.rocks/image?repo=langchain-ai/langchain&max=2000)](https://github.com/langchain-ai/langchain/graphs/contributors)
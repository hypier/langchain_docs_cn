# rag-conversation-zep

此模板演示了如何使用 Zep 构建 RAG 对话应用程序。

此模板中包含：
- 使用一组文档填充 [Zep 文档集合](https://docs.getzep.com/sdk/documents/)（集合类似于其他向量数据库中的索引）。
- 使用 Zep 的 [集成嵌入](https://docs.getzep.com/deployment/embeddings/) 功能将文档嵌入为向量。
- 配置 LangChain [ZepVectorStore 检索器](https://docs.getzep.com/sdk/documents/)，使用 Zep 内置的、硬件加速的 [最大边际相关性](https://docs.getzep.com/sdk/search_query/) (MMR) 重新排名来检索文档。
- 提示、简单的聊天历史数据结构以及构建 RAG 对话应用程序所需的其他组件。
- RAG 对话链。

## 关于 [Zep - 快速、可扩展的 LLM 应用构建模块](https://www.getzep.com/)
Zep 是一个用于生产化 LLM 应用的开源平台。您可以在几分钟内将基于 LangChain 或 LlamaIndex 构建的原型或自定义应用程序投入生产，而无需重写代码。

主要特点：

- 快速！Zep 的异步提取器独立于您的聊天循环操作，确保用户体验流畅。
- 长期记忆持久性，无论您的摘要策略如何，都可以访问历史消息。
- 基于可配置消息窗口的记忆消息自动摘要。一系列摘要被存储，为未来的摘要策略提供灵活性。
- 对记忆和元数据的混合搜索，消息在创建时自动嵌入。
- 实体提取器自动从消息中提取命名实体并将其存储在消息元数据中。
- 记忆和摘要的自动令牌计数，允许对提示组装进行更细粒度的控制。
- Python 和 JavaScript SDK。

Zep 项目： https://github.com/getzep/zep | 文档： https://docs.getzep.com/

## 环境设置

通过遵循 [快速入门指南](https://docs.getzep.com/deployment/quickstart/) 来设置 Zep 服务。

## 将文档导入 Zep 集合

运行 `python ingest.py` 将测试文档导入 Zep 集合。查看文件以修改集合名称和文档来源。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的 LangChain 项目并将此作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package rag-conversation-zep
```

如果您想将其添加到现有项目中，可以直接运行：

```shell
langchain app add rag-conversation-zep
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_conversation_zep import chain as rag_conversation_zep_chain

add_routes(app, rag_conversation_zep_chain, path="/rag-conversation-zep")
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

如果您在此目录内，则可以直接通过以下方式启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/rag-conversation-zep/playground](http://127.0.0.1:8000/rag-conversation-zep/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-conversation-zep")
```
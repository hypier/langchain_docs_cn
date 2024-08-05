---
sidebar_class_name: hidden
custom_edit_url:
---

# 模板

突出显示几种不同类别的模板

## ⭐ 热门

这些是一些更受欢迎的模板，供您入门使用。

- [检索增强生成聊天机器人](/docs/templates/rag-conversation): 在您的数据上构建一个聊天机器人。默认使用 OpenAI 和 PineconeVectorStore。
- [使用 OpenAI 函数进行提取](/docs/templates/extraction-openai-functions): 从非结构化数据中提取结构化数据。使用 OpenAI 函数调用。
- [本地检索增强生成](/docs/templates/rag-chroma-private): 在您的数据上构建一个聊天机器人。仅使用本地工具：Ollama、GPT4all、Chroma。
- [OpenAI 函数代理](/docs/templates/openai-functions-agent): 构建一个可以采取行动的聊天机器人。使用 OpenAI 函数调用和 Tavily。
- [XML 代理](/docs/templates/xml-agent): 构建一个可以采取行动的聊天机器人。使用 Anthropic 和 You.com。

## 📥 高级检索

这些模板涵盖了高级检索技术，可用于数据库或文档的聊天和问答。

- [重新排序](/docs/templates/rag-pinecone-rerank): 该检索技术使用Cohere的重新排序端点对初始检索步骤中的文档进行重新排序。
- [Anthropic迭代搜索](/docs/templates/anthropic-iterative-search): 该检索技术使用迭代提示来确定检索内容以及检索到的文档是否足够好。
- **父文档检索** 使用 [Neo4j](/docs/templates/neo4j-parent) 或 [MongoDB](/docs/templates/mongo-parent-document-retrieval): 该检索技术存储较小块的嵌入，但随后返回较大块以传递给模型进行生成。
- [半结构化RAG](/docs/templates/rag-semi-structured): 该模板展示了如何对半结构化数据（例如，涉及文本和表格的数据）进行检索。
- [时间RAG](/docs/templates/rag-timescale-hybrid-search-time): 该模板展示了如何对具有时间组件的数据进行混合搜索，使用 [Timescale Vector](https://www.timescale.com/ai?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral)。

## 🔍高级检索 - 查询转换

一系列涉及转换原始用户查询的高级检索方法，这可以提高检索质量。

- [假设文档嵌入](/docs/templates/hyde)：一种检索技术，为给定查询生成假设文档，然后使用该文档的嵌入进行语义搜索。[论文](https://arxiv.org/abs/2212.10496)。
- [重写-检索-阅读](/docs/templates/rewrite-retrieve-read)：一种在将给定查询传递给搜索引擎之前重写查询的检索技术。[论文](https://arxiv.org/abs/2305.14283)。
- [回退问答提示](/docs/templates/stepback-qa-prompting)：一种生成“回退”问题的检索技术，然后检索与该问题及原始问题相关的文档。[论文](https://arxiv.org/abs//2310.06117)。
- [RAG融合](/docs/templates/rag-fusion)：一种生成多个查询的检索技术，然后使用互惠排名融合对检索到的文档进行重新排序。[文章](https://towardsdatascience.com/forget-rag-the-future-is-rag-fusion-1147298d8ad1)。
- [多查询检索器](/docs/templates/rag-pinecone-multi-query)：该检索技术使用LLM生成多个查询，然后为所有查询获取文档。

## 🧠高级检索 - 查询构建

一系列高级检索方法，涉及在与自然语言不同的DSL中构建查询，从而实现自然语言与各种结构化数据库之间的对话。

- [Elastic Query Generator](/docs/templates/elastic-query-generator): 从自然语言生成弹性搜索查询。
- [Neo4j Cypher Generation](/docs/templates/neo4j-cypher): 从自然语言生成Cypher语句。也提供 ["全文" 选项](/docs/templates/neo4j-cypher-ft)。
- [Supabase Self Query](/docs/templates/self-query-supabase): 将自然语言查询解析为语义查询以及Supabase的元数据过滤器。

## 🦙 OSS 模型

这些模板使用 OSS 模型，可以为敏感数据提供隐私保护。

- [本地检索增强生成](/docs/templates/rag-chroma-private): 在您的数据上构建聊天机器人。仅使用本地工具：Ollama, GPT4all, Chroma。
- [SQL 问答（Replicate）](/docs/templates/sql-llama2): 在 SQL 数据库上进行问答，使用托管在 [Replicate](https://replicate.com/) 上的 Llama2。
- [SQL 问答（LlamaCpp）](/docs/templates/sql-llamacpp): 在 SQL 数据库上进行问答，使用通过 [LlamaCpp](https://github.com/ggerganov/llama.cpp) 的 Llama2。
- [SQL 问答（Ollama）](/docs/templates/sql-ollama): 在 SQL 数据库上进行问答，使用通过 [Ollama](https://github.com/jmorganca/ollama) 的 Llama2。

## ⛏️ 数据提取

这些模板根据用户指定的架构以结构化格式提取数据。

- [使用 OpenAI 函数提取](/docs/templates/extraction-openai-functions): 使用 OpenAI 函数调用从文本中提取信息。
- [使用 Anthropic 函数提取](/docs/templates/extraction-anthropic-functions): 使用 LangChain 包装器围绕 Anthropic 端点提取文本中的信息，旨在模拟函数调用。
- [提取生物技术板数据](/docs/templates/plate-chain): 从杂乱的 Excel 电子表格中提取微孔板数据，转换为更规范的格式。

## ⛏️总结与标签

这些模板用于总结或分类文档和文本。

- [使用Anthropic进行总结](/docs/templates/summarize-anthropic): 使用Anthropic的Claude2对长文档进行总结。

## 🤖 代理

这些模板构建可以采取行动的聊天机器人，帮助自动化任务。

- [OpenAI Functions Agent](/docs/templates/openai-functions-agent): 构建一个可以采取行动的聊天机器人。使用OpenAI函数调用和Tavily。
- [XML Agent](/docs/templates/xml-agent): 构建一个可以采取行动的聊天机器人。使用Anthropic和You.com。

## :rotating_light: 安全与评估

这些模板支持对LLM输出进行审核或评估。

- [Guardrails Output Parser](/docs/templates/guardrails-output-parser): 使用guardrails-ai验证LLM输出。
- [Chatbot Feedback](/docs/templates/chat-bot-feedback): 使用LangSmith评估聊天机器人响应。
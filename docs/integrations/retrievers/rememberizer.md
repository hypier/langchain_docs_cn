---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/rememberizer.ipynb
---

# Rememberizer

>[Rememberizer](https://rememberizer.ai/) 是由 SkyDeck AI Inc. 创建的 AI 应用程序知识增强服务。

本笔记本展示了如何将文档从 `Rememberizer` 检索到下游使用的文档格式。

# 准备

您需要一个 API 密钥：您可以在 [https://rememberizer.ai](https://rememberizer.ai/) 创建一个公共知识后获取。获得 API 密钥后，您必须将其设置为环境变量 `REMEMBERIZER_API_KEY`，或者在初始化 `RememberizerRetriever` 时将其作为 `rememberizer_api_key` 传递。

`RememberizerRetriever` 具有以下参数：
- 可选的 `top_k_results`：默认值为 10。用于限制返回的文档数量。
- 可选的 `rememberizer_api_key`：如果您没有设置环境变量 `REMEMBERIZER_API_KEY`，则该参数是必需的。

`get_relevant_documents()` 有一个参数，`query`：用于在 `Rememberizer.ai` 的公共知识中查找文档的自由文本。

# 示例

## 基本用法


```python
# 设置 API 密钥
from getpass import getpass

REMEMBERIZER_API_KEY = getpass()
```


```python
import os

from langchain_community.retrievers import RememberizerRetriever

os.environ["REMEMBERIZER_API_KEY"] = REMEMBERIZER_API_KEY
retriever = RememberizerRetriever(top_k_results=5)
```


```python
docs = retriever.get_relevant_documents(query="大型语言模型是如何工作的？")
```


```python
docs[0].metadata  # 文档的元信息
```



```output
{'id': 13646493,
 'document_id': '17s3LlMbpkTk0ikvGwV0iLMCj-MNubIaP',
 'name': 'What is a large language model (LLM)_ _ Cloudflare.pdf',
 'type': 'application/pdf',
 'path': '/langchain/What is a large language model (LLM)_ _ Cloudflare.pdf',
 'url': 'https://drive.google.com/file/d/17s3LlMbpkTk0ikvGwV0iLMCj-MNubIaP/view',
 'size': 337089,
 'created_time': '',
 'modified_time': '',
 'indexed_on': '2024-04-04T03:36:28.886170Z',
 'integration': {'id': 347, 'integration_type': 'google_drive'}}
```



```python
print(docs[0].page_content[:400])  # 文档的内容
```
```output
before, or contextualized in new ways. on some level they " understand " semantics in that they can associate words and concepts by their meaning, having seen them grouped together in that way millions or billions of times. how developers can quickly start building their own llms to build llm applications, developers need easy access to multiple data sets, and they need places for those data sets
```

# 链中的使用


```python
OPENAI_API_KEY = getpass()
```


```python
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```


```python
from langchain.chains import ConversationalRetrievalChain
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model_name="gpt-3.5-turbo")
qa = ConversationalRetrievalChain.from_llm(model, retriever=retriever)
```


```python
questions = [
    "什么是RAG？",
    "大型语言模型是如何工作的？",
]
chat_history = []

for question in questions:
    result = qa.invoke({"question": question, "chat_history": chat_history})
    chat_history.append((question, result["answer"]))
    print(f"-> **问题**: {question} \n")
    print(f"**回答**: {result['answer']} \n")
```
```output
-> **问题**: 什么是RAG？ 

**回答**: RAG代表检索增强生成。它是一个人工智能框架，从外部知识库中检索事实，以通过提供最新和准确的信息来增强大型语言模型（LLMs）生成的响应。该框架帮助用户理解LLMs的生成过程，并确保模型能够访问可靠的信息来源。 

-> **问题**: 大型语言模型是如何工作的？ 

**回答**: 大型语言模型（LLMs）通过分析大量语言数据集来理解和生成自然语言文本。它们基于机器学习，特别是深度学习，涉及训练一个程序在没有人工干预的情况下识别数据特征。LLMs使用神经网络，特别是变压器模型，来理解人类语言的上下文，使它们在模糊或新颖的上下文中更好地解释语言。开发人员可以通过访问多个数据集并使用Cloudflare的Vectorize和Cloudflare Workers AI平台等服务，快速开始构建自己的LLMs。
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)
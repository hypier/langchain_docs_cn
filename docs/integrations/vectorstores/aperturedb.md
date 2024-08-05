---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/aperturedb.ipynb
---

# ApertureDB

[ApertureDB](https://docs.aperturedata.io) 是一个数据库，用于存储、索引和管理多模态数据，如文本、图像、视频、边界框和嵌入，以及它们相关的元数据。

本笔记本解释了如何使用 ApertureDB 的嵌入功能。

## 安装 ApertureDB Python SDK

这将安装用于编写 ApertureDB 客户端代码的 [Python SDK](https://docs.aperturedata.io/category/aperturedb-python-sdk)。

```python
%pip install --upgrade --quiet aperturedb
```
```output
Note: you may need to restart the kernel to use updated packages.
```

## 运行 ApertureDB 实例

要继续，您应该有一个 [ApertureDB 实例正在运行](https://docs.aperturedata.io/HowToGuides/start/Setup)，并配置您的环境以使用它。  
有多种方法可以做到这一点，例如：

```bash
docker run --publish 55555:55555 aperturedata/aperturedb-standalone
adb config create local --active --no-interactive
```

## 下载一些网页文档
我们将在这里对一个网页进行小规模爬取。

```python
# For loading documents from web
from langchain_community.document_loaders import WebBaseLoader

loader = WebBaseLoader("https://docs.aperturedata.io")
docs = loader.load()
```
```output
USER_AGENT environment variable not set, consider setting it to identify your requests.
```

## 选择嵌入模型

我们想要使用 OllamaEmbeddings，因此我们需要导入必要的模块。

Ollama 可以按照 [文档](https://hub.docker.com/r/ollama/ollama) 中的描述设置为 Docker 容器，例如：
```bash
# 运行服务器
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
# 告诉服务器加载特定模型
docker exec ollama ollama run llama2
```


```python
from langchain_community.embeddings import OllamaEmbeddings

embeddings = OllamaEmbeddings()
```

## 将文档拆分为多个部分

我们希望将单个文档转换为多个部分。


```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter()
documents = text_splitter.split_documents(docs)
```

## 从文档和嵌入创建向量存储

此代码在ApertureDB实例中创建一个向量存储。在该实例中，此向量存储表示为一个"[descriptor set](https://docs.aperturedata.io/category/descriptorset-commands)"。默认情况下，descriptor set被命名为`langchain`。以下代码将为每个文档生成嵌入并将其作为描述符存储在ApertureDB中。这将花费几秒钟，因为正在生成嵌入。

```python
from langchain_community.vectorstores import ApertureDB

vector_db = ApertureDB.from_documents(documents, embeddings)
```

## 选择一个大型语言模型

再次，我们使用为本地处理设置的Ollama服务器。


```python
from langchain_community.llms import Ollama

llm = Ollama(model="llama2")
```

## 构建 RAG 链

现在我们拥有创建 RAG（检索增强生成）链所需的所有组件。该链执行以下操作：
1. 为用户查询生成嵌入描述符
2. 使用向量存储查找与用户查询相似的文本片段
3. 使用提示模板将用户查询和上下文文档传递给 LLM
4. 返回 LLM 的答案


```python
# Create prompt
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_template("""Answer the following question based only on the provided context:

<context>
{context}
</context>

Question: {input}""")


# Create a chain that passes documents to an LLM
from langchain.chains.combine_documents import create_stuff_documents_chain

document_chain = create_stuff_documents_chain(llm, prompt)


# Treat the vectorstore as a document retriever
retriever = vector_db.as_retriever()


# Create a RAG chain that connects the retriever to the LLM
from langchain.chains import create_retrieval_chain

retrieval_chain = create_retrieval_chain(retriever, document_chain)
```
```output
Based on the provided context, ApertureDB can store images. In fact, it is specifically designed to manage multimodal data such as images, videos, documents, embeddings, and associated metadata including annotations. So, ApertureDB has the capability to store and manage images.
```

## 运行 RAG 链

最后，我们将问题传递给链并得到我们的答案。这将需要几秒钟的时间，因为 LLM 从查询和上下文文档中生成答案。

```python
user_query = "How can ApertureDB store images?"
response = retrieval_chain.invoke({"input": user_query})
print(response["answer"])
```
```output
根据提供的上下文，ApertureDB 可以以几种方式存储图像：

1. 多模态数据管理：ApertureDB 提供统一的接口来管理多模态数据，如图像、视频、文档、嵌入和相关元数据，包括注释。这意味着图像可以与其他类型的数据一起存储在单个数据库实例中。
2. 图像存储：ApertureDB 通过与公共云提供商或本地安装的集成提供图像存储能力。这允许客户托管自己的 ApertureDB 实例，并在他们首选的云提供商或本地基础设施上存储图像。
3. 向量数据库：ApertureDB 还提供向量数据库，能够基于图像的语义意义进行高效的相似性搜索和分类。这在图像搜索和分类很重要的应用中非常有用，例如计算机视觉或机器学习工作流。

总体而言，ApertureDB 为图像提供灵活且可扩展的存储选项，允许客户选择最适合其需求的部署模型。
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
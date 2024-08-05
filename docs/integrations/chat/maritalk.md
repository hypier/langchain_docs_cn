---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/maritalk.ipynb
---
<a href="https://colab.research.google.com/github/langchain-ai/langchain/blob/master/docs/docs/integrations/chat/maritalk.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

# 海事技术

## 介绍

MariTalk 是由巴西公司 [Maritaca AI](https://www.maritaca.ai) 开发的助手。  
MariTalk 基于经过特别训练的语言模型，能够很好地理解葡萄牙语。

本笔记本演示了如何通过两个示例使用 MariTalk 与 LangChain：

1. MariTalk 执行任务的简单示例。  
2. LLM + RAG：第二个示例展示了如何回答一个答案在不符合 MariTalk 的令牌限制的长文档中的问题。为此，我们将使用一个简单的搜索器 (BM25) 首先搜索文档中最相关的部分，然后将其提供给 MariTalk 进行回答。

## 安装
首先，使用以下命令安装 LangChain 库（及其所有依赖项）：

```python
!pip install langchain langchain-core langchain-community httpx
```

## API 密钥
您需要一个可以从 chat.maritaca.ai 获取的 API 密钥（“Chaves da API” 部分）。

### 示例 1 - 宠物名称建议

让我们定义我们的语言模型 ChatMaritalk，并用您的 API 密钥进行配置。

```python
from langchain_community.chat_models import ChatMaritalk
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts.chat import ChatPromptTemplate

llm = ChatMaritalk(
    model="sabia-2-medium",  # 可用模型：sabia-2-small 和 sabia-2-medium
    api_key="",  # 在此处插入您的 API 密钥
    temperature=0.7,
    max_tokens=100,
)

output_parser = StrOutputParser()

chat_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "您是一个专门建议宠物名称的助手。根据动物，您必须建议 4 个名称。",
        ),
        ("human", "我有一只 {animal}"),
    ]
)

chain = chat_prompt | llm | output_parser

response = chain.invoke({"animal": "dog"})
print(response)  # 应该回答类似 "1. Max\n2. Bella\n3. Charlie\n4. Rocky"
```

### 流生成

对于涉及生成长文本的任务，例如创建一篇详尽的文章或翻译一份大型文档，按部分接收响应可能更具优势，因为文本是逐步生成的，而不是等待完整文本。这使得应用程序更加灵活和高效，特别是在生成的文本较长时。我们提供两种方法来满足这一需求：一种是同步的，另一种是异步的。

#### 同步：

```python
from langchain_core.messages import HumanMessage

messages = [HumanMessage(content="Suggest 3 names for my dog")]

for chunk in llm.stream(messages):
    print(chunk.content, end="", flush=True)
```

#### 异步：

```python
from langchain_core.messages import HumanMessage


async def async_invoke_chain(animal: str):
    messages = [HumanMessage(content=f"Suggest 3 names for my {animal}")]
    async for chunk in llm._astream(messages):
        print(chunk.message.content, end="", flush=True)


await async_invoke_chain("dog")
```

### 示例 2 - RAG + LLM：UNICAMP 2024 入学考试问答系统
对于这个示例，我们需要安装一些额外的库：

```python
!pip install unstructured rank_bm25 pdf2image pdfminer-six pikepdf pypdf unstructured_inference fastapi kaleido uvicorn "pillow<10.1.0" pillow_heif -q
```

#### 加载数据库

第一步是创建一个包含通知信息的数据库。为此，我们将从 COMVEST 网站下载通知，并将提取的文本分段为 500 个字符的窗口。

```python
from langchain_community.document_loaders import OnlinePDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter

# 加载 COMVEST 2024 通知
loader = OnlinePDFLoader(
    "https://www.comvest.unicamp.br/wp-content/uploads/2023/10/31-2023-Dispoe-sobre-o-Vestibular-Unicamp-2024_com-retificacao.pdf"
)
data = loader.load()

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500, chunk_overlap=100, separators=["\n", " ", ""]
)
texts = text_splitter.split_documents(data)
```

#### 创建搜索器
现在我们有了数据库，我们需要一个搜索器。对于这个示例，我们将使用简单的 BM25 作为搜索系统，但这可以替换为任何其他搜索器（例如通过嵌入进行搜索）。

```python
from langchain_community.retrievers import BM25Retriever

retriever = BM25Retriever.from_documents(texts)
```

#### 结合搜索系统 + LLM
现在我们有了搜索器，我们只需实现一个提示，指定任务并调用链。

```python
from langchain.chains.question_answering import load_qa_chain

prompt = """基于以下文档，回答下面的问题。

{context}

问题：{query}
"""

qa_prompt = ChatPromptTemplate.from_messages([("human", prompt)])

chain = load_qa_chain(llm, chain_type="stuff", verbose=True, prompt=qa_prompt)

query = "考试的最长时间是多少？"

docs = retriever.invoke(query)

chain.invoke(
    {"input_documents": docs, "query": query}
)  # 应该输出类似于：“考试的最长时间是 5 小时。”
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)
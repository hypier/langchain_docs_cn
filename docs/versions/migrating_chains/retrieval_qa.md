---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/versions/migrating_chains/retrieval_qa.ipynb
title: 从 RetrievalQA 迁移
---

[`RetrievalQA`](https://api.python.langchain.com/en/latest/chains/langchain.chains.retrieval_qa.base.RetrievalQA.html) 链通过检索增强生成在数据源上执行自然语言问答。

切换到 LCEL 实现的一些优势包括：

- 更容易自定义。诸如提示和文档格式化等细节仅通过 `RetrievalQA` 链中的特定参数进行配置。
- 更容易返回源文档。
- 支持可运行的方法，如流式和异步操作。

现在让我们并排查看它们。我们将使用相同的摄取代码将 [Lilian Weng 的博客文章](https://lilianweng.github.io/posts/2023-06-23-agent/) 关于自主代理加载到本地向量存储中：

```python
%pip install --upgrade --quiet langchain-community langchain langchain-openai faiss-cpu
```

```python
import os
from getpass import getpass

os.environ["OPENAI_API_KEY"] = getpass()
```

```python
# 加载文档
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import WebBaseLoader
from langchain_community.vectorstores import FAISS
from langchain_openai.chat_models import ChatOpenAI
from langchain_openai.embeddings import OpenAIEmbeddings

loader = WebBaseLoader("https://lilianweng.github.io/posts/2023-06-23-agent/")
data = loader.load()

# 切分
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
all_splits = text_splitter.split_documents(data)

# 存储切分
vectorstore = FAISS.from_documents(documents=all_splits, embedding=OpenAIEmbeddings())

# LLM
llm = ChatOpenAI()
```

import { ColumnContainer, Column } from "@theme/Columns";

<ColumnContainer>

<Column>

#### 传统版


```python
from langchain import hub
from langchain.chains import RetrievalQA

# 查看完整提示 https://smith.langchain.com/hub/rlm/rag-prompt
prompt = hub.pull("rlm/rag-prompt")

qa_chain = RetrievalQA.from_llm(
    llm, retriever=vectorstore.as_retriever(), prompt=prompt
)

qa_chain("什么是自主代理？")
```


```output
{'query': '什么是自主代理？',
 'result': '自主代理是由大型语言模型（LLM）赋能的代理，能够处理复杂科学实验的自主设计、规划和执行。这些代理可以浏览互联网，阅读文档，执行代码，调用机器人实验 API，并利用其他 LLM。它们可以生成推理步骤，例如根据请求的任务开发新型抗癌药物。'}
```


</Column>

<Column>

#### LCEL




```python
from langchain import hub
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# 查看完整提示 https://smith.langchain.com/hub/rlm/rag-prompt
prompt = hub.pull("rlm/rag-prompt")


def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)


qa_chain = (
    {
        "context": vectorstore.as_retriever() | format_docs,
        "question": RunnablePassthrough(),
    }
    | prompt
    | llm
    | StrOutputParser()
)

qa_chain.invoke("什么是自主代理？")
```


```output
'自主代理是由大型语言模型（LLMs）赋能的代理，能够处理自主设计、规划和执行复杂任务，例如科学实验。这些代理可以使用工具浏览互联网，阅读文档，执行代码，调用机器人实验 API，并利用其他 LLM 来完成任务。当给定特定任务时，模型可以提出推理步骤，例如开发新型抗癌药物。'
```


</Column>
</ColumnContainer>

LCEL 实现揭示了检索、格式化文档和将其传递给 LLM 的内部过程，但它更冗长。您可以在辅助函数中自定义和包装此组合逻辑，或使用更高级的 [`create_retrieval_chain`](https://api.python.langchain.com/en/latest/chains/langchain.chains.retrieval.create_retrieval_chain.html) 和 [`create_stuff_documents_chain`](https://api.python.langchain.com/en/latest/chains/langchain.chains.combine_documents.stuff.create_stuff_documents_chain.html) 辅助方法：

```python
from langchain import hub
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain

# 查看完整提示 https://smith.langchain.com/hub/langchain-ai/retrieval-qa-chat
retrieval_qa_chat_prompt = hub.pull("langchain-ai/retrieval-qa-chat")

combine_docs_chain = create_stuff_documents_chain(llm, retrieval_qa_chat_prompt)
rag_chain = create_retrieval_chain(vectorstore.as_retriever(), combine_docs_chain)

rag_chain.invoke({"input": "什么是自主代理？"})
```

## 下一步

查看 [LCEL 概念文档](/docs/concepts/#langchain-expression-language-lcel) 以获取更多背景信息。
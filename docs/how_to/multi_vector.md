---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/multi_vector.ipynb
---

# 如何使用每个文档的多个向量进行检索

存储每个文档的多个向量通常是有用的。这在多个用例中是有益的。例如，我们可以将文档的多个块嵌入并将这些嵌入与父文档关联，从而允许对块的检索命中返回更大的文档。

LangChain 实现了一个基础的 [MultiVectorRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain.retrievers.multi_vector.MultiVectorRetriever.html)，简化了这个过程。大部分复杂性在于如何为每个文档创建多个向量。本笔记本涵盖了一些创建这些向量的常见方法以及如何使用 `MultiVectorRetriever`。

为每个文档创建多个向量的方法包括：

- 较小的块：将文档拆分为较小的块，并嵌入这些块（这就是 [ParentDocumentRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain.retrievers.parent_document_retriever.ParentDocumentRetriever.html)）。
- 摘要：为每个文档创建摘要，将其与文档一起嵌入（或替代文档）。
- 假设性问题：创建适合每个文档回答的假设性问题，将这些问题与文档一起嵌入（或替代文档）。

请注意，这还启用了一种添加嵌入的其他方法 - 手动添加。这是有用的，因为您可以明确添加应该导致文档被恢复的问题或查询，从而给予您更多控制。

下面我们将通过一个示例进行说明。首先，我们实例化一些文档。我们将使用 [OpenAI](https://python.langchain.com/v0.2/docs/integrations/text_embedding/openai/) 嵌入在 (内存中) [Chroma](/docs/integrations/providers/chroma/) 向量存储中对它们进行索引，但任何 LangChain 向量存储或嵌入模型都可以。

```python
%pip install --upgrade --quiet  langchain-chroma langchain langchain-openai > /dev/null
```

```python
from langchain.storage import InMemoryByteStore
from langchain_chroma import Chroma
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

loaders = [
    TextLoader("paul_graham_essay.txt"),
    TextLoader("state_of_the_union.txt"),
]
docs = []
for loader in loaders:
    docs.extend(loader.load())
text_splitter = RecursiveCharacterTextSplitter(chunk_size=10000)
docs = text_splitter.split_documents(docs)

# The vectorstore to use to index the child chunks
vectorstore = Chroma(
    collection_name="full_documents", embedding_function=OpenAIEmbeddings()
)
```

## 更小的块

有时候，检索较大信息块是有用的，但嵌入较小的信息块。这使得嵌入能够尽可能准确地捕捉语义含义，同时尽可能多地传递上下文。请注意，这正是[ParentDocumentRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain.retrievers.parent_document_retriever.ParentDocumentRetriever.html)所做的。这里我们展示了其背后的工作原理。

我们将区分向量存储，它索引（子）文档的嵌入，以及文档存储，它存储“父”文档并将其与标识符关联。

```python
import uuid

from langchain.retrievers.multi_vector import MultiVectorRetriever

# 存储父文档的存储层
store = InMemoryByteStore()
id_key = "doc_id"

# 检索器（开始时为空）
retriever = MultiVectorRetriever(
    vectorstore=vectorstore,
    byte_store=store,
    id_key=id_key,
)

doc_ids = [str(uuid.uuid4()) for _ in docs]
```

接下来，我们通过拆分原始文档来生成“子”文档。请注意，我们将文档标识符存储在相应的[Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html)对象的`metadata`中。

```python
# 用于创建更小块的拆分器
child_text_splitter = RecursiveCharacterTextSplitter(chunk_size=400)

sub_docs = []
for i, doc in enumerate(docs):
    _id = doc_ids[i]
    _sub_docs = child_text_splitter.split_documents([doc])
    for _doc in _sub_docs:
        _doc.metadata[id_key] = _id
    sub_docs.extend(_sub_docs)
```

最后，我们在向量存储和文档存储中索引文档：

```python
retriever.vectorstore.add_documents(sub_docs)
retriever.docstore.mset(list(zip(doc_ids, docs)))
```

仅向量存储将检索小块：

```python
retriever.vectorstore.similarity_search("justice breyer")[0]
```

```output
Document(page_content='Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court.', metadata={'doc_id': '064eca46-a4c4-4789-8e3b-583f9597e54f', 'source': 'state_of_the_union.txt'})
```

而检索器将返回较大的父文档：

```python
len(retriever.invoke("justice breyer")[0].page_content)
```

```output
9875
```

检索器在向量数据库上执行的默认搜索类型是相似性搜索。LangChain向量存储还支持通过[Max Marginal Relevance](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html#langchain_core.vectorstores.VectorStore.max_marginal_relevance_search)进行搜索。这可以通过检索器的`search_type`参数进行控制：

```python
from langchain.retrievers.multi_vector import SearchType

retriever.search_type = SearchType.mmr

len(retriever.invoke("justice breyer")[0].page_content)
```

```output
9875
```

## 将摘要与文档关联以便检索

摘要可能更准确地提炼出一个片段的内容，从而提高检索效果。这里我们展示如何创建摘要，并将其嵌入。

我们构建一个简单的 [chain](/docs/how_to/sequence)，它将接收一个输入 [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html) 对象，并使用 LLM 生成摘要。

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />


```python
import uuid

from langchain_core.documents import Document
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate

chain = (
    {"doc": lambda x: x.page_content}
    | ChatPromptTemplate.from_template("Summarize the following document:\n\n{doc}")
    | llm
    | StrOutputParser()
)
```

请注意，我们可以 [batch](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable) 该链以处理多个文档：


```python
summaries = chain.batch(docs, {"max_concurrency": 5})
```

然后我们可以像之前一样初始化一个 `MultiVectorRetriever`，在我们的向量存储中索引摘要，并在文档存储中保留原始文档：


```python
# The vectorstore to use to index the child chunks
vectorstore = Chroma(collection_name="summaries", embedding_function=OpenAIEmbeddings())
# The storage layer for the parent documents
store = InMemoryByteStore()
id_key = "doc_id"
# The retriever (empty to start)
retriever = MultiVectorRetriever(
    vectorstore=vectorstore,
    byte_store=store,
    id_key=id_key,
)
doc_ids = [str(uuid.uuid4()) for _ in docs]

summary_docs = [
    Document(page_content=s, metadata={id_key: doc_ids[i]})
    for i, s in enumerate(summaries)
]

retriever.vectorstore.add_documents(summary_docs)
retriever.docstore.mset(list(zip(doc_ids, docs)))
```


```python
# # We can also add the original chunks to the vectorstore if we so want
# for i, doc in enumerate(docs):
#     doc.metadata[id_key] = doc_ids[i]
# retriever.vectorstore.add_documents(docs)
```

查询向量存储将返回摘要：


```python
sub_docs = retriever.vectorstore.similarity_search("justice breyer")

sub_docs[0]
```



```output
Document(page_content="President Biden recently nominated Judge Ketanji Brown Jackson to serve on the United States Supreme Court, emphasizing her qualifications and broad support. The President also outlined a plan to secure the border, fix the immigration system, protect women's rights, support LGBTQ+ Americans, and advance mental health services. He highlighted the importance of bipartisan unity in passing legislation, such as the Violence Against Women Act. The President also addressed supporting veterans, particularly those impacted by exposure to burn pits, and announced plans to expand benefits for veterans with respiratory cancers. Additionally, he proposed a plan to end cancer as we know it through the Cancer Moonshot initiative. President Biden expressed optimism about the future of America and emphasized the strength of the American people in overcoming challenges.", metadata={'doc_id': '84015b1b-980e-400a-94d8-cf95d7e079bd'})
```


而检索器将返回更大的源文档：


```python
retrieved_docs = retriever.invoke("justice breyer")

len(retrieved_docs[0].page_content)
```



```output
9194
```

## 假设性查询

LLM 还可以用于生成一组假设性问题，这些问题可能与特定文档的相关查询在语义上非常相似，这在 [RAG](/docs/tutorials/rag) 应用中可能会有用。这些问题可以被嵌入并与文档关联，以改善检索。

下面，我们使用 [with_structured_output](/docs/how_to/structured_output/) 方法将 LLM 输出结构化为字符串列表。


```python
from typing import List

from langchain_core.pydantic_v1 import BaseModel, Field


class HypotheticalQuestions(BaseModel):
    """生成假设性问题。"""

    questions: List[str] = Field(..., description="问题列表")


chain = (
    {"doc": lambda x: x.page_content}
    # 仅请求 3 个假设性问题，但这可以调整
    | ChatPromptTemplate.from_template(
        "生成一组恰好 3 个假设性问题，这些问题可以用以下文档来回答：\n\n{doc}"
    )
    | ChatOpenAI(max_retries=0, model="gpt-4o").with_structured_output(
        HypotheticalQuestions
    )
    | (lambda x: x.questions)
)
```

在单个文档上调用链演示了它输出一组问题：


```python
chain.invoke(docs[0])
```



```output
["IBM 1401 对作者早期编程经验产生了什么影响？",
 "从使用 IBM 1401 过渡到微型计算机如何影响作者的编程历程？",
 "Lisp 在塑造作者对人工智能的理解和方法中扮演了什么角色？"]
```


然后我们可以对所有文档进行批处理，并像之前一样组装我们的向量存储和文档存储：


```python
# 对文档进行批处理以生成假设性问题
hypothetical_questions = chain.batch(docs, {"max_concurrency": 5})


# 用于索引子块的向量存储
vectorstore = Chroma(
    collection_name="hypo-questions", embedding_function=OpenAIEmbeddings()
)
# 存储父文档的存储层
store = InMemoryByteStore()
id_key = "doc_id"
# 检索器（开始时为空）
retriever = MultiVectorRetriever(
    vectorstore=vectorstore,
    byte_store=store,
    id_key=id_key,
)
doc_ids = [str(uuid.uuid4()) for _ in docs]


# 从假设性问题生成文档对象
question_docs = []
for i, question_list in enumerate(hypothetical_questions):
    question_docs.extend(
        [Document(page_content=s, metadata={id_key: doc_ids[i]}) for s in question_list]
    )


retriever.vectorstore.add_documents(question_docs)
retriever.docstore.mset(list(zip(doc_ids, docs)))
```

请注意，查询底层向量存储将检索出与输入查询在语义上相似的假设性问题：


```python
sub_docs = retriever.vectorstore.similarity_search("justice breyer")

sub_docs
```



```output
[Document(page_content='提名巡回上诉法庭法官 Ketanji Brown Jackson 到美国最高法院可能带来什么潜在好处？', metadata={'doc_id': '43292b74-d1b8-4200-8a8b-ea0cb57fbcdb'}),
 Document(page_content='两党基础设施法将如何影响美国与中国之间的经济竞争？', metadata={'doc_id': '66174780-d00c-4166-9791-f0069846e734'}),
 Document(page_content='是什么因素导致了 Y Combinator 的创建？', metadata={'doc_id': '72003c4e-4cc9-4f09-a787-0b541a65b38c'}),
 Document(page_content='在线发表文章的能力如何改变了作家和思想家的格局？', metadata={'doc_id': 'e8d2c648-f245-4bcc-b8d3-14e64a164b64'})]
```


调用检索器将返回相应的文档：


```python
retrieved_docs = retriever.invoke("justice breyer")
len(retrieved_docs[0].page_content)
```



```output
9194
```
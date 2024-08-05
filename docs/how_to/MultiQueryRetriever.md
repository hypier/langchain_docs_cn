---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/MultiQueryRetriever.ipynb
---

# 如何使用 MultiQueryRetriever

基于距离的向量数据库检索在高维空间中嵌入（表示）查询，并根据距离度量找到相似的嵌入文档。但是，检索可能会因查询措辞的细微变化而产生不同的结果，或者如果嵌入未能很好地捕捉数据的语义。提示工程/调优有时会手动解决这些问题，但可能会很繁琐。

[MultiQueryRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain.retrievers.multi_query.MultiQueryRetriever.html) 通过使用 LLM 从给定用户输入查询生成多个不同视角的查询，自动化提示调优的过程。对于每个查询，它检索一组相关文档，并在所有查询中取唯一的并集，以获取一组更大可能相关的文档。通过对同一问题生成多个视角，`MultiQueryRetriever` 可以减轻基于距离的检索的一些限制，并获得更丰富的结果。

让我们使用 Lilian Weng 的 [RAG 教程](/docs/tutorials/rag) 中的 [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) 博客文章构建一个向量存储：

```python
# Build a sample vectorDB
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

# Load blog post
loader = WebBaseLoader("https://lilianweng.github.io/posts/2023-06-23-agent/")
data = loader.load()

# Split
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
splits = text_splitter.split_documents(data)

# VectorDB
embedding = OpenAIEmbeddings()
vectordb = Chroma.from_documents(documents=splits, embedding=embedding)
```

#### 简单使用

指定用于查询生成的 LLM，检索器将处理其余部分。

```python
from langchain.retrievers.multi_query import MultiQueryRetriever
from langchain_openai import ChatOpenAI

question = "What are the approaches to Task Decomposition?"
llm = ChatOpenAI(temperature=0)
retriever_from_llm = MultiQueryRetriever.from_llm(
    retriever=vectordb.as_retriever(), llm=llm
)
```

```python
# Set logging for the queries
import logging

logging.basicConfig()
logging.getLogger("langchain.retrievers.multi_query").setLevel(logging.INFO)
```

```python
unique_docs = retriever_from_llm.invoke(question)
len(unique_docs)
```
```output
INFO:langchain.retrievers.multi_query:Generated queries: ['1. How can Task Decomposition be achieved through different methods?', '2. What strategies are commonly used for Task Decomposition?', '3. What are the various techniques for breaking down tasks in Task Decomposition?']
```

```output
5
```

请注意，检索器生成的底层查询以 `INFO` 级别记录。

#### 提供您自己的提示

在后台，`MultiQueryRetriever` 使用特定的 [prompt](https://api.python.langchain.com/en/latest/_modules/langchain/retrievers/multi_query.html#MultiQueryRetriever) 生成查询。要自定义此提示：

1. 创建一个带有问题输入变量的 [PromptTemplate](https://api.python.langchain.com/en/latest/prompts/langchain_core.prompts.prompt.PromptTemplate.html)；
2. 实现一个 [output parser](/docs/concepts#output-parsers)，如下所示，将结果拆分为查询列表。

提示和输出解析器必须共同支持生成查询列表。

```python
from typing import List

from langchain_core.output_parsers import BaseOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field

# Output parser will split the LLM result into a list of queries
class LineListOutputParser(BaseOutputParser[List[str]]):
    """Output parser for a list of lines."""

    def parse(self, text: str) -> List[str]:
        lines = text.strip().split("\n")
        return list(filter(None, lines))  # Remove empty lines

output_parser = LineListOutputParser()

QUERY_PROMPT = PromptTemplate(
    input_variables=["question"],
    template="""You are an AI language model assistant. Your task is to generate five 
    different versions of the given user question to retrieve relevant documents from a vector 
    database. By generating multiple perspectives on the user question, your goal is to help
    the user overcome some of the limitations of the distance-based similarity search. 
    Provide these alternative questions separated by newlines.
    Original question: {question}""",
)
llm = ChatOpenAI(temperature=0)

# Chain
llm_chain = QUERY_PROMPT | llm | output_parser

# Other inputs
question = "What are the approaches to Task Decomposition?"
```

```python
# Run
retriever = MultiQueryRetriever(
    retriever=vectordb.as_retriever(), llm_chain=llm_chain, parser_key="lines"
)  # "lines" is the key (attribute name) of the parsed output

# Results
unique_docs = retriever.invoke("What does the course say about regression?")
len(unique_docs)
```
```output
INFO:langchain.retrievers.multi_query:Generated queries: ['1. Can you provide insights on regression from the course material?', '2. How is regression discussed in the course content?', '3. What information does the course offer about regression?', '4. In what way is regression covered in the course?', '5. What are the teachings of the course regarding regression?']
```

```output
9
```
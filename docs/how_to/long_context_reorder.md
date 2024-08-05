---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/long_context_reorder.ipynb
---

# 如何重新排序检索结果以减轻“中间丢失”效应

在[RAG](/docs/tutorials/rag)应用中，随着检索文档数量的增加（例如，超过十个），性能显著下降的情况已被[记录](https://arxiv.org/abs/2307.03172)。简而言之：模型容易在长上下文中遗漏相关信息。

相比之下，对向量存储的查询通常会按相关性降序返回文档（例如，通过[嵌入](/docs/concepts/#embedding-models)的余弦相似度进行测量）。

为减轻["中间丢失"](https://arxiv.org/abs/2307.03172)效应，您可以在检索后重新排序文档，使得最相关的文档位于极端位置（例如，上下文的首尾），而最不相关的文档则位于中间。在某些情况下，这可以帮助LLM更好地呈现最相关的信息。

[LongContextReorder](https://api.python.langchain.com/en/latest/document_transformers/langchain_community.document_transformers.long_context_reorder.LongContextReorder.html)文档转换器实现了这一重新排序过程。下面我们展示一个示例。


```python
%pip install --upgrade --quiet  sentence-transformers langchain-chroma langchain langchain-openai langchain-huggingface > /dev/null
```

首先，我们嵌入一些人工文档并将它们索引到一个（内存中的）[Chroma](/docs/integrations/providers/chroma/)向量存储中。我们将使用[Hugging Face](/docs/integrations/text_embedding/huggingfacehub/)嵌入，但任何LangChain向量存储或嵌入模型都可以使用。


```python
from langchain_chroma import Chroma
from langchain_huggingface import HuggingFaceEmbeddings

# 获取嵌入。
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")

texts = [
    "篮球是一项伟大的运动。",
    "飞我去月球是我最喜欢的歌曲之一。",
    "凯尔特人是我最喜欢的球队。",
    "这是关于波士顿凯尔特人的文档",
    "我非常喜欢去电影院",
    "波士顿凯尔特人以20分赢得了比赛",
    "这只是一个随机文本。",
    "艾尔登法环是过去15年中最好的游戏之一。",
    "L. 科尔内特是凯尔特人队最优秀的球员之一。",
    "拉里·伯德是一位标志性的NBA球员。",
]

# 创建检索器
retriever = Chroma.from_texts(texts, embedding=embeddings).as_retriever(
    search_kwargs={"k": 10}
)
query = "你能告诉我关于凯尔特人的事情吗？"

# 获取按相关性评分排序的相关文档
docs = retriever.invoke(query)
docs
```



```output
[Document(page_content='这是关于波士顿凯尔特人的文档'),
 Document(page_content='凯尔特人是我最喜欢的球队。'),
 Document(page_content='L. 科尔内特是凯尔特人队最优秀的球员之一。'),
 Document(page_content='波士顿凯尔特人以20分赢得了比赛'),
 Document(page_content='拉里·伯德是一位标志性的NBA球员。'),
 Document(page_content='艾尔登法环是过去15年中最好的游戏之一。'),
 Document(page_content='篮球是一项伟大的运动。'),
 Document(page_content='我非常喜欢去电影院'),
 Document(page_content='飞我去月球是我最喜欢的歌曲之一。'),
 Document(page_content='这只是一个随机文本。')]
```


注意，文档是按与查询的相关性降序返回的。`LongContextReorder`文档转换器将实现上述重新排序：


```python
from langchain_community.document_transformers import LongContextReorder

# 重新排序文档：
# 不太相关的文档将位于列表的中间，而更相关的元素位于开头/结尾。
reordering = LongContextReorder()
reordered_docs = reordering.transform_documents(docs)

# 确认4个相关文档位于开头和结尾。
reordered_docs
```



```output
[Document(page_content='凯尔特人是我最喜欢的球队。'),
 Document(page_content='波士顿凯尔特人以20分赢得了比赛'),
 Document(page_content='艾尔登法环是过去15年中最好的游戏之一。'),
 Document(page_content='我非常喜欢去电影院'),
 Document(page_content='这只是一个随机文本。'),
 Document(page_content='飞我去月球是我最喜欢的歌曲之一。'),
 Document(page_content='篮球是一项伟大的运动。'),
 Document(page_content='拉里·伯德是一位标志性的NBA球员。'),
 Document(page_content='L. 科尔内特是凯尔特人队最优秀的球员之一。'),
 Document(page_content='这是关于波士顿凯尔特人的文档')]
```


下面，我们展示如何将重新排序的文档纳入一个简单的问题回答链：


```python
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI

llm = OpenAI()

prompt_template = """
给定这些文本：
-----
{context}
-----
请回答以下问题：
{query}
"""

prompt = PromptTemplate(
    template=prompt_template,
    input_variables=["context", "query"],
)

# 创建并调用链：
chain = create_stuff_documents_chain(llm, prompt)
response = chain.invoke({"context": reordered_docs, "query": query})
print(response)
```
```output

凯尔特人是一支职业篮球队，是NBA中最具标志性的特许经营之一。他们备受推崇，并拥有庞大的粉丝基础。球队有过许多成功的赛季，常常被认为是联盟中最顶尖的球队之一。他们有着辉煌的历史，培养了许多优秀的球员，如拉里·伯德和L. 科尔内特。球队位于波士顿，通常被称为波士顿凯尔特人。
```
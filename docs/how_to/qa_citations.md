---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/qa_citations.ipynb
---

# 如何让RAG应用程序添加引用

本指南回顾了获取模型引用其在生成响应时参考的源文档的哪些部分的方法。

我们将介绍五种方法：

1. 使用工具调用引用文档ID；
2. 使用工具调用引用文档ID并提供文本片段；
3. 直接提示；
4. 检索后处理（即压缩检索到的上下文以使其更相关）；
5. 生成后处理（即发出第二个LLM调用，以引用注释生成的答案）。

我们通常建议使用列表中适合您用例的第一项。也就是说，如果您的模型支持工具调用，请尝试方法1或2；否则，如果这些方法失败，请按顺序继续。

让我们首先创建一个简单的RAG链。首先，我们将使用[WikipediaRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.wikipedia.WikipediaRetriever.html)从维基百科进行检索。

## 设置

首先，我们需要安装一些依赖项并设置我们将使用的模型的环境变量。

```python
%pip install -qU langchain langchain-openai langchain-anthropic langchain-community wikipedia
```

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()
os.environ["ANTHROPIC_API_KEY"] = getpass.getpass()

# 如果您想记录到 LangSmith，请取消注释
# os.environ["LANGCHAIN_TRACING_V2"] = "true
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

让我们首先选择一个 LLM：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />

```python
from langchain_community.retrievers import WikipediaRetriever
from langchain_core.prompts import ChatPromptTemplate

system_prompt = (
    "You're a helpful AI assistant. Given a user question "
    "and some Wikipedia article snippets, answer the user "
    "question. If none of the articles answer the question, "
    "just say you don't know."
    "\n\nHere are the Wikipedia articles: "
    "{context}"
)

retriever = WikipediaRetriever(top_k_results=6, doc_content_chars_max=2000)
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human", "{input}"),
    ]
)
prompt.pretty_print()
```
```output
================================[1m 系统消息 [0m================================

You're a helpful AI assistant. Given a user question and some Wikipedia article snippets, answer the user question. If none of the articles answer the question, just say you don't know.

Here are the Wikipedia articles: [33;1m[1;3m{context}[0m

================================[1m 人类消息 [0m=================================

[33;1m[1;3m{input}[0m
```
现在我们已经有了模型、检索器和提示，让我们将它们串联在一起。我们需要添加一些逻辑，将检索到的文档格式化为可以传递给我们的提示的字符串。按照 [为 RAG 应用添加引用](/docs/how_to/qa_citations) 的指南，我们将使我们的链返回答案和检索到的文档。

```python
from typing import List

from langchain_core.documents import Document
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough


def format_docs(docs: List[Document]):
    return "\n\n".join(doc.page_content for doc in docs)


rag_chain_from_docs = (
    RunnablePassthrough.assign(context=(lambda x: format_docs(x["context"])))
    | prompt
    | llm
    | StrOutputParser()
)

retrieve_docs = (lambda x: x["input"]) | retriever

chain = RunnablePassthrough.assign(context=retrieve_docs).assign(
    answer=rag_chain_from_docs
)
```

```python
result = chain.invoke({"input": "How fast are cheetahs?"})
```

```python
print(result.keys())
```
```output
dict_keys(['input', 'context', 'answer'])
```

```python
print(result["context"][0])
```
```output
page_content='The cheetah (Acinonyx jubatus) is a large cat and the fastest land animal. It has a tawny to creamy white or pale buff fur that is marked with evenly spaced, solid black spots. The head is small and rounded, with a short snout and black tear-like facial streaks. It reaches 67–94 cm (26–37 in) at the shoulder, and the head-and-body length is between 1.1 and 1.5 m (3 ft 7 in and 4 ft 11 in). Adults weigh between 21 and 72 kg (46 and 159 lb). The cheetah is capable of running at 93 to 104 km/h (58 to 65 mph); it has evolved specialized adaptations for speed, including a light build, long thin legs and a long tail.\nThe cheetah was first described in the late 18th century. Four subspecies are recognised today that are native to Africa and central Iran. An African subspecies was introduced to India in 2022. It is now distributed mainly in small, fragmented populations in northwestern, eastern and southern Africa and central Iran. It lives in a variety of habitats such as savannahs in the Serengeti, arid mountain ranges in the Sahara, and hilly desert terrain.\nThe cheetah lives in three main social groups: females and their cubs, male "coalitions", and solitary males. While females lead a nomadic life searching for prey in large home ranges, males are more sedentary and instead establish much smaller territories in areas with plentiful prey and access to females. The cheetah is active during the day, with peaks during dawn and dusk. It feeds on small- to medium-sized prey, mostly weighing under 40 kg (88 lb), and prefers medium-sized ungulates such as impala, springbok and Thomson\'s gazelles. The cheetah typically stalks its prey within 60–100 m (200–330 ft) before charging towards it, trips it during the chase and bites its throat to suffocate it to death. It breeds throughout the year. After a gestation of nearly three months, females give birth to a litter of three or four cubs. Cheetah cubs are highly vulnerable to predation by other large carnivores. They are weaned a' metadata={'title': 'Cheetah', 'summary': 'The cheetah (Acinonyx jubatus) is a large cat and the fastest land animal. It has a tawny to creamy white or pale buff fur that is marked with evenly spaced, solid black spots. The head is small and rounded, with a short snout and black tear-like facial streaks. It reaches 67–94 cm (26–37 in) at the shoulder, and the head-and-body length is between 1.1 and 1.5 m (3 ft 7 in and 4 ft 11 in). Adults weigh between 21 and 72 kg (46 and 159 lb). The cheetah is capable of running at 93 to 104 km/h (58 to 65 mph); it has evolved specialized adaptations for speed, including a light build, long thin legs and a long tail.\nThe cheetah was first described in the late 18th century. Four subspecies are recognised today that are native to Africa and central Iran. An African subspecies was introduced to India in 2022. It is now distributed mainly in small, fragmented populations in northwestern, eastern and southern Africa and central Iran. It lives in a variety of habitats such as savannahs in the Serengeti, arid mountain ranges in the Sahara, and hilly desert terrain.\nThe cheetah lives in three main social groups: females and their cubs, male "coalitions", and solitary males. While females lead a nomadic life searching for prey in large home ranges, males are more sedentary and instead establish much smaller territories in areas with plentiful prey and access to females. The cheetah is active during the day, with peaks during dawn and dusk. It feeds on small- to medium-sized prey, mostly weighing under 40 kg (88 lb), and prefers medium-sized ungulates such as impala, springbok and Thomson\'s gazelles. The cheetah typically stalks its prey within 60–100 m (200–330 ft) before charging towards it, trips it during the chase and bites its throat to suffocate it to death. It breeds throughout the year. After a gestation of nearly three months, females give birth to a litter of three or four cubs. Cheetah cubs are highly vulnerable to predation by other large carnivores. They are weaned at around four months and are independent by around 20 months of age.\nThe cheetah is threatened by habitat loss, conflict with humans, poaching and high susceptibility to diseases. In 2016, the global cheetah population was estimated at 7,100 individuals in the wild; it is listed as Vulnerable on the IUCN Red List. It has been widely depicted in art, literature, advertising, and animation. It was tamed in ancient Egypt and trained for hunting ungulates in the Arabian Peninsula and India. It has been kept in zoos since the early 19th century.', 'source': 'https://en.wikipedia.org/wiki/Cheetah'}
```

```python
print(result["answer"])
```
```output
Cheetahs are capable of running at speeds of 93 to 104 km/h (58 to 65 mph). They have evolved specialized adaptations for speed, including a light build, long thin legs, and a long tail.
```
LangSmith trace: https://smith.langchain.com/public/0472c5d1-49dc-4c1c-8100-61910067d7ed/r

## 函数调用

如果您选择的 LLM 实现了 [工具调用](/docs/concepts#functiontool-calling) 功能，您可以使用它让模型在生成答案时指定引用的文档。LangChain 工具调用模型实现了一个 `.with_structured_output` 方法，该方法将强制生成遵循所需模式的输出（例如，参见 [这里](https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.base.ChatOpenAI.html#langchain_openai.chat_models.base.ChatOpenAI.with_structured_output)）。

### 引用文档

要使用标识符引用文档，我们将标识符格式化到提示中，然后使用 `.with_structured_output` 强制 LLM 在其输出中引用这些标识符。

首先，我们为输出定义一个模式。`.with_structured_output` 支持多种格式，包括 JSON schema 和 Pydantic。这里我们将使用 Pydantic：

```python
from langchain_core.pydantic_v1 import BaseModel, Field


class CitedAnswer(BaseModel):
    """仅根据给定来源回答用户问题，并引用所用的来源。"""

    answer: str = Field(
        ...,
        description="基于给定来源的用户问题的答案。",
    )
    citations: List[int] = Field(
        ...,
        description="证明答案的特定来源的整数 ID。",
    )
```

让我们看看当我们传入我们的函数和用户输入时，模型输出是什么样的：

```python
structured_llm = llm.with_structured_output(CitedAnswer)

example_q = """布莱恩的身高是多少？

来源：1
信息：苏西的身高是6'2"

来源：2
信息：杰里迈亚是金发

来源：3
信息：布莱恩比苏西矮3英寸"""
result = structured_llm.invoke(example_q)

result
```

```output
CitedAnswer(answer='布莱恩的身高是5\'11".', citations=[1, 3])
```

或者作为字典：

```python
result.dict()
```

```output
{'answer': '布莱恩的身高是5\'11".', 'citations': [1, 3]}
```

现在我们将来源标识符结构化到提示中，以便与我们的链复制。我们将进行三项更改：

1. 更新提示以包括来源标识符；
2. 使用 `structured_llm`（即 `llm.with_structured_output(CitedAnswer)`）；
3. 移除 `StrOutputParser`，以在输出中保留 Pydantic 对象。

```python
def format_docs_with_id(docs: List[Document]) -> str:
    formatted = [
        f"来源 ID: {i}\n文章标题: {doc.metadata['title']}\n文章摘录: {doc.page_content}"
        for i, doc in enumerate(docs)
    ]
    return "\n\n" + "\n\n".join(formatted)


rag_chain_from_docs = (
    RunnablePassthrough.assign(context=(lambda x: format_docs_with_id(x["context"])))
    | prompt
    | structured_llm
)

retrieve_docs = (lambda x: x["input"]) | retriever

chain = RunnablePassthrough.assign(context=retrieve_docs).assign(
    answer=rag_chain_from_docs
)
```

```python
result = chain.invoke({"input": "猎豹的速度有多快？"})
```

```python
print(result["answer"])
```
```output
answer='猎豹的速度可以达到93到104公里每小时（58到65英里每小时）。它们被认为是最快的陆地动物。' citations=[0]
```
我们可以检查模型引用的索引 0 的文档：

```python
print(result["context"][0])
```
```output
page_content='猎豹（Acinonyx jubatus）是一种大型猫科动物，也是最快的陆地动物。它的毛色为黄褐色至奶油白或淡棕色，身上有均匀分布的黑色实心斑点。头部小而圆，鼻子短，脸部有黑色泪痕状的条纹。它的肩高为67–94厘米（26–37英寸），头身长在1.1米到1.5米（3英尺7英寸到4英尺11英寸）之间。成年猎豹的体重在21到72公斤（46到159磅）之间。猎豹能够以93到104公里每小时（58到65英里每小时）的速度奔跑；它已经进化出适应速度的特殊适应性，包括轻巧的体型、细长的腿和长尾巴。\n猎豹在18世纪末首次被描述。今天公认的四个亚种原产于非洲和中伊朗。一个非洲亚种于2022年被引入印度。现在它主要分布在西北、东部和南部非洲以及中伊朗的小型、支离破碎的种群中。它生活在各种栖息地中，如塞伦盖蒂的草原、撒哈拉的干燥山脉和丘陵沙漠地形。\n猎豹生活在三种主要的社会群体中：雌性及其幼崽、雄性“联盟”和独居雄性。雌性过着游牧生活，寻找猎物，拥有较大的生活范围，而雄性则更为定居，通常在猎物丰富且可接触雌性的地区建立较小的领地。猎豹在白天活动，黎明和黄昏时最为活跃。它以小型到中型猎物为食，主要重量在40公斤（88磅）以下，偏好中型有蹄类动物，如瞪羚、春羚和汤姆逊瞪羚。猎豹通常在60–100米（200–330英尺）内潜行猎物，然后向其冲刺，在追逐过程中绊倒猎物并咬住其喉咙使其窒息而死。它全年繁殖。经过近三个月的妊娠，雌性会产下三到四只幼崽。猎豹幼崽对其他大型食肉动物的捕食高度脆弱。它们在大约四个月时断奶，并在大约20个月大时独立。\n猎豹面临栖息地丧失、与人类冲突、偷猎和对疾病的高度易感等威胁。2016年，全球猎豹种群估计在野外有7100只；在IUCN红色名录中被列为脆弱。它在艺术、文学、广告和动画中被广泛描绘。它在古埃及时被驯化，并在阿拉伯半岛和印度被训练用于猎捕有蹄类动物。自19世纪初以来，它一直被饲养在动物园中。' metadata={'title': '猎豹', 'summary': '猎豹（Acinonyx jubatus）是一种大型猫科动物，也是最快的陆地动物。它的毛色为黄褐色至奶油白或淡棕色，身上有均匀分布的黑色实心斑点。头部小而圆，鼻子短，脸部有黑色泪痕状的条纹。它的肩高为67–94厘米（26–37英寸），头身长在1.1米到1.5米（3英尺7英寸到4英尺11英寸）之间。成年猎豹的体重在21到72公斤（46到159磅）之间。猎豹能够以93到104公里每小时（58到65英里每小时）的速度奔跑；它已经进化出适应速度的特殊适应性，包括轻巧的体型、细长的腿和长尾巴。\n猎豹在18世纪末首次被描述。今天公认的四个亚种原产于非洲和中伊朗。一个非洲亚种于2022年被引入印度。现在它主要分布在西北、东部和南部非洲以及中伊朗的小型、支离破碎的种群中。它生活在各种栖息地中，如塞伦盖蒂的草原、撒哈拉的干燥山脉和丘陵沙漠地形。\n猎豹生活在三种主要的社会群体中：雌性及其幼崽、雄性“联盟”和独居雄性。雌性过着游牧生活，寻找猎物，拥有较大的生活范围，而雄性则更为定居，通常在猎物丰富且可接触雌性的地区建立较小的领地。猎豹在白天活动，黎明和黄昏时最为活跃。它以小型到中型猎物为食，主要重量在40公斤（88磅）以下，偏好中型有蹄类动物，如瞪羚、春羚和汤姆逊瞪羚。猎豹通常在60–100米（200–330英尺）内潜行猎物，然后向其冲刺，在追逐过程中绊倒猎物并咬住其喉咙使其窒息而死。它全年繁殖。经过近三个月的妊娠，雌性会产下三到四只幼崽。猎豹幼崽对其他大型食肉动物的捕食高度脆弱。它们在大约四个月时断奶，并在大约20个月大时独立。\n猎豹面临栖息地丧失、与人类冲突、偷猎和对疾病的高度易感等威胁。2016年，全球猎豹种群估计在野外有7100只；在IUCN红色名录中被列为脆弱。它在艺术、文学、广告和动画中被广泛描绘。它在古埃及时被驯化，并在阿拉伯半岛和印度被训练用于猎捕有蹄类动物。自19世纪初以来，它一直被饲养在动物园中。', 'source': 'https://en.wikipedia.org/wiki/Cheetah'}
```

### 引用片段

为了返回文本片段（可能还包括源标识符），我们可以使用相同的方法。唯一的变化是构建一个更复杂的输出模式，这里使用 Pydantic，包括一个“引用”和一个源标识符。

*附注：请注意，如果我们将文档拆分成许多只有一两句话的文档，而不是几个较长的文档，引用文档大致等同于引用片段，并且可能对模型来说更容易，因为模型只需要返回每个片段的标识符，而不是实际文本。可能值得尝试这两种方法并进行评估。*

```python
class Citation(BaseModel):
    source_id: int = Field(
        ...,
        description="特定来源的整数 ID，用于证明答案的合理性。",
    )
    quote: str = Field(
        ...,
        description="来自指定来源的逐字引用，用于证明答案的合理性。",
    )


class QuotedAnswer(BaseModel):
    """仅基于给定来源回答用户问题，并引用所用来源。"""

    answer: str = Field(
        ...,
        description="基于给定来源的用户问题的答案。",
    )
    citations: List[Citation] = Field(
        ..., description="来自给定来源的引用，用于证明答案的合理性。"
    )
```

```python
rag_chain_from_docs = (
    RunnablePassthrough.assign(context=(lambda x: format_docs_with_id(x["context"])))
    | prompt
    | llm.with_structured_output(QuotedAnswer)
)

retrieve_docs = (lambda x: x["input"]) | retriever

chain = RunnablePassthrough.assign(context=retrieve_docs).assign(
    answer=rag_chain_from_docs
)
```

```python
result = chain.invoke({"input": "猎豹的速度有多快？"})
```

在这里我们看到模型从源 0 中提取了相关的文本片段：

```python
result["answer"]
```

```output
QuotedAnswer(answer='猎豹的速度可达到 93 到 104 公里/小时（58 到 65 英里/小时）。', citations=[Citation(source_id=0, quote='猎豹能够以 93 到 104 公里/小时（58 到 65 英里/小时）的速度奔跑；它已进化出专门的速度适应性，包括轻盈的体型、修长的腿和长尾巴。')])
```

LangSmith 跟踪： https://smith.langchain.com/public/0f638cc9-8409-4a53-9010-86ac28144129/r

## 直接提示

许多模型不支持函数调用。我们可以通过直接提示实现类似的结果。让我们尝试指示模型生成结构化的 XML 作为输出：


```python
xml_system = """You're a helpful AI assistant. Given a user question and some Wikipedia article snippets, \
answer the user question and provide citations. If none of the articles answer the question, just say you don't know.

Remember, you must return both an answer and citations. A citation consists of a VERBATIM quote that \
justifies the answer and the ID of the quote article. Return a citation for every quote across all articles \
that justify the answer. Use the following format for your final output:

<cited_answer>
    <answer></answer>
    <citations>
        <citation><source_id></source_id><quote></quote></citation>
        <citation><source_id></source_id><quote></quote></citation>
        ...
    </citations>
</cited_answer>

Here are the Wikipedia articles:{context}"""
xml_prompt = ChatPromptTemplate.from_messages(
    [("system", xml_system), ("human", "{input}")]
)
```

我们现在对我们的链进行类似的小更新：

1. 我们更新格式化函数，以将检索到的上下文包装在 XML 标签中；
2. 我们不使用 `.with_structured_output`（例如，因为它在某个模型中不存在）；
3. 我们使用 [XMLOutputParser](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.xml.XMLOutputParser.html) 代替 `StrOutputParser` 来将答案解析为字典。


```python
from langchain_core.output_parsers import XMLOutputParser


def format_docs_xml(docs: List[Document]) -> str:
    formatted = []
    for i, doc in enumerate(docs):
        doc_str = f"""\
    <source id=\"{i}\">
        <title>{doc.metadata['title']}</title>
        <article_snippet>{doc.page_content}</article_snippet>
    </source>"""
        formatted.append(doc_str)
    return "\n\n<sources>" + "\n".join(formatted) + "</sources>"


rag_chain_from_docs = (
    RunnablePassthrough.assign(context=(lambda x: format_docs_xml(x["context"])))
    | xml_prompt
    | llm
    | XMLOutputParser()
)

retrieve_docs = (lambda x: x["input"]) | retriever

chain = RunnablePassthrough.assign(context=retrieve_docs).assign(
    answer=rag_chain_from_docs
)
```


```python
result = chain.invoke({"input": "How fast are cheetahs?"})
```

请注意，引用再次被结构化到答案中：


```python
result["answer"]
```



```output
{'cited_answer': [{'answer': 'Cheetahs are capable of running at 93 to 104 km/h (58 to 65 mph).'},
  {'citations': [{'citation': [{'source_id': '0'},
      {'quote': 'The cheetah is capable of running at 93 to 104 km/h (58 to 65 mph); it has evolved specialized adaptations for speed, including a light build, long thin legs and a long tail.'}]}]}]}
```


LangSmith 跟踪： https://smith.langchain.com/public/a3636c70-39c6-4c8f-bc83-1c7a174c237e/r

## 检索后处理

另一种方法是对我们检索到的文档进行后处理，以压缩内容，使得源内容已经足够简洁，以至于我们不需要模型引用特定的来源或片段。例如，我们可以将每个文档拆分成一两句话，嵌入这些句子并仅保留最相关的句子。LangChain提供了一些内置组件用于此。这里我们将使用一个 [RecursiveCharacterTextSplitter](https://api.python.langchain.com/en/latest/text_splitter/langchain_text_splitters.RecursiveCharacterTextSplitter.html#langchain_text_splitters.RecursiveCharacterTextSplitter)，它通过在分隔子字符串上进行拆分来创建指定大小的块，以及一个 [EmbeddingsFilter](https://api.python.langchain.com/en/latest/retrievers/langchain.retrievers.document_compressors.embeddings_filter.EmbeddingsFilter.html#langchain.retrievers.document_compressors.embeddings_filter.EmbeddingsFilter)，它仅保留具有最相关嵌入的文本。

这种方法有效地用一个更新的检索器替换了我们原来的检索器，该检索器压缩了文档。首先，我们构建检索器：

```python
from langchain.retrievers.document_compressors import EmbeddingsFilter
from langchain_core.runnables import RunnableParallel
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=400,
    chunk_overlap=0,
    separators=["\n\n", "\n", ".", " "],
    keep_separator=False,
)
compressor = EmbeddingsFilter(embeddings=OpenAIEmbeddings(), k=10)


def split_and_filter(input) -> List[Document]:
    docs = input["docs"]
    question = input["question"]
    split_docs = splitter.split_documents(docs)
    stateful_docs = compressor.compress_documents(split_docs, question)
    return [stateful_doc for stateful_doc in stateful_docs]


new_retriever = (
    RunnableParallel(question=RunnablePassthrough(), docs=retriever) | split_and_filter
)
docs = new_retriever.invoke("How fast are cheetahs?")
for doc in docs:
    print(doc.page_content)
    print("\n\n")
```
```output
Adults weigh between 21 and 72 kg (46 and 159 lb). The cheetah is capable of running at 93 to 104 km/h (58 to 65 mph); it has evolved specialized adaptations for speed, including a light build, long thin legs and a long tail



The cheetah (Acinonyx jubatus) is a large cat and the fastest land animal. It has a tawny to creamy white or pale buff fur that is marked with evenly spaced, solid black spots. The head is small and rounded, with a short snout and black tear-like facial streaks. It reaches 67–94 cm (26–37 in) at the shoulder, and the head-and-body length is between 1.1 and 1.5 m (3 ft 7 in and 4 ft 11 in)



2 mph), or 171 body lengths per second. The cheetah, the fastest land mammal, scores at only 16 body lengths per second, while Anna's hummingbird has the highest known length-specific velocity attained by any vertebrate



It feeds on small- to medium-sized prey, mostly weighing under 40 kg (88 lb), and prefers medium-sized ungulates such as impala, springbok and Thomson's gazelles. The cheetah typically stalks its prey within 60–100 m (200–330 ft) before charging towards it, trips it during the chase and bites its throat to suffocate it to death. It breeds throughout the year



The cheetah was first described in the late 18th century. Four subspecies are recognised today that are native to Africa and central Iran. An African subspecies was introduced to India in 2022. It is now distributed mainly in small, fragmented populations in northwestern, eastern and southern Africa and central Iran



The cheetah lives in three main social groups: females and their cubs, male "coalitions", and solitary males. While females lead a nomadic life searching for prey in large home ranges, males are more sedentary and instead establish much smaller territories in areas with plentiful prey and access to females. The cheetah is active during the day, with peaks during dawn and dusk



The Southeast African cheetah (Acinonyx jubatus jubatus) is the nominate cheetah subspecies native to East and Southern Africa. The Southern African cheetah lives mainly in the lowland areas and deserts of the Kalahari, the savannahs of Okavango Delta, and the grasslands of the Transvaal region in South Africa. In Namibia, cheetahs are mostly found in farmlands



Subpopulations have been called "South African cheetah" and "Namibian cheetah."



In India, four cheetahs of the subspecies are living in Kuno National Park in Madhya Pradesh after having been introduced there



Acinonyx jubatus velox proposed in 1913 by Edmund Heller on basis of a cheetah that was shot by Kermit Roosevelt in June 1909 in the Kenyan highlands.
Acinonyx rex proposed in 1927 by Reginald Innes Pocock on basis of a specimen from the Umvukwe Range in Rhodesia.
```
接下来，我们将其组装成我们的链，如之前所述：

```python
rag_chain_from_docs = (
    RunnablePassthrough.assign(context=(lambda x: format_docs(x["context"])))
    | prompt
    | llm
    | StrOutputParser()
)

chain = RunnablePassthrough.assign(
    context=(lambda x: x["input"]) | new_retriever
).assign(answer=rag_chain_from_docs)
```


```python
result = chain.invoke({"input": "How fast are cheetahs?"})

print(result["answer"])
```
```output
Cheetahs are capable of running at speeds between 93 to 104 km/h (58 to 65 mph), making them the fastest land animals.
```
请注意，文档内容现在已被压缩，尽管文档对象在其元数据中的“summary”键中保留了原始内容。这些摘要不会传递给模型；只有压缩的内容会传递。

```python
result["context"][0].page_content  # passed to model
```



```output
'Adults weigh between 21 and 72 kg (46 and 159 lb). The cheetah is capable of running at 93 to 104 km/h (58 to 65 mph); it has evolved specialized adaptations for speed, including a light build, long thin legs and a long tail'
```



```python
result["context"][0].metadata["summary"]  # original document
```



```output
'The cheetah (Acinonyx jubatus) is a large cat and the fastest land animal. It has a tawny to creamy white or pale buff fur that is marked with evenly spaced, solid black spots. The head is small and rounded, with a short snout and black tear-like facial streaks. It reaches 67–94 cm (26–37 in) at the shoulder, and the head-and-body length is between 1.1 and 1.5 m (3 ft 7 in and 4 ft 11 in). Adults weigh between 21 and 72 kg (46 and 159 lb). The cheetah is capable of running at 93 to 104 km/h (58 to 65 mph); it has evolved specialized adaptations for speed, including a light build, long thin legs and a long tail.\nThe cheetah was first described in the late 18th century. Four subspecies are recognised today that are native to Africa and central Iran. An African subspecies was introduced to India in 2022. It is now distributed mainly in small, fragmented populations in northwestern, eastern and southern Africa and central Iran. It lives in a variety of habitats such as savannahs in the Serengeti, arid mountain ranges in the Sahara, and hilly desert terrain.\nThe cheetah lives in three main social groups: females and their cubs, male "coalitions", and solitary males. While females lead a nomadic life searching for prey in large home ranges, males are more sedentary and instead establish much smaller territories in areas with plentiful prey and access to females. The cheetah is active during the day, with peaks during dawn and dusk. It feeds on small- to medium-sized prey, mostly weighing under 40 kg (88 lb), and prefers medium-sized ungulates such as impala, springbok and Thomson\'s gazelles. The cheetah typically stalks its prey within 60–100 m (200–330 ft) before charging towards it, trips it during the chase and bites its throat to suffocate it to death. It breeds throughout the year. After a gestation of nearly three months, females give birth to a litter of three or four cubs. Cheetah cubs are highly vulnerable to predation by other large carnivores. They are weaned at around four months and are independent by around 20 months of age.\nThe cheetah is threatened by habitat loss, conflict with humans, poaching and high susceptibility to diseases. In 2016, the global cheetah population was estimated at 7,100 individuals in the wild; it is listed as Vulnerable on the IUCN Red List. It has been widely depicted in art, literature, advertising, and animation. It was tamed in ancient Egypt and trained for hunting ungulates in the Arabian Peninsula and India. It has been kept in zoos since the early 19th century.'
```


LangSmith trace: https://smith.langchain.com/public/a61304fa-e5a5-4c64-a268-b0aef1130d53/r

## 生成后处理

另一种方法是对我们的模型生成进行后处理。在这个例子中，我们将首先生成一个答案，然后要求模型用引用对其答案进行注释。这种方法的缺点当然是速度较慢且成本较高，因为需要进行两次模型调用。

让我们将其应用于我们的初始链。

```python
class Citation(BaseModel):
    source_id: int = Field(
        ...,
        description="The integer ID of a SPECIFIC source which justifies the answer.",
    )
    quote: str = Field(
        ...,
        description="The VERBATIM quote from the specified source that justifies the answer.",
    )


class AnnotatedAnswer(BaseModel):
    """Annotate the answer to the user question with quote citations that justify the answer."""

    citations: List[Citation] = Field(
        ..., description="Citations from the given sources that justify the answer."
    )


structured_llm = llm.with_structured_output(AnnotatedAnswer)
```

```python
from langchain_core.prompts import MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human", "{question}"),
        MessagesPlaceholder("chat_history", optional=True),
    ]
)
answer = prompt | llm
annotation_chain = prompt | structured_llm

chain = (
    RunnableParallel(
        question=RunnablePassthrough(), docs=(lambda x: x["input"]) | retriever
    )
    .assign(context=format)
    .assign(ai_message=answer)
    .assign(
        chat_history=(lambda x: [x["ai_message"]]),
        answer=(lambda x: x["ai_message"].content),
    )
    .assign(annotations=annotation_chain)
    .pick(["answer", "docs", "annotations"])
)
```

```python
result = chain.invoke({"input": "How fast are cheetahs?"})
```

```python
print(result["answer"])
```
```output
Cheetahs are capable of running at speeds between 93 to 104 km/h (58 to 65 mph). Their specialized adaptations for speed, such as a light build, long thin legs, and a long tail, allow them to be the fastest land animals.
```

```python
result["annotations"]
```

```output
AnnotatedAnswer(citations=[Citation(source_id=0, quote='The cheetah is capable of running at 93 to 104 km/h (58 to 65 mph); it has evolved specialized adaptations for speed, including a light build, long thin legs and a long tail.')])
```

LangSmith trace: https://smith.langchain.com/public/bf5e8856-193b-4ff2-af8d-c0f4fbd1d9cb/r
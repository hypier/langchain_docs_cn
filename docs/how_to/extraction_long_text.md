---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/extraction_long_text.ipynb
---

# 如何处理提取时的长文本

在处理文件时，例如PDF，您可能会遇到超出语言模型上下文窗口的文本。要处理这些文本，请考虑以下策略：

1. **更改LLM** 选择支持更大上下文窗口的不同LLM。
2. **暴力法** 将文档分块，并从每个块中提取内容。
3. **RAG** 将文档分块，索引这些块，仅从看起来“相关”的子集块中提取内容。

请记住，这些策略有不同的权衡，最佳策略可能取决于您正在设计的应用程序！

本指南演示了如何实现策略2和3。

## 设置

我们需要一些示例数据！让我们下载一篇关于[汽车的维基百科文章](https://en.wikipedia.org/wiki/Car)并将其作为LangChain [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html)加载。

```python
import re

import requests
from langchain_community.document_loaders import BSHTMLLoader

# Download the content
response = requests.get("https://en.wikipedia.org/wiki/Car")
# Write it to a file
with open("car.html", "w", encoding="utf-8") as f:
    f.write(response.text)
# Load it with an HTML parser
loader = BSHTMLLoader("car.html")
document = loader.load()[0]
# Clean up code
# Replace consecutive new lines with a single new line
document.page_content = re.sub("\n\n+", "\n", document.page_content)
```

```python
print(len(document.page_content))
```
```output
79174
```

## 定义模式

根据 [提取教程](/docs/tutorials/extraction)，我们将使用 Pydantic 来定义我们希望提取的信息的模式。在这种情况下，我们将提取一个“关键发展”列表（例如，重要的历史事件），其中包括年份和描述。

请注意，我们还包括一个 `evidence` 键，并指示模型逐字提供与文章相关的句子。这使我们能够将提取结果与（模型重构的）原始文档中的文本进行比较。

```python
from typing import List, Optional

from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.pydantic_v1 import BaseModel, Field


class KeyDevelopment(BaseModel):
    """Information about a development in the history of cars."""

    year: int = Field(
        ..., description="The year when there was an important historic development."
    )
    description: str = Field(
        ..., description="What happened in this year? What was the development?"
    )
    evidence: str = Field(
        ...,
        description="Repeat in verbatim the sentence(s) from which the year and description information were extracted",
    )


class ExtractionData(BaseModel):
    """Extracted information about key developments in the history of cars."""

    key_developments: List[KeyDevelopment]


# Define a custom prompt to provide instructions and any additional context.
# 1) You can add examples into the prompt template to improve extraction quality
# 2) Introduce additional parameters to take context into account (e.g., include metadata
#    about the document from which the text was extracted.)
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are an expert at identifying key historic development in text. "
            "Only extract important historic developments. Extract nothing if no important information can be found in the text.",
        ),
        ("human", "{text}"),
    ]
)
```

## 创建提取器

让我们选择一个 LLM。因为我们使用的是工具调用，所以我们需要一个支持工具调用特性的模型。请参见[此表](/docs/integrations/chat)以获取可用的 LLM。

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs
  customVarName="llm"
  openaiParams={`model="gpt-4-0125-preview", temperature=0`}
/>


```python
extractor = prompt | llm.with_structured_output(
    schema=ExtractionData,
    include_raw=False,
)
```

## 暴力破解方法

将文档拆分为多个块，以便每个块适合 LLM 的上下文窗口。

```python
from langchain_text_splitters import TokenTextSplitter

text_splitter = TokenTextSplitter(
    # Controls the size of each chunk
    chunk_size=2000,
    # Controls overlap between chunks
    chunk_overlap=20,
)

texts = text_splitter.split_text(document.page_content)
```

使用 [batch](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html) 功能在每个块中 **并行** 运行提取！

:::tip
您通常可以使用 .batch() 来并行化提取！`.batch` 在底层使用线程池来帮助您并行化工作负载。

如果您的模型通过 API 暴露，这可能会加快您的提取流程！
:::


```python
# Limit just to the first 3 chunks
# so the code can be re-run quickly
first_few = texts[:3]

extractions = extractor.batch(
    [{"text": text} for text in first_few],
    {"max_concurrency": 5},  # limit the concurrency by passing max concurrency!
)
```

### 合并结果

在从各个数据块中提取数据后，我们需要将提取结果合并在一起。

```python
key_developments = []

for extraction in extractions:
    key_developments.extend(extraction.key_developments)

key_developments[:10]
```

```output
[KeyDevelopment(year=1966, description='丰田卡罗拉开始生产，成为历史上最畅销的汽车系列。', evidence='自1966年开始生产的丰田卡罗拉，是历史上最畅销的汽车系列。'),
 KeyDevelopment(year=1769, description='尼古拉-约瑟夫·居诺建造了第一辆蒸汽动力道路车辆。', evidence='法国发明家尼古拉-约瑟夫·居诺在1769年建造了第一辆蒸汽动力道路车辆。'),
 KeyDevelopment(year=1808, description='弗朗索瓦·艾萨克·德·里瓦兹设计并建造了第一辆内燃机汽车。', evidence='瑞士发明家弗朗索瓦·艾萨克·德·里瓦兹在1808年设计并建造了第一辆内燃机汽车。'),
 KeyDevelopment(year=1886, description='卡尔·本茨为他的本茨专利汽车申请了专利，发明了现代汽车。', evidence='现代汽车——一种实用的、可市场化的日常使用汽车——是在1886年由德国发明家卡尔·本茨申请专利的本茨专利汽车发明的。'),
 KeyDevelopment(year=1908, description='福特T型车，最早由大众负担得起的汽车之一，开始生产。', evidence='最早由大众负担得起的汽车之一是福特T型车，始于1908年，由福特汽车公司制造。'),
 KeyDevelopment(year=1888, description="贝尔塔·本茨进行了第一次汽车公路旅行，以证明她丈夫发明的道路适用性。", evidence="1888年8月，卡尔·本茨的妻子贝尔塔·本茨进行了第一次汽车公路旅行，以证明她丈夫发明的道路适用性。"),
 KeyDevelopment(year=1896, description='本茨设计并申请了第一款内燃机平面发动机的专利，称为boxermotor。', evidence='在1896年，本茨设计并申请了第一款内燃机平面发动机的专利，称为boxermotor。'),
 KeyDevelopment(year=1897, description='内塞尔斯多夫车辆制造公司生产了总统汽车，这是世界上第一批工厂制造的汽车之一。', evidence='中欧第一辆汽车以及世界上第一批工厂制造的汽车之一，是由捷克公司内塞尔斯多夫车辆制造公司（后更名为塔特拉）在1897年生产的总统汽车。'),
 KeyDevelopment(year=1890, description='戴姆勒和迈巴赫在坎斯塔特成立了戴姆勒发动机公司（DMG）。', evidence='戴姆勒和迈巴赫于1890年在坎斯塔特成立了戴姆勒发动机公司（DMG）。'),
 KeyDevelopment(year=1891, description='奥古斯特·多里奥和路易·里古洛完成了由戴姆勒动力的标致Type 3驱动的汽油车最长旅行。', evidence='在1891年，奥古斯特·多里奥和他的标致同事路易·里古洛完成了由戴姆勒动力的标致Type 3驱动的汽油车最长旅行，他们自制的标致Type 3完成了从瓦朗蒂尼到巴黎和布雷斯特的2100公里（1300英里）往返。')]
```

## 基于RAG的方法

另一个简单的想法是将文本分块，但不是从每个块中提取信息，而是专注于最相关的块。

:::caution
确定哪些块是相关的可能很困难。

例如，在我们这里使用的`car`文章中，大部分文章包含关键的开发信息。因此，通过使用**RAG**，我们可能会丢弃很多相关信息。

我们建议您根据自己的用例进行实验，确定这种方法是否有效。
:::

要实现基于RAG的方法：

1. 将您的文档分块并进行索引（例如，在向量存储中）；
2. 在`extractor`链前添加一个使用向量存储的检索步骤。

以下是一个依赖于`FAISS`向量存储的简单示例。

```python
from langchain_community.vectorstores import FAISS
from langchain_core.documents import Document
from langchain_core.runnables import RunnableLambda
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

texts = text_splitter.split_text(document.page_content)
vectorstore = FAISS.from_texts(texts, embedding=OpenAIEmbeddings())

retriever = vectorstore.as_retriever(
    search_kwargs={"k": 1}
)  # 仅从第一篇文档中提取
```

在这种情况下，RAG提取器只关注顶部文档。

```python
rag_extractor = {
    "text": retriever | (lambda docs: docs[0].page_content)  # 获取顶部文档的内容
} | extractor
```

```python
results = rag_extractor.invoke("与汽车相关的关键发展")
```

```python
for key_development in results.key_developments:
    print(key_development)
```
```output
year=1869 description='Mary Ward成为爱尔兰Parsonstown记录在案的第一起汽车事故的受害者之一。' evidence='Mary Ward在1869年成为爱尔兰Parsonstown记录在案的第一起汽车事故的受害者之一。'
year=1899 description="Henry Bliss成为美国纽约市首批行人汽车事故的受害者之一。" evidence="Henry Bliss在1899年成为美国纽约市首批行人汽车事故的受害者之一。"
year=2030 description='阿姆斯特丹将禁止所有化石燃料车辆。' evidence='从2030年起，阿姆斯特丹将禁止所有化石燃料车辆。'
```

## 常见问题

不同的方法在成本、速度和准确性方面各有优缺点。

注意这些问题：

* 内容分块意味着如果信息分散在多个块中，LLM 可能无法提取信息。
* 大块重叠可能导致相同的信息被提取两次，因此要做好去重的准备！
* LLM 可能会编造数据。如果在大量文本中寻找单一事实并使用暴力破解的方法，您可能会得到更多编造的数据。
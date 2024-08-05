---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/pebblo/pebblo_retrieval_qa.ipynb
---

# 基于身份的 RAG 使用 PebbloRetrievalQA

> PebbloRetrievalQA 是一个带有身份和语义强制的检索链，用于针对向量数据库进行问答。

本笔记本涵盖了如何使用身份和语义强制（拒绝主题/实体）来检索文档。
有关 Pebblo 及其 SafeRetriever 功能的更多详细信息，请访问 [Pebblo 文档](https://daxa-ai.github.io/pebblo/retrieval_chain/)

### 步骤：

1. **加载文档：**
我们将把带有授权和语义元数据的文档加载到内存中的 Qdrant 向量存储中。这个向量存储将作为 PebbloRetrievalQA 中的检索器使用。

> **注意：** 建议使用 [PebbloSafeLoader](https://daxa-ai.github.io/pebblo/rag) 作为加载带有身份验证和语义元数据的文档的对应工具。`PebbloSafeLoader` 确保安全高效地加载文档，同时保持元数据的完整性。

2. **测试强制机制：**
   我们将分别测试身份和语义强制机制。对于每个用例，我们将定义一个特定的“请求”函数，包含所需的上下文 (*auth_context* 和 *semantic_context*)，然后提出我们的问题。

## 设置

### 依赖

在本教程中，我们将使用 OpenAI LLM、OpenAI 嵌入和 Qdrant 向量存储。



```python
%pip install --upgrade --quiet langchain langchain_core langchain-community langchain-openai qdrant_client
```

### 身份感知数据摄取

在这里，我们使用 Qdrant 作为向量数据库；然而，您可以使用任何支持的向量数据库。

**PebbloRetrievalQA 链支持以下向量数据库：**
- Qdrant
- Pinecone
- Postgres（利用 pgvector 扩展）

**加载带有授权和语义信息的向量数据库元数据：**

在此步骤中，我们将源文档的授权和语义信息捕获到每个块的 VectorDB 条目的元数据中的 `authorized_identities`、`pebblo_semantic_topics` 和 `pebblo_semantic_entities` 字段中。

*注意：要使用 PebbloRetrievalQA 链，您必须始终将授权和语义元数据放置在指定字段中。这些字段必须包含字符串列表。*

```python
from langchain_community.vectorstores.qdrant import Qdrant
from langchain_core.documents import Document
from langchain_openai.embeddings import OpenAIEmbeddings
from langchain_openai.llms import OpenAI

llm = OpenAI()
embeddings = OpenAIEmbeddings()
collection_name = "pebblo-identity-and-semantic-rag"

page_content = """
**ACME Corp 财务报告**

**概述：**
ACME Corp，作为并购行业的领先者，发布了截至 2020 年 12 月 31 日的财务报告。
尽管面临严峻的经济环境，ACME Corp 依然展现出强劲的业绩和战略增长。

**财务亮点：**
收入飙升至 5000 万美元，比去年增长 15%，得益于成功的交易完成和新市场的扩展。
净利润达到 1200 万美元，展现出 24% 的健康利润率。

**关键指标：**
总资产激增至 8000 万美元，反映出 20% 的增长，突显了 ACME Corp 强大的财务状况和资产基础。
此外，公司维持了 0.5 的保守债务与股本比率，确保可持续的财务稳定性。

**未来展望：**
ACME Corp 对未来持乐观态度，计划在全球并购领域抓住新兴机会。
公司致力于为股东创造价值，同时保持道德商业实践。

**银行账户详情：**
如需咨询或交易，请参考 ACME Corp 的美国银行账户：
账户号码：123456789012
银行名称：虚构美国银行
"""

documents = [
    Document(
        **{
            "page_content": page_content,
            "metadata": {
                "pebblo_semantic_topics": ["financial-report"],
                "pebblo_semantic_entities": ["us-bank-account-number"],
                "authorized_identities": ["finance-team", "exec-leadership"],
                "page": 0,
                "source": "https://drive.google.com/file/d/xxxxxxxxxxxxx/view",
                "title": "ACME Corp 财务报告.pdf",
            },
        }
    )
]

vectordb = Qdrant.from_documents(
    documents,
    embeddings,
    location=":memory:",
    collection_name=collection_name,
)

print("Vectordb loaded.")
```
```output
Vectordb loaded.
```

## 身份强制检索

PebbloRetrievalQA链使用SafeRetrieval来确保用于上下文的片段仅从用户授权的文档中检索。  
为了实现这一点，Gen-AI应用程序需要为这个检索链提供一个授权上下文。  
这个*auth_context*应该填充访问Gen-AI应用程序的用户的身份和授权组。

以下是`PebbloRetrievalQA`的示例代码，包含来自访问RAG应用程序的用户的`user_auth`（用户授权列表，可能包括他们的用户ID和他们所属的组），并传递到`auth_context`中。

```python
from langchain_community.chains import PebbloRetrievalQA
from langchain_community.chains.pebblo_retrieval.models import AuthContext, ChainInput

# Initialize PebbloRetrievalQA chain
qa_chain = PebbloRetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectordb.as_retriever(),
    app_name="pebblo-identity-rag",
    description="Identity Enforcement app using PebbloRetrievalQA",
    owner="ACME Corp",
)


def ask(question: str, auth_context: dict):
    """
    Ask a question to the PebbloRetrievalQA chain
    """
    auth_context_obj = AuthContext(**auth_context) if auth_context else None
    chain_input_obj = ChainInput(query=question, auth_context=auth_context_obj)
    return qa_chain.invoke(chain_input_obj.dict())
```

### 1. 授权用户的问题

我们为授权身份 `["finance-team", "exec-leadership"]` 输入了数据，因此具有授权身份/组 `finance-team` 的用户应该能够得到正确的答案。

```python
auth = {
    "user_id": "finance-user@acme.org",
    "user_auth": [
        "finance-team",
    ],
}

question = "Share the financial performance of ACME Corp for the year 2020"
resp = ask(question, auth)
print(f"Question: {question}\n\nAnswer: {resp['result']}")
```
```output
Question: Share the financial performance of ACME Corp for the year 2020

Answer: 
Revenue: $50 million (15% increase from previous year)
Net profit: $12 million (24% margin)
Total assets: $80 million (20% growth)
Debt-to-equity ratio: 0.5
```

### 2. 未授权用户的问题

由于用户的授权身份/组 `eng-support` 不在授权身份 `["finance-team", "exec-leadership"]` 中，因此我们不应该收到答案。


```python
auth = {
    "user_id": "eng-user@acme.org",
    "user_auth": [
        "eng-support",
    ],
}

question = "Share the financial performance of ACME Corp for the year 2020"
resp = ask(question, auth)
print(f"Question: {question}\n\nAnswer: {resp['result']}")
```
```output
Question: Share the financial performance of ACME Corp for the year 2020

Answer:  I don't know.
```

### 3. 使用 PromptTemplate 提供额外指令
您可以使用 PromptTemplate 为 LLM 提供额外指令，以生成自定义响应。

```python
from langchain_core.prompts import PromptTemplate

prompt_template = PromptTemplate.from_template(
    """
Answer the question using the provided context. 
If no context is provided, just say "I'm sorry, but that information is unavailable, or Access to it is restricted.".

Question: {question}
"""
)

question = "Share the financial performance of ACME Corp for the year 2020"
prompt = prompt_template.format(question=question)
```

#### 3.1 授权用户提问

```python
auth = {
    "user_id": "finance-user@acme.org",
    "user_auth": [
        "finance-team",
    ],
}
resp = ask(prompt, auth)
print(f"Question: {question}\n\nAnswer: {resp['result']}")
```
```output
Question: Share the financial performance of ACME Corp for the year 2020

Answer: 
Revenue soared to $50 million, marking a 15% increase from the previous year, and net profit reached $12 million, showcasing a healthy margin of 24%. Total assets also grew by 20% to $80 million, and the company maintained a conservative debt-to-equity ratio of 0.5.
```
#### 3.2 未授权用户提问

```python
auth = {
    "user_id": "eng-user@acme.org",
    "user_auth": [
        "eng-support",
    ],
}
resp = ask(prompt, auth)
print(f"Question: {question}\n\nAnswer: {resp['result']}")
```
```output
Question: Share the financial performance of ACME Corp for the year 2020

Answer: 
I'm sorry, but that information is unavailable, or Access to it is restricted.
```

## 语义强制检索

PebbloRetrievalQA 链使用 SafeRetrieval 确保在上下文中使用的片段仅从符合提供的语义上下文的文档中检索。 
为此，Gen-AI 应用程序必须为此检索链提供语义上下文。 
此 `semantic_context` 应包括应被拒绝的主题和实体，以防用户访问 Gen-AI 应用程序。

下面是一个包含 `topics_to_deny` 和 `entities_to_deny` 的 PebbloRetrievalQA 示例代码。这些内容作为 `semantic_context` 传递给链输入。

```python
from typing import List, Optional

from langchain_community.chains import PebbloRetrievalQA
from langchain_community.chains.pebblo_retrieval.models import (
    ChainInput,
    SemanticContext,
)

# Initialize PebbloRetrievalQA chain
qa_chain = PebbloRetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectordb.as_retriever(),
    app_name="pebblo-semantic-rag",
    description="Semantic Enforcement app using PebbloRetrievalQA",
    owner="ACME Corp",
)


def ask(
    question: str,
    topics_to_deny: Optional[List[str]] = None,
    entities_to_deny: Optional[List[str]] = None,
):
    """
    Ask a question to the PebbloRetrievalQA chain
    """
    semantic_context = dict()
    if topics_to_deny:
        semantic_context["pebblo_semantic_topics"] = {"deny": topics_to_deny}
    if entities_to_deny:
        semantic_context["pebblo_semantic_entities"] = {"deny": entities_to_deny}

    semantic_context_obj = (
        SemanticContext(**semantic_context) if semantic_context else None
    )
    chain_input_obj = ChainInput(query=question, semantic_context=semantic_context_obj)
    return qa_chain.invoke(chain_input_obj.dict())
```

### 1. 无语义强制

由于未应用语义强制，系统应返回答案，而不因与上下文相关的语义标签而排除任何上下文。

```python
topic_to_deny = []
entities_to_deny = []
question = "Share the financial performance of ACME Corp for the year 2020"
resp = ask(question, topics_to_deny=topic_to_deny, entities_to_deny=entities_to_deny)
print(
    f"Topics to deny: {topic_to_deny}\nEntities to deny: {entities_to_deny}\n"
    f"Question: {question}\nAnswer: {resp['result']}"
)
```
```output
Topics to deny: []
Entities to deny: []
Question: Share the financial performance of ACME Corp for the year 2020
Answer: 
Revenue for ACME Corp increased by 15% to $50 million in 2020, with a net profit of $12 million and a strong asset base of $80 million. The company also maintained a conservative debt-to-equity ratio of 0.5.
```

### 2. 拒绝财务报告主题

数据已被导入，主题为： `["financial-report"]`。因此，拒绝 `financial-report` 主题的应用程序不应收到答案。

```python
topic_to_deny = ["financial-report"]
entities_to_deny = []
question = "Share the financial performance of ACME Corp for the year 2020"
resp = ask(question, topics_to_deny=topic_to_deny, entities_to_deny=entities_to_deny)
print(
    f"Topics to deny: {topic_to_deny}\nEntities to deny: {entities_to_deny}\n"
    f"Question: {question}\nAnswer: {resp['result']}"
)
```
```output
Topics to deny: ['financial-report']
Entities to deny: []
Question: Share the financial performance of ACME Corp for the year 2020
Answer: 

Unfortunately, I do not have access to the financial performance of ACME Corp for the year 2020.
```

### 3. 拒绝 us-bank-account-number 实体
由于实体 `us-bank-account-number` 被拒绝，系统不应返回答案。


```python
topic_to_deny = []
entities_to_deny = ["us-bank-account-number"]
question = "Share the financial performance of ACME Corp for the year 2020"
resp = ask(question, topics_to_deny=topic_to_deny, entities_to_deny=entities_to_deny)
print(
    f"Topics to deny: {topic_to_deny}\nEntities to deny: {entities_to_deny}\n"
    f"Question: {question}\nAnswer: {resp['result']}"
)
```
```output
Topics to deny: []
Entities to deny: ['us-bank-account-number']
Question: Share the financial performance of ACME Corp for the year 2020
Answer:  I don't have information about ACME Corp's financial performance for 2020.
```
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/kdbai.ipynb
---

# KDB.AI

> [KDB.AI](https://kdb.ai/) 是一个强大的基于知识的向量数据库和搜索引擎，允许您通过提供先进的搜索、推荐和个性化功能，使用实时数据构建可扩展、可靠的 AI 应用程序。

[此示例](https://github.com/KxSystems/kdbai-samples/blob/main/document_search/document_search.ipynb) 演示了如何使用 KDB.AI 在非结构化文本文档上运行语义搜索。

要访问您的端点和 API 密钥，请 [在此注册 KDB.AI](https://kdb.ai/get-started/)。

要设置您的开发环境，请按照 [KDB.AI 先决条件页面](https://code.kx.com/kdbai/pre-requisites.html) 上的说明进行操作。

以下示例演示了您可以通过 LangChain 与 KDB.AI 进行交互的一些方式。

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 以使用此集成。

## 导入所需的包


```python
import os
import time
from getpass import getpass

import kdbai_client as kdbai
import pandas as pd
import requests
from langchain.chains import RetrievalQA
from langchain_community.document_loaders import PyPDFLoader
from langchain_community.vectorstores import KDBAI
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
```


```python
KDBAI_ENDPOINT = input("KDB.AI endpoint: ")
KDBAI_API_KEY = getpass("KDB.AI API key: ")
os.environ["OPENAI_API_KEY"] = getpass("OpenAI API Key: ")
```
```output
KDB.AI endpoint:  https://ui.qa.cld.kx.com/instance/pcnvlmi860
KDB.AI API key:  ········
OpenAI API Key:  ········
```

```python
TEMP = 0.0
K = 3
```

## 创建 KBD.AI 会话


```python
print("Create a KDB.AI session...")
session = kdbai.Session(endpoint=KDBAI_ENDPOINT, api_key=KDBAI_API_KEY)
```
```output
Create a KDB.AI session...
```

## 创建表格


```python
print('Create table "documents"...')
schema = {
    "columns": [
        {"name": "id", "pytype": "str"},
        {"name": "text", "pytype": "bytes"},
        {
            "name": "embeddings",
            "pytype": "float32",
            "vectorIndex": {"dims": 1536, "metric": "L2", "type": "hnsw"},
        },
        {"name": "tag", "pytype": "str"},
        {"name": "title", "pytype": "bytes"},
    ]
}
table = session.create_table("documents", schema)
```
```output
Create table "documents"...
```

```python
%%time
URL = "https://www.conseil-constitutionnel.fr/node/3850/pdf"
PDF = "Déclaration_des_droits_de_l_homme_et_du_citoyen.pdf"
open(PDF, "wb").write(requests.get(URL).content)
```
```output
CPU times: user 44.1 ms, sys: 6.04 ms, total: 50.2 ms
Wall time: 213 ms
```


```output
562978
```

## 读取 PDF


```python
%%time
print("读取 PDF...")
loader = PyPDFLoader(PDF)
pages = loader.load_and_split()
len(pages)
```
```output
读取 PDF...
CPU times: user 156 ms, sys: 12.5 ms, total: 169 ms
Wall time: 183 ms
```


```output
3
```

## 从 PDF 文本创建向量数据库


```python
%%time
print("从 PDF 文本创建向量数据库...")
embeddings = OpenAIEmbeddings(model="text-embedding-ada-002")
texts = [p.page_content for p in pages]
metadata = pd.DataFrame(index=list(range(len(texts))))
metadata["tag"] = "law"
metadata["title"] = "Déclaration des Droits de l'Homme et du Citoyen de 1789".encode(
    "utf-8"
)
vectordb = KDBAI(table, embeddings)
vectordb.add_texts(texts=texts, metadatas=metadata)
```
```output
从 PDF 文本创建向量数据库...
CPU times: user 211 ms, sys: 18.4 ms, total: 229 ms
Wall time: 2.23 s
```


```output
['3ef27d23-47cf-419b-8fe9-5dfae9e8e895',
 'd3a9a69d-28f5-434b-b95b-135db46695c8',
 'd2069bda-c0b8-4791-b84d-0c6f84f4be34']
```

## 创建 LangChain 管道


```python
%%time
print("Create LangChain Pipeline...")
qabot = RetrievalQA.from_chain_type(
    chain_type="stuff",
    llm=ChatOpenAI(model="gpt-3.5-turbo-16k", temperature=TEMP),
    retriever=vectordb.as_retriever(search_kwargs=dict(k=K)),
    return_source_documents=True,
)
```
```output
Create LangChain Pipeline...
CPU times: user 40.8 ms, sys: 4.69 ms, total: 45.5 ms
Wall time: 44.7 ms
```

## 总结文件内容


```python
%%time
Q = "Summarize the document in English:"
print(f"\n\n{Q}\n")
print(qabot.invoke(dict(query=Q))["result"])
```
```output


Summarize the document in English:

The document is the Declaration of the Rights of Man and of the Citizen of 1789. It was written by the representatives of the French people and aims to declare the natural, inalienable, and sacred rights of every individual. These rights include freedom, property, security, and resistance to oppression. The document emphasizes the importance of equality and the principle that sovereignty resides in the nation. It also highlights the role of law in protecting individual rights and ensuring the common good. The document asserts the right to freedom of thought, expression, and religion, as long as it does not disturb public order. It emphasizes the need for a public force to guarantee the rights of all citizens and the importance of a fair and equal distribution of public contributions. The document also recognizes the right of citizens to hold public officials accountable and states that any society without the guarantee of rights and separation of powers does not have a constitution. Finally, it affirms the inviolable and sacred nature of property, stating that it can only be taken away for public necessity and with just compensation.
CPU times: user 144 ms, sys: 50.2 ms, total: 194 ms
Wall time: 4.96 s
```

## 查询数据


```python
%%time
Q = "这是一条公平的法律吗，为什么？"
print(f"\n\n{Q}\n")
print(qabot.invoke(dict(query=Q))["result"])
```
```output


这是一条公平的法律吗，为什么？

作为一个AI语言模型，我没有个人观点。然而，我可以根据给定的背景提供一些分析。所提供的文本摘自1789年《人权和公民权宣言》，该宣言被认为是人权历史上的基础性文件。它概述了个人的自然和不可剥夺的权利，例如自由、财产、安全和抵抗压迫。它还强调了平等原则、法治和权力分立。

这条法律是否被认为是公平的，具有主观性，可能因个人观点和社会规范而异。然而，许多人认为该宣言中概述的原则和权利是基本和公正的。值得注意的是，该宣言是确立平等和个人权利原则的重要一步，并对全球后续的人权文件产生了影响。
CPU times: user 85.1 ms, sys: 5.93 ms, total: 91.1 ms
Wall time: 5.11 s
```

```python
%%time
Q = "人的权利和义务、公民的权利和义务以及社会的权利和义务是什么？"
print(f"\n\n{Q}\n")
print(qabot.invoke(dict(query=Q))["result"])
```
```output


人的权利和义务、公民的权利和义务以及社会的权利和义务是什么？

根据1789年《人权和公民权宣言》，人的权利和义务、公民的权利和义务以及社会的权利和义务如下：

人的权利：
1. 人生而自由，在权利上平等。社会的差别只能建立在共同的利益上。
2. 政治协会的目的是保护人的自然和不可剥夺的权利，即自由、财产、安全和抵抗压迫。
3. 主权原则本质上属于国家。任何机构或个人不得行使不明确来源于国家的权威。
4. 自由是指能够做任何不损害他人的事情。每个人行使自然权利的限制仅限于确保其他社会成员享有这些权利的范围。这些限制只能由法律确定。
5. 法律有权禁止仅对社会有害的行为。法律未禁止的任何行为不得被阻止，任何人都不能被强迫做法律不命令的事情。
6. 法律是一般意志的表现。所有公民都有权亲自或通过代表参与法律的制定。法律对所有人都必须相同，无论是保护还是惩罚。所有公民在法律面前平等，依据其能力平等有资格获得所有公共荣誉、职位和工作，除了其美德和才能的区别。
7. 任何人不得被指控、逮捕或拘留，除非在法律规定的情况下并按照法律所规定的程序。那些请求、加速、执行或导致执行任意命令的人必须受到惩罚。但是，任何因法律被传唤或拘留的公民必须立即服从；抵抗将使其承担责任。
8. 法律应仅设定严格和明显必要的惩罚，任何人不得因法律在犯罪之前未建立和公布而受到惩罚，并依法适用。
9. 每个人在被宣告有罪之前应被视为无罪，如果被认为必须逮捕他，任何不必要的严厉措施都必须受到法律的严厉制裁。
10. 任何人不得因其意见而受到干扰，即使是宗教意见，只要其表现不干扰法律所规定的公共秩序。
11. 思想和意见的自由交流是人类最宝贵的权利之一。因此，每位公民可以自由地讲话、写作和印刷，除非法律规定的情况下回应这种自由的滥用。
12. 人权和公民权的保障需要一支公共力量。因此，这种力量是为了所有人的利益而建立的，而不是为了那些被赋予权力的人的特定利益。
13. 为了维护公共力量和管理费用，必须有共同的贡献。这种贡献必须在所有公民中按其能力平等分配。
14. 所有公民都有权通过自己或其代表确认公共贡献的必要性，自由同意，跟踪其使用，并确定其金额、基础、征收和持续时间。
15. 社会有权要求任何公共代理人对其管理进行说明。
16. 任何未能确保权利保障或未能确定权力分立的社会都没有宪法。
17. 财产是不可侵犯和神圣的权利，任何人不得被剥夺财产，除非公共必要性经过合法确定，显然要求这样，并且在公平和事先赔偿的条件下。

公民的义务：
宣言没有明确提到公民的义务，但强调了法律的重要性和公民遵守法律的必要性。它还强调了公民参与法律制定和对公共代理人进行问责的责任。

社会的义务：
宣言没有明确提到社会的义务，但强调了保障人权和公民权的重要性。社会有责任确保这些权利的保护和权力的分立，并为所有人提供公共力量。
CPU times: user 86.5 ms, sys: 5.45 ms, total: 92 ms
Wall time: 14.9 s
```

```python
%%time
Q = "这条法律实用吗？"
print(f"\n\n{Q}\n")
print(qabot.invoke(dict(query=Q))["result"])
```
```output


这条法律实用吗？

作为一个AI语言模型，我无法对法律是否实用提供个人意见或主观判断。所提供的文本摘自法国宪法和1789年《人权和公民权宣言》。这些文本概述了构成法国法律体系基础的基本权利和原则。法律的实用性往往是一个解释问题，并可能因上下文和具体情况而异。最终，由法律专家、立法者和司法机关来确定这些法律在具体案件中的实用性和适用性。
CPU times: user 91.4 ms, sys: 5.89 ms, total: 97.3 ms
Wall time: 2.78 s
```

## 清理文档表


```python
# Clean up KDB.AI "documents" table and index for similarity search
# so this notebook could be played again and again
session.table("documents").drop()
```



```output
True
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/vectara/vectara_chat.ipynb
---

# Vectara 聊天

[Vectara](https://vectara.com/) 提供一个可信的生成式 AI 平台，使组织能够快速创建类似 ChatGPT 的体验（一个 AI 助手），该体验基于他们拥有的数据、文档和知识（从技术上讲，它是检索增强生成即服务）。

Vectara 无服务器 RAG 即服务提供了所有 RAG 组件，背后有一个易于使用的 API，包括：
1. 从文件中提取文本的方法（PDF、PPT、DOCX 等）
2. 基于机器学习的分块，提供最先进的性能。
3. [Boomerang](https://vectara.com/how-boomerang-takes-retrieval-augmented-generation-to-the-next-level-via-grounded-generation/) 嵌入模型。
4. 自有的内部向量数据库，用于存储文本块和嵌入向量。
5. 一个查询服务，自动将查询编码为嵌入，并检索最相关的文本段落（包括支持 [Hybrid Search](https://docs.vectara.com/docs/api-reference/search-apis/lexical-matching) 和 [MMR](https://vectara.com/get-diverse-results-and-comprehensive-summaries-with-vectaras-mmr-reranker/)）
7. 一个 LLM 用于基于检索到的文档（上下文）创建 [生成摘要](https://docs.vectara.com/docs/learn/grounded-generation/grounded-generation-overview)，包括引用。

有关如何使用 API 的更多信息，请参见 [Vectara API 文档](https://docs.vectara.com/docs/)。

本笔记本展示了如何使用 Vectara 的 [聊天](https://docs.vectara.com/docs/api-reference/chat-apis/chat-apis-overview) 功能。

# 开始使用

要开始使用，请按照以下步骤操作：
1. 如果您还没有账户，请[注册](https://www.vectara.com/integrations/langchain)一个免费的 Vectara 账户。注册完成后，您将获得一个 Vectara 客户 ID。您可以通过点击 Vectara 控制台窗口右上角的您的名字找到您的客户 ID。
2. 在您的账户中，您可以创建一个或多个语料库。每个语料库表示一个存储从输入文档中获取的文本数据的区域。要创建一个语料库，请使用 **"创建语料库"** 按钮。然后为您的语料库提供一个名称和描述。您可以选择定义过滤属性并应用一些高级选项。如果您点击您创建的语料库，您可以在顶部看到它的名称和语料库 ID。
3. 接下来，您需要创建 API 密钥以访问语料库。在语料库视图中点击 **"访问控制"** 选项卡，然后点击 **"创建 API 密钥"** 按钮。为您的密钥命名，并选择您希望密钥是仅查询还是查询+索引。点击 "创建"，您现在拥有一个有效的 API 密钥。请保密此密钥。

要将 LangChain 与 Vectara 一起使用，您需要这三个值： `customer ID`， `corpus ID` 和 `api_key`。您可以通过两种方式将它们提供给 LangChain：

1. 在您的环境中包含这三个变量： `VECTARA_CUSTOMER_ID`， `VECTARA_CORPUS_ID` 和 `VECTARA_API_KEY`。

   例如，您可以使用 os.environ 和 getpass 设置这些变量，如下所示：

```python
import os
import getpass

os.environ["VECTARA_CUSTOMER_ID"] = getpass.getpass("Vectara Customer ID:")
os.environ["VECTARA_CORPUS_ID"] = getpass.getpass("Vectara Corpus ID:")
os.environ["VECTARA_API_KEY"] = getpass.getpass("Vectara API Key:")
```

2. 将它们添加到 `Vectara` 向量存储构造函数中：

```python
vectara = Vectara(
                vectara_customer_id=vectara_customer_id,
                vectara_corpus_id=vectara_corpus_id,
                vectara_api_key=vectara_api_key
            )
```
在本笔记本中，我们假设它们在环境中提供。


```python
import os

os.environ["VECTARA_API_KEY"] = "<YOUR_VECTARA_API_KEY>"
os.environ["VECTARA_CORPUS_ID"] = "<YOUR_VECTARA_CORPUS_ID>"
os.environ["VECTARA_CUSTOMER_ID"] = "<YOUR_VECTARA_CUSTOMER_ID>"

from langchain_community.vectorstores import Vectara
from langchain_community.vectorstores.vectara import (
    RerankConfig,
    SummaryConfig,
    VectaraQueryConfig,
)
```

## Vectara Chat 解释

在大多数使用 LangChain 创建聊天机器人的场景中，必须集成一个特殊的 `memory` 组件，该组件维护聊天会话的历史记录，然后利用这些历史记录确保聊天机器人了解对话历史。

使用 Vectara Chat - 所有这些操作都由 Vectara 在后台自动执行。您可以查看 [Chat](https://docs.vectara.com/docs/api-reference/chat-apis/chat-apis-overview) 文档以获取详细信息，了解其实现的内部机制，但使用 LangChain，您只需在 Vectara 向量存储中启用该功能即可。

让我们看一个例子。首先，我们加载 SOTU 文档（请记住，文本提取和分块在 Vectara 平台上会自动进行）：

```python
from langchain.document_loaders import TextLoader

loader = TextLoader("state_of_the_union.txt")
documents = loader.load()

vectara = Vectara.from_documents(documents, embedding=None)
```

现在我们使用 `as_chat` 方法创建一个聊天可运行对象：

```python
summary_config = SummaryConfig(is_enabled=True, max_results=7, response_lang="eng")
rerank_config = RerankConfig(reranker="mmr", rerank_k=50, mmr_diversity_bias=0.2)
config = VectaraQueryConfig(
    k=10, lambda_val=0.005, rerank_config=rerank_config, summary_config=summary_config
)

bot = vectara.as_chat(config)
```

这是一个没有聊天历史的提问示例：

```python
bot.invoke("What did the president say about Ketanji Brown Jackson?")["answer"]
```

```output
'The President expressed gratitude to Justice Breyer and highlighted the significance of nominating Ketanji Brown Jackson to the Supreme Court, praising her legal expertise and commitment to upholding excellence [1]. The President also reassured the public about the situation with gas prices and the conflict in Ukraine, emphasizing unity with allies and the belief that the world will emerge stronger from these challenges [2][4]. Additionally, the President shared personal experiences related to economic struggles and the importance of passing the American Rescue Plan to support those in need [3]. The focus was also on job creation and economic growth, acknowledging the impact of inflation on families [5]. While addressing cancer as a significant issue, the President discussed plans to enhance cancer research and support for patients and families [7].'
```

这是一个带有一些聊天历史的提问示例：

```python
bot.invoke("Did he mention who she suceeded?")["answer"]
```

```output
"In his remarks, the President specified that Ketanji Brown Jackson is succeeding Justice Breyer on the United States Supreme Court[1]. The President praised Jackson as a top legal mind who will continue Justice Breyer's legacy of excellence. The nomination of Jackson was highlighted as a significant constitutional responsibility of the President[1]. The President emphasized the importance of this nomination and the qualities that Jackson brings to the role. The focus was on the transition from Justice Breyer to Judge Ketanji Brown Jackson on the Supreme Court[1]."
```

## 流式聊天

当然，聊天机器人界面也支持流式传输。  
您只需使用 `stream` 方法，而不是 `invoke` 方法：

```python
output = {}
curr_key = None
for chunk in bot.stream("what about her accopmlishments?"):
    for key in chunk:
        if key not in output:
            output[key] = chunk[key]
        else:
            output[key] += chunk[key]
        if key == "answer":
            print(chunk[key], end="", flush=True)
        curr_key = key
```
```output
Judge Ketanji Brown Jackson is a nominee for the United States Supreme Court, known for her legal expertise and experience as a former litigator. She is praised for her potential to continue the legacy of excellence on the Court[1]. While the search results provide information on various topics like innovation, economic growth, and healthcare initiatives, they do not directly address Judge Ketanji Brown Jackson's specific accomplishments. Therefore, I do not have enough information to answer this question.
```
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/llm_rails.ipynb
---

# LLMRails

>[LLMRails](https://www.llmrails.com/) 是一个用于构建 GenAI 应用程序的 API 平台。它提供了一个易于使用的 API，用于文档索引和查询，由 LLMRails 管理，并针对性能和准确性进行了优化。有关如何使用 API 的更多信息，请参见 [LLMRails API 文档](https://docs.llmrails.com/)。

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 以使用此集成。

本笔记本展示了如何使用与 `LLMRails` 与 langchain 集成相关的功能。请注意，与此类别中的许多其他集成不同，LLMRails 提供了端到端的管理服务，用于检索增强生成，包括：
1. 从文档文件中提取文本并将其分块成句子的方式。
2. 自有的嵌入模型和向量存储 - 每个文本段落都被编码为向量嵌入并存储在 LLMRails 内部向量存储中。
3. 一个查询服务，自动将查询编码为嵌入，并检索最相关的文本段落（包括对 [混合搜索](https://docs.llmrails.com/datastores/search) 的支持）。

所有这些都在此 LangChain 集成中得到支持。

# 设置

您需要一个 LLMRails 帐户才能将 LLMRails 与 LangChain 一起使用。要开始，请按照以下步骤操作：
1. 如果您还没有帐户，请[注册](https://console.llmrails.com/signup)一个 LLMRails 帐户。
2. 接下来，您需要创建 API 密钥以访问 API。在语料库视图中单击 **“API Keys”** 选项卡，然后单击 **“Create API Key”** 按钮。给您的密钥命名。单击“Create key”，您现在拥有一个有效的 API 密钥。请保密此密钥。

要将 LangChain 与 LLMRails 一起使用，您需要具有以下值：api_key。
您可以通过两种方式将其提供给 LangChain：

1. 在您的环境中包含这两个变量：`LLM_RAILS_API_KEY`，`LLM_RAILS_DATASTORE_ID`。

> 例如，您可以使用 os.environ 和 getpass 以如下方式设置这些变量：

```python
import os
import getpass

os.environ["LLM_RAILS_API_KEY"] = getpass.getpass("LLMRails API Key:")
os.environ["LLM_RAILS_DATASTORE_ID"] = getpass.getpass("LLMRails Datastore Id:")
```

1. 在创建 LLMRails 向量存储对象时将它们作为参数提供：

```python
vectorstore = LLMRails(
    api_key=llm_rails_api_key,
    datastore_id=datastore_id
)
```

## 添加文本

要将文本添加到您的数据存储，首先您需要访问 [Datastores](https://console.llmrails.com/datastores) 页面并创建一个。点击创建数据存储按钮，选择一个名称和嵌入模型。然后从新创建的数据存储设置中获取您的数据存储 ID。

```python
%pip install tika
```
```output
Collecting tika
  Downloading tika-2.6.0.tar.gz (27 kB)
  Preparing metadata (setup.py) ... [?25ldone
[?25hRequirement already satisfied: setuptools in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from tika) (68.2.2)
Requirement already satisfied: requests in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from tika) (2.31.0)
Requirement already satisfied: charset-normalizer<4,>=2 in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from requests->tika) (2.1.1)
Requirement already satisfied: idna<4,>=2.5 in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from requests->tika) (3.4)
Requirement already satisfied: urllib3<3,>=1.21.1 in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from requests->tika) (1.26.16)
Requirement already satisfied: certifi>=2017.4.17 in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from requests->tika) (2022.12.7)
Building wheels for collected packages: tika
  Building wheel for tika (setup.py) ... [?25ldone
[?25h  Created wheel for tika: filename=tika-2.6.0-py3-none-any.whl size=32621 sha256=b3f03c9dbd7f347d712c49027704d48f1a368f31560be9b4ee131f79a52e176f
  Stored in directory: /Users/omaraly/Library/Caches/pip/wheels/27/ba/2f/37420d1191bdae5e855d69b8e913673045bfd395cbd78ad697
Successfully built tika
Installing collected packages: tika
Successfully installed tika-2.6.0

[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m A new release of pip is available: [0m[31;49m23.3.1[0m[39;49m -> [0m[32;49m23.3.2[0m
[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m To update, run: [0m[32;49mpip install --upgrade pip[0m
Note: you may need to restart the kernel to use updated packages.
```

```python
import os

from langchain_community.vectorstores import LLMRails

os.environ["LLM_RAILS_DATASTORE_ID"] = "Your datastore id "
os.environ["LLM_RAILS_API_KEY"] = "Your API Key"

llm_rails = LLMRails.from_texts(["Your text here"])
```

## 相似性搜索

使用 LLMRails 的最简单场景是执行相似性搜索。

```python
query = "What do you plan to do about national security?"
found_docs = llm_rails.similarity_search(query, k=5)
```

```python
print(found_docs[0].page_content)
```
```output
6  N A T I O N A L S E C U R I T Y S T R A T E G Y Page 7 

This National Security Strategy lays out our plan to achieve a better future of a free, open, secure, and prosperous world.

Our strategy is rooted in our national interests: to protect the security of the American people; to expand economic prosperity and opportunity; and to realize and defend the democratic values at the heart of the American way of life.

We can do none of this alone and we do not have to.

Most nations around the world define their interests in ways that are compatible with ours.

We will build the strongest and broadest possible coalition of nations that seek to cooperate with each other, while competing with those powers that offer a darker vision and thwarting their efforts to threaten our interests.

Our Enduring Role The need for a strong and purposeful American role in the world has never been greater.

The world is becoming more divided and unstable.

Global increases in inflation since the COVID-19 pandemic began have made life more difficult for many.

The basic laws and principles governing relations among nations, including the United Nations Charter and the protection it affords all states from being invaded by their neighbors or having their borders redrawn by force, are under attack.

The risk of conflict between major powers is increasing.

Democracies and autocracies are engaged in a contest to show which system of governance can best deliver for their people and the world.

Competition to develop and deploy foundational technologies that will transform our security and economy is intensifying.

Global cooperation on shared interests has frayed, even as the need for that cooperation takes on existential importance.

The scale of these changes grows with each passing year, as do the risks of inaction.

Although the international environment has become more contested, the United States remains the world’s leading power.
```

## 相似性搜索与评分

有时我们可能希望执行搜索，同时获得相关性评分，以了解特定结果的优劣。

```python
query = "What is your approach to national defense"
found_docs = llm_rails.similarity_search_with_score(
    query,
    k=5,
)
```

```python
document, score = found_docs[0]
print(document.page_content)
print(f"\nScore: {score}")
```
```output
But we will do so as the last resort and only when the objectives and mission are clear and achievable, consistent with our values and laws, alongside non-military tools, and the mission is undertaken with the informed consent of the American people.

Our approach to national defense is described in detail in the 2022 National Defense Strategy.

Our starting premise is that a powerful U.S. military helps advance and safeguard vital U.S. national interests by backstopping diplomacy, confronting aggression, deterring conflict, projecting strength, and protecting the American people and their economic interests.

Amid intensifying competition, the military’s role is to maintain and gain warfighting advantages while limiting those of our competitors.

The military will act urgently to sustain and strengthen deterrence, with the PRC as its pacing challenge.

We will make disciplined choices regarding our national defense and focus our attention on the military’s primary responsibilities: to defend the homeland, and deter attacks and aggression against the United States, our allies and partners, while being prepared to fight and win the Nation’s wars should diplomacy and deterrence fail.

To do so, we will combine our strengths to achieve maximum effect in deterring acts of aggression—an approach we refer to as integrated deterrence (see text box on page 22).

We will operate our military using a campaigning mindset—sequencing logically linked military activities to advance strategy-aligned priorities.

And, we will build a resilient force and defense ecosystem to ensure we can perform these functions for decades to come.

We ended America’s longest war in Afghanistan, and with it an era of major military operations to remake other societies, even as we have maintained the capacity to address terrorist threats to the American people as they emerge.

20  NATIONAL SECURITY STRATEGY Page 21 

A combat-credible military is the foundation of deterrence and America’s ability to prevail in conflict.

Score: 0.5040982687179959
```

## LLMRails 作为检索器

LLMRails 和其他 LangChain 向量存储一样，最常用作 LangChain 检索器：

```python
retriever = llm_rails.as_retriever()
retriever
```

```output
LLMRailsRetriever(vectorstore=<langchain_community.vectorstores.llm_rails.LLMRails object at 0x1235b0e50>)
```

```python
query = "What is your approach to national defense"
retriever.invoke(query)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/zep.ipynb
---

# Zep
> 回顾、理解并提取聊天记录中的数据。赋予个性化AI体验强大功能。

> [Zep](https://www.getzep.com) 是一个用于AI助手应用的长期记忆服务。
> 借助Zep，您可以为AI助手提供回忆过去对话的能力，无论多么久远，
> 同时还可以减少幻觉、延迟和成本。

> 对Zep Cloud感兴趣吗？请参见 [Zep Cloud安装指南](https://help.getzep.com/sdks) 和 [Zep Cloud向量存储示例](https://help.getzep.com/langchain/examples/vectorstore-example)

## 开源安装与设置

> Zep 开源项目: [https://github.com/getzep/zep](https://github.com/getzep/zep)
>
> Zep 开源文档: [https://docs.getzep.com/](https://docs.getzep.com/)

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 才能使用此集成。

## 使用方法

在下面的示例中，我们使用了 Zep 的自动嵌入功能，该功能使用低延迟嵌入模型自动在 Zep 服务器上嵌入文档。

## 注意
- 这些示例使用 Zep 的异步接口。通过去掉方法名称前的 `a` 前缀来调用同步接口。
- 如果传入一个 `Embeddings` 实例，Zep 将使用此实例来嵌入文档，而不是自动嵌入。您还必须将文档集合设置为 `isAutoEmbedded === false`。
- 如果将集合设置为 `isAutoEmbedded === false`，则必须传入一个 `Embeddings` 实例。

## 从文档加载或创建集合


```python
from uuid import uuid4

from langchain_community.document_loaders import WebBaseLoader
from langchain_community.vectorstores import ZepVectorStore
from langchain_community.vectorstores.zep import CollectionConfig
from langchain_text_splitters import RecursiveCharacterTextSplitter

ZEP_API_URL = "http://localhost:8000"  # 这是您 Zep 实例的 API URL
ZEP_API_KEY = "<optional_key>"  # 您 Zep 实例的可选 API 密钥
collection_name = f"babbage{uuid4().hex}"  # 唯一集合名称，仅限字母数字

# 如果我们要创建一个新的 Zep 集合，则需要集合配置
config = CollectionConfig(
    name=collection_name,
    description="<optional description>",
    metadata={"optional_metadata": "与该集合相关联"},
    is_auto_embedded=True,  # 我们将让 Zep 使用其低延迟嵌入器嵌入我们的文档
    embedding_dimensions=1536,  # 这应与您配置的 Zep 使用的模型匹配。
)

# 加载文档
article_url = "https://www.gutenberg.org/cache/epub/71292/pg71292.txt"
loader = WebBaseLoader(article_url)
documents = loader.load()

# 将其拆分为块
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

# 实例化 VectorStore。由于该集合在 Zep 中尚不存在，
# 它将被创建并填充我们传入的文档。
vs = ZepVectorStore.from_documents(
    docs,
    collection_name=collection_name,
    config=config,
    api_url=ZEP_API_URL,
    api_key=ZEP_API_KEY,
    embedding=None,  # 我们将让 Zep 使用其低延迟嵌入器嵌入我们的文档
)
```


```python
# 等待集合嵌入完成


async def wait_for_ready(collection_name: str) -> None:
    import time

    from zep_python import ZepClient

    client = ZepClient(ZEP_API_URL, ZEP_API_KEY)

    while True:
        c = await client.document.aget_collection(collection_name)
        print(
            "嵌入状态: "
            f"{c.document_embedded_count}/{c.document_count} 个文档已嵌入"
        )
        time.sleep(1)
        if c.status == "ready":
            break


await wait_for_ready(collection_name)
```
```output
嵌入状态: 0/401 个文档已嵌入
嵌入状态: 0/401 个文档已嵌入
嵌入状态: 0/401 个文档已嵌入
嵌入状态: 0/401 个文档已嵌入
嵌入状态: 0/401 个文档已嵌入
嵌入状态: 0/401 个文档已嵌入
嵌入状态: 401/401 个文档已嵌入
```

## 相似性搜索查询

```python
# query it
query = "what is the structure of our solar system?"
docs_scores = await vs.asimilarity_search_with_relevance_scores(query, k=3)

# print results
for d, s in docs_scores:
    print(d.page_content, " -> ", s, "\n====\n")
```
```output
两个主要行星的位置（这些对于导航员来说是最必要的），木星和土星，每个需要不少于一百十六个表格。然而，不仅需要预测这些天体的位置，而且同样有必要列出木星四个卫星的运动，以预测它们进入木星阴影的确切时间，以及它们的阴影与木星盘面交叉的时间，以及它们被遮挡的时间  ->  0.9003241539387915 
====

为了导航（广义上讲）提供的帮助，远不止于此，借助于更高效、更迅速和更经济的表格计算和打印，它可以提供更多的支持。

确定行星位置所需的表格与太阳、月亮和星星的表格同样重要。当我们说明这些表格的数量和复杂性时，可以形成一些概念  ->  0.8911165633479508 
====

因此应用的符号方案，立即提示了作为表达动物系统的结构、操作和循环的工具所必须具备的优势；我们对其适用性没有任何怀疑。不仅是人类和动物身体的固体部分的机械连接，还有包括肌肉、皮肤、膜等在内的软组织的结构和操作，以及性质、运动  ->  0.8899750214770481 
====
```

## 按照 MMR 重新排序的集合搜索

Zep 提供原生的硬件加速 MMR 重新排序搜索结果。

```python
query = "what is the structure of our solar system?"
docs = await vs.asearch(query, search_type="mmr", k=3)

for d in docs:
    print(d.page_content, "\n====\n")
```
```output
the positions of the two principal planets, (and these the most
necessary for the navigator,) Jupiter and Saturn, require each not less
than one hundred and sixteen tables. Yet it is not only necessary to
predict the position of these bodies, but it is likewise expedient to
tabulate the motions of the four satellites of Jupiter, to predict the
exact times at which they enter his shadow, and at which their shadows
cross his disc, as well as the times at which they are interposed 
====

the scheme of notation thus applied, immediately suggested the
advantages which must attend it as an instrument for expressing the
structure, operation, and circulation of the animal system; and we
entertain no doubt of its adequacy for that purpose. Not only the
mechanical connexion of the solid members of the bodies of men and
animals, but likewise the structure and operation of the softer parts,
including the muscles, integuments, membranes, &c. the nature, motion, 
====

resistance, economizing time, harmonizing the mechanism, and giving to
the whole mechanical action the utmost practical perfection.

The system of mechanical contrivances by which the results, here
attempted to be described, are attained, form only one order of
expedients adopted in this machinery;--although such is the perfection
of their action, that in any ordinary case they would be regarded as
having attained the ends in view with an almost superfluous degree of 
====
```

# 通过元数据过滤

使用元数据过滤器来缩小结果范围。首先，加载另一本书：“福尔摩斯的冒险”

```python
# Let's add more content to the existing Collection
article_url = "https://www.gutenberg.org/files/48320/48320-0.txt"
loader = WebBaseLoader(article_url)
documents = loader.load()

# split it into chunks
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

await vs.aadd_documents(docs)

await wait_for_ready(collection_name)
```
```output
Embedding status: 401/1691 documents embedded
Embedding status: 401/1691 documents embedded
Embedding status: 401/1691 documents embedded
Embedding status: 401/1691 documents embedded
Embedding status: 401/1691 documents embedded
Embedding status: 401/1691 documents embedded
Embedding status: 901/1691 documents embedded
Embedding status: 901/1691 documents embedded
Embedding status: 901/1691 documents embedded
Embedding status: 901/1691 documents embedded
Embedding status: 901/1691 documents embedded
Embedding status: 901/1691 documents embedded
Embedding status: 1401/1691 documents embedded
Embedding status: 1401/1691 documents embedded
Embedding status: 1401/1691 documents embedded
Embedding status: 1401/1691 documents embedded
Embedding status: 1691/1691 documents embedded
```
我们看到来自两本书的结果。注意 `source` 元数据

```python
query = "Was he interested in astronomy?"
docs = await vs.asearch(query, search_type="similarity", k=3)

for d in docs:
    print(d.page_content, " -> ", d.metadata, "\n====\n")
```
```output
or remotely, for this purpose. But in addition to these, a great number
of tables, exclusively astronomical, are likewise indispensable. The
predictions of the astronomer, with respect to the positions and motions
of the bodies of the firmament, are the means, and the only means, which
enable the mariner to prosecute his art. By these he is enabled to
discover the distance of his ship from the Line, and the extent of his  ->  {'source': 'https://www.gutenberg.org/cache/epub/71292/pg71292.txt'} 
====

possess all knowledge which is likely to be useful to him in his work,
and this I have endeavored in my case to do. If I remember rightly, you
on one occasion, in the early days of our friendship, defined my limits
in a very precise fashion.”

“Yes,” I answered, laughing. “It was a singular document. Philosophy,
astronomy, and politics were marked at zero, I remember. Botany
variable, geology profound as regards the mud-stains from any region  ->  {'source': 'https://www.gutenberg.org/files/48320/48320-0.txt'} 
====

of astronomy, and its kindred sciences, with the various arts dependent
on them. In none are computations more operose than those which
astronomy in particular requires;--in none are preparatory facilities
more needful;--in none is error more detrimental. The practical
astronomer is interrupted in his pursuit, and diverted from his task of
observation by the irksome labours of computation, or his diligence in
observing becomes ineffectual for want of yet greater industry of  ->  {'source': 'https://www.gutenberg.org/cache/epub/71292/pg71292.txt'} 
====
```
现在，我们设置一个过滤器

```python
filter = {
    "where": {
        "jsonpath": (
            "$[*] ? (@.source == 'https://www.gutenberg.org/files/48320/48320-0.txt')"
        )
    },
}

docs = await vs.asearch(query, search_type="similarity", metadata=filter, k=3)

for d in docs:
    print(d.page_content, " -> ", d.metadata, "\n====\n")
```
```output
possess all knowledge which is likely to be useful to him in his work,
and this I have endeavored in my case to do. If I remember rightly, you
on one occasion, in the early days of our friendship, defined my limits
in a very precise fashion.”

“Yes,” I answered, laughing. “It was a singular document. Philosophy,
astronomy, and politics were marked at zero, I remember. Botany
variable, geology profound as regards the mud-stains from any region  ->  {'source': 'https://www.gutenberg.org/files/48320/48320-0.txt'} 
====

the light shining upon his strong-set aquiline features. So he sat as I
dropped off to sleep, and so he sat when a sudden ejaculation caused me
to wake up, and I found the summer sun shining into the apartment. The
pipe was still between his lips, the smoke still curled upward, and the
room was full of a dense tobacco haze, but nothing remained of the heap
of shag which I had seen upon the previous night.

“Awake, Watson?” he asked.

“Yes.”

“Game for a morning drive?”  ->  {'source': 'https://www.gutenberg.org/files/48320/48320-0.txt'} 
====

“I glanced at the books upon the table, and in spite of my ignorance
of German I could see that two of them were treatises on science, the
others being volumes of poetry. Then I walked across to the window,
hoping that I might catch some glimpse of the country-side, but an oak
shutter, heavily barred, was folded across it. It was a wonderfully
silent house. There was an old clock ticking loudly somewhere in the
passage, but otherwise everything was deadly still. A vague feeling of  ->  {'source': 'https://www.gutenberg.org/files/48320/48320-0.txt'} 
====
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
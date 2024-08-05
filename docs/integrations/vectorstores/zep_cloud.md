---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/zep_cloud.ipynb
---

# Zep Cloud
> 回忆、理解并提取聊天记录中的数据。为个性化的AI体验提供动力。

> [Zep](https://www.getzep.com) 是一个用于AI助手应用的长期记忆服务。
> 借助Zep，您可以让AI助手具备回忆过去对话的能力，无论时间多么久远，
> 同时减少幻觉、延迟和成本。

> 请参阅 [Zep Cloud安装指南](https://help.getzep.com/sdks)

## 使用方法

在下面的示例中，我们使用 Zep 的自动嵌入功能，该功能使用低延迟嵌入模型自动在 Zep 服务器上嵌入文档。

## 注意
- 这些示例使用 Zep 的异步接口。通过去掉方法名称中的 `a` 前缀来调用同步接口。

## 从文档加载或创建集合


```python
from uuid import uuid4

from langchain_community.document_loaders import WebBaseLoader
from langchain_community.vectorstores import ZepCloudVectorStore
from langchain_text_splitters import RecursiveCharacterTextSplitter

ZEP_API_KEY = "<your zep project key>"  # 您可以从 Zep 仪表板生成您的 zep 项目密钥
collection_name = f"babbage{uuid4().hex}"  # 唯一的集合名称，仅限字母和数字

# 加载文档
article_url = "https://www.gutenberg.org/cache/epub/71292/pg71292.txt"
loader = WebBaseLoader(article_url)
documents = loader.load()

# 将其拆分为块
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

# 实例化 VectorStore。由于集合尚不存在于 Zep 中，
# 它将被创建并填充我们传入的文档。
vs = ZepCloudVectorStore.from_documents(
    docs,
    embedding=None,
    collection_name=collection_name,
    api_key=ZEP_API_KEY,
)
```


```python
# 等待集合嵌入完成


async def wait_for_ready(collection_name: str) -> None:
    import time

    from zep_cloud.client import AsyncZep

    client = AsyncZep(api_key=ZEP_API_KEY)

    while True:
        c = await client.document.get_collection(collection_name)
        print(
            "嵌入状态: "
            f"{c.document_embedded_count}/{c.document_count} 个文档已嵌入"
        )
        time.sleep(1)
        if c.document_embedded_count == c.document_count:
            break


await wait_for_ready(collection_name)
```
```output
嵌入状态: 401/401 个文档已嵌入
```

## 集合上的相似性搜索查询


```python
# query it
query = "what is the structure of our solar system?"
docs_scores = await vs.asimilarity_search_with_relevance_scores(query, k=3)

# print results
for d, s in docs_scores:
    print(d.page_content, " -> ", s, "\n====\n")
```
```output
两个主要行星的位置（这些对导航员来说是最必要的），木星和土星，每个都需要不少于一百一十六个表格。然而，不仅需要预测这些天体的位置，而且同样有必要将木星的四个卫星的运动进行表格化，以预测它们进入木星阴影的确切时间，以及它们的阴影与木星的盘面交叉的时间，以及它们被遮挡的时间  ->  0.78691166639328 
====

被简化为一个齿轮系统。然而，我们仍然希望能够向那些不擅长数学的读者传达一些关于这个主题的令人满意的概念。

_第三_，解释当前机械的实际状态；完成的进展如何；以及造成这些进展延迟的可能原因，这必然是所有科学朋友们感到遗憾的主题。我们将指出什么  ->  0.7853284478187561 
====

由于天文学的改进状态，他发现有必要在1821年重新计算这些表格。

尽管自从发现四颗新行星——谷神星、帕拉斯、朱诺和维斯塔，已经大约三十年，但直到最近才出版了它们的运动表。它们最近出现在恩克的历书中。

因此，我们试图传达一些概念（尽管必然是非常不充分的）关于所需的庞大数值表的范围  ->  0.7840130925178528 
====
```

## 基于MMR的重新排名搜索

Zep提供本地硬件加速的MMR重新排名搜索结果。

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

are reduced to a system of wheel-work. We are, nevertheless, not without
hopes of conveying, even to readers unskilled in mathematics, some
satisfactory notions of a general nature on this subject.

_Thirdly_, To explain the actual state of the machinery at the present
time; what progress has been made towards its completion; and what are
the probable causes of those delays in its progress, which must be a
subject of regret to all friends of science. We shall indicate what 
====

general commerce. But the science in which, above all others, the most
extensive and accurate tables are indispensable, is Astronomy; with the
improvement and perfection of which is inseparably connected that of the
kindred art of Navigation. We scarcely dare hope to convey to the
general reader any thing approaching to an adequate notion of the
multiplicity and complexity of the tables necessary for the purposes of
the astronomer and navigator. We feel, nevertheless, that the truly 
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

我们可以看到来自两本书的结果。注意 `source` 元数据

```python
query = "Was he interested in astronomy?"
docs = await vs.asearch(query, search_type="similarity", k=3)

for d in docs:
    print(d.page_content, " -> ", d.metadata, "\n====\n")
```
```output
of astronomy, and its kindred sciences, with the various arts dependent
on them. In none are computations more operose than those which
astronomy in particular requires;--in none are preparatory facilities
more needful;--in none is error more detrimental. The practical
astronomer is interrupted in his pursuit, and diverted from his task of
observation by the irksome labours of computation, or his diligence in
observing becomes ineffectual for want of yet greater industry of  ->  {'source': 'https://www.gutenberg.org/cache/epub/71292/pg71292.txt'} 
====

possess all knowledge which is likely to be useful to him in his work,
and this I have endeavored in my case to do. If I remember rightly, you
on one occasion, in the early days of our friendship, defined my limits
in a very precise fashion.”

“Yes,” I answered, laughing. “It was a singular document. Philosophy,
astronomy, and politics were marked at zero, I remember. Botany
variable, geology profound as regards the mud-stains from any region  ->  {'source': 'https://www.gutenberg.org/files/48320/48320-0.txt'} 
====

easily admitted, that an assembly of eminent naturalists and physicians,
with a sprinkling of astronomers, and one or two abstract
mathematicians, were not precisely the persons best qualified to
appreciate such an instrument of mechanical investigation as we have
here described. We shall not therefore be understood as intending the
slightest disrespect for these distinguished persons, when we express
our regret, that a discovery of such paramount practical value, in a  ->  {'source': 'https://www.gutenberg.org/cache/epub/71292/pg71292.txt'} 
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

the evening than in the daylight, for he said that he hated to be
conspicuous. Very retiring and gentlemanly he was. Even his voice was
gentle. He’d had the quinsy and swollen glands when he was young, he
told me, and it had left him with a weak throat, and a hesitating,
whispering fashion of speech. He was always well dressed, very neat and
plain, but his eyes were weak, just as mine are, and he wore tinted
glasses against the glare.”  ->  {'source': 'https://www.gutenberg.org/files/48320/48320-0.txt'} 
====

which was characteristic of him. “It is perhaps less suggestive than
it might have been,” he remarked, “and yet there are a few inferences
which are very distinct, and a few others which represent at least a
strong balance of probability. That the man was highly intellectual
is of course obvious upon the face of it, and also that he was fairly
well-to-do within the last three years, although he has now fallen upon
evil days. He had foresight, but has less now than formerly, pointing  ->  {'source': 'https://www.gutenberg.org/files/48320/48320-0.txt'} 
====
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
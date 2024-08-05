---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/lancedb.ipynb
---

# LanceDB

>[LanceDB](https://lancedb.com/) 是一个开源的向量搜索数据库，具有持久存储功能，极大简化了嵌入的检索、过滤和管理。完全开源。

本笔记本展示了如何使用与 `LanceDB` 向量数据库相关的功能，基于Lance数据格式。

```python
! pip install tantivy
```

```python
! pip install -U langchain-openai langchain-community
```

```python
! pip install lancedb
```

我们想使用 OpenAIEmbeddings，因此我们必须获取 OpenAI API 密钥。

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
! rm -rf /tmp/lancedb
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import LanceDB
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()

documents = CharacterTextSplitter().split_documents(documents)
embeddings = OpenAIEmbeddings()
```

##### 对于 LanceDB 云，您可以按如下方式调用向量存储：

```python
db_url = "db://lang_test" # 您创建的数据库的 URL
api_key = "xxxxx" # 您的 API 密钥
region="us-east-1-dev"  # 您选择的区域

vector_store = LanceDB(
    uri=db_url,
    api_key=api_key,
    region=region,
    embedding=embeddings,
    table_name='langchain_test'
    )
```

您还可以将 `region`、`api_key`、`uri` 添加到 `from_documents()` 类方法中。

```python
from lancedb.rerankers import LinearCombinationReranker

reranker = LinearCombinationReranker(weight=0.3)

docsearch = LanceDB.from_documents(documents, embeddings, reranker=reranker)
query = "总统对 Ketanji Brown Jackson 的评价是什么"
```

```python
docs = docsearch.similarity_search_with_relevance_scores(query)
print("相关性得分 - ", docs[0][1])
print("文本 - ", docs[0][0].page_content[:1000])
```
```output
相关性得分 -  0.7066475030191711
文本 -  他们在回应一通 9-1-1 的电话时，一名男子用一把偷来的枪射杀了他们。

摩拉警官27岁。

里维拉警官22岁。

他们都是多米尼加裔美国人，曾在后来选择作为警察巡逻的同一条街道上长大。

我与他们的家人交谈，告诉他们我们永远感激他们的牺牲，我们将继续他们的使命，以恢复每个社区应有的信任和安全。

我在这些问题上工作了很长时间。

我知道什么有效：投资于犯罪预防和社区警察，他们会在街道上巡逻，了解社区，并能恢复信任和安全。

所以我们不要抛弃我们的街道。或者在安全与平等正义之间做选择。

让我们团结起来，保护我们的社区，恢复信任，并让执法部门承担责任。

这就是为什么司法部要求佩戴执法记录仪，禁止窒息手段，并限制无敲门令的原因。

这就是为什么美国救助计划提供了 3500 亿美元，供城市、州和县用于雇佣更多警察并投资于经过验证的策略，如社区暴力干预——由值得信赖的信使打破暴力和创伤的循环，并给予年轻人希望。

我们应该都同意：答案不是削减警察经费。答案是为警察提供保护我们社区所需的资源和培训。

我请求民主党和共和党都支持：通过我的预算，确保我们的邻里安全。

我将继续尽我所能打击枪支走私和可以在线购买并在家中制造的幽灵枪——它们没有序列号，无法追踪。

我请求国会通过经过验证的措施以减少枪支暴力。通过普遍背景调查。为什么恐怖分子名单上的任何人都可以购买武器？

禁止突击步枪和高容量弹匣。

废除使枪支制造商成为美国唯一不能被起诉的行业的责任保护。

这些法律并不侵犯第二修正案。它们拯救生命。

在美国最基本的权利是投票权——并被计算在内。它正受到攻击。

在一个又一个州，新法律的通过，不仅压制投票，还颠覆整个选举。

我们不能让这种情况发生。

今晚。我呼吁参议院：通过《投票自由法》。通过《约翰·刘易斯投票权法》。同时通过《披露法》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为国家服务一生的人：大法官斯蒂芬·布雷耶——一位退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶法官，感谢您的服务。

总统最严肃的宪法责任之一就是提名某人担任美国最高法院法官。

我在四天前做到了这一点，当时我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律人才之一，将继续布雷耶法官的卓越遗产。

曾是私人执业中的顶级诉讼律师。曾是联邦公设辩护人。来自公立学校教育工作者和警察的家庭。一个共识建立者。自从她被提名以来，得到了广泛的支持——从警察兄弟会到由民主党和共和党任命的前法官。

如果我们要推进自由与正义，就需要确保边界并修复移民系统。

我们可以同时做到这一点。在我们的边界，我们安装了新技术，如尖端扫描仪，以更好地检测毒品走私。

我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人贩子。

我们正在设立专门的移民法官，以便逃离迫害和暴力的家庭可以更快地听到他们的案件。
```

## 添加图像 


```python
! pip install -U langchain-experimental
```


```python
! pip install open_clip_torch torch
```


```python
! rm -rf '/tmp/multimmodal_lance'
```


```python
from langchain_experimental.open_clip import OpenCLIPEmbeddings
```


```python
import os

import requests

# 下载的图像URL列表
image_urls = [
    "https://github.com/raghavdixit99/assets/assets/34462078/abf47cc4-d979-4aaa-83be-53a2115bf318",
    "https://github.com/raghavdixit99/assets/assets/34462078/93be928e-522b-4e37-889d-d4efd54b2112",
]

texts = ["鸟", "龙"]

# 保存图像的目录
dir_name = "./photos/"

# 如果目录不存在则创建
os.makedirs(dir_name, exist_ok=True)

image_uris = []
# 下载并保存每张图像
for i, url in enumerate(image_urls, start=1):
    response = requests.get(url)
    path = os.path.join(dir_name, f"image{i}.jpg")
    image_uris.append(path)
    with open(path, "wb") as f:
        f.write(response.content)
```


```python
from langchain_community.vectorstores import LanceDB

vec_store = LanceDB(
    table_name="multimodal_test",
    embedding=OpenCLIPEmbeddings(),
)
```


```python
vec_store.add_images(uris=image_uris)
```



```output
['b673620b-01f0-42ca-a92e-d033bb92c0a6',
 '99c3a5b0-b577-417a-8177-92f4a655dbfb']
```



```python
vec_store.add_texts(texts)
```



```output
['f7adde5d-a4a3-402b-9e73-088b230722c3',
 'cbed59da-0aec-4bff-8820-9e59d81a2140']
```



```python
img_embed = vec_store._embedding.embed_query("bird")
```


```python
vec_store.similarity_search_by_vector(img_embed)[0]
```



```output
Document(page_content='鸟', metadata={'id': 'f7adde5d-a4a3-402b-9e73-088b230722c3'})
```



```python
vec_store._table
```



```output
LanceTable(connection=LanceDBConnection(/tmp/lancedb), name="multimodal_test")
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)
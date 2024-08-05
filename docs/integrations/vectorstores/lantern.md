---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/lantern.ipynb
---

# Lantern

>[Lantern](https://github.com/lanterndata/lantern) 是一个开源的 `Postgres` 向量相似性搜索工具

它支持：
- 精确和近似最近邻搜索
- L2 平方距离、汉明距离和余弦距离

您需要通过 `pip install -qU langchain-community` 安装 `langchain-community` 才能使用此集成

本笔记本展示了如何使用 `Postgres` 向量数据库（`Lantern`）。

请参阅 [安装说明](https://github.com/lanterndata/lantern#-quick-install)。

我们想使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

# 安装必要的包
!pip install openai
!pip install psycopg2-binary
!pip install tiktoken


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```
```output
OpenAI API Key: ········
```

```python
## 加载环境变量
from typing import List, Tuple

from dotenv import load_dotenv

load_dotenv()
```



```output
False
```



```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Lantern
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```


```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```


```python
# Lantern 需要数据库的连接字符串。
# 示例 postgresql://postgres:postgres@localhost:5432/postgres
CONNECTION_STRING = getpass.getpass("DB Connection String:")

# # 或者，您可以从环境变量中创建它。
# import os

# CONNECTION_STRING = Lantern.connection_string_from_db_params(
#     driver=os.environ.get("LANTERN_DRIVER", "psycopg2"),
#     host=os.environ.get("LANTERN_HOST", "localhost"),
#     port=int(os.environ.get("LANTERN_PORT", "5432")),
#     database=os.environ.get("LANTERN_DATABASE", "postgres"),
#     user=os.environ.get("LANTERN_USER", "postgres"),
#     password=os.environ.get("LANTERN_PASSWORD", "postgres"),
# )

# 或者您可以通过 `LANTERN_CONNECTION_STRING` 环境变量传递它
```
```output
DB Connection String: ········
```

## 使用余弦距离进行相似性搜索（默认）

```python
# Lantern模块将尝试创建一个与集合名称相同的表。
# 因此，请确保集合名称是唯一的，并且用户有权限创建表。

COLLECTION_NAME = "state_of_the_union_test"

db = Lantern.from_documents(
    embedding=embeddings,
    documents=docs,
    collection_name=COLLECTION_NAME,
    connection_string=CONNECTION_STRING,
    pre_delete_collection=True,
)
```

```python
query = "总统对Ketanji Brown Jackson的评价是什么"
docs_with_score = db.similarity_search_with_score(query)
```

```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```
```output
--------------------------------------------------------------------------------
Score:  0.18440479
今晚。我呼吁参议院：通过《投票自由法》。通过《约翰·刘易斯投票权法》。同时，通过《披露法》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为国家奉献一生的人：史蒂芬·布雷耶大法官——一位退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最重要的宪法责任之一就是提名某人担任美国最高法院大法官。

四天前，我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官的卓越遗产。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.21727282
一位曾经的顶级私人执业律师。一位前联邦公设辩护人。而且来自一个公共学校教育者和警察的家庭。一个共识建立者。自从她被提名以来，她获得了广泛的支持——从警察兄弟会到由民主党和共和党任命的前法官。

如果我们要推进自由和正义，就需要确保边界安全并修复移民系统。

我们可以同时做到这两点。在我们的边界，我们安装了新技术，比如尖端扫描仪，以更好地检测毒品走私。

我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人贩子。

我们正在设立专门的移民法官，以便逃离迫害和暴力的家庭能够更快地得到审理。

我们正在确保承诺并支持南美和中美洲的合作伙伴，接纳更多的难民并确保他们自己的边界安全。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.22621095
对于我们的LGBTQ+美国人，让我们终于将两党《平等法案》送到我桌上。针对跨性别美国人及其家庭的州法律攻击是错误的。

正如我去年所说，尤其是对我们年轻的跨性别美国人，我将永远支持你们，作为你们的总统，让你们能够做自己，发挥上帝赋予的潜力。

虽然我们似乎总是意见不合，但这并不是真的。我去年签署了80项两党法案。从防止政府停摆到保护亚裔美国人免受仍然过于普遍的仇恨犯罪，再到改革军事司法。

不久之后，我们将加强我三十年前首次起草的《反对对女性暴力法》。我们必须向全国展示，我们能够团结一致，做出重大成就。

因此，今晚我提出国家团结议程。我们可以一起做的四件大事。

首先，打击阿片类药物流行病。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.22654456
今晚，我宣布对这些向美国企业和消费者收取过高费用的公司进行打击。

随着华尔街公司接管越来越多的养老院，这些养老院的质量下降，费用上升。

这在我任期内将结束。

医疗保险将为养老院设定更高的标准，确保你的亲人得到他们应得和期待的照顾。

我们还将降低成本，保持经济强劲，通过让工人获得公平的机会，提供更多的培训和学徒，依据技能而非学位来雇用他们。

让我们通过《工资公平法案》和带薪休假。

将最低工资提高到每小时15美元，并延长儿童税收抵免，以便没有人需要在贫困中养家糊口。

让我们增加佩尔助学金，并增加我们对历史性黑人大学的支持，投资于我们第一夫人——全职教师Jill所称的美国最被低估的秘密：社区大学。
--------------------------------------------------------------------------------
```

## 最大边际相关性搜索 (MMR)
最大边际相关性优化查询的相似性与所选文档之间的多样性。

```python
docs_with_score = db.max_marginal_relevance_search_with_score(query)
```

```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```
```output
--------------------------------------------------------------------------------
Score:  0.18440479
今晚，我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯投票权法案》。同时，通过《披露法案》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：大法官斯蒂芬·布雷耶——一位退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶法官，感谢您的服务。

总统最重要的宪法责任之一就是提名人选担任美国最高法院法官。

我在四天前做了这件事，当时我提名了巡回上诉法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律人才，将继续布雷耶法官的卓越遗产。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.23515457
我们无法改变我们之间的分歧。但我们可以改变如何向前迈进——在COVID-19和其他我们必须共同面对的问题上。

我最近在警员威尔伯特·莫拉和他的搭档杰森·里维拉的葬礼后几天访问了纽约市警察局。

他们是在响应911电话时被一名用偷来的枪射杀的。

莫拉警员27岁。

里维拉警员22岁。

两位多米尼加裔美国人，他们在后来选择作为警察巡逻的街道上长大。

我与他们的家人交谈，并告诉他们我们永远欠下他们的牺牲，我们将继续他们恢复每个社区应有的信任和安全的使命。

我在这些问题上工作了很长时间。

我知道什么有效：投资于犯罪预防和社区警察，他们将走上街头，了解邻里，并能够恢复信任和安全。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.24478757
其中一位驻扎在基地，吸入了来自“焚烧坑”的有毒烟雾，这些焚烧坑焚烧战争的废物——医疗和危险材料、喷气燃料等。

当他们回家时，许多世界上最强壮和训练有素的战士再也不是原来的样子。

头痛。麻木。头晕。

一种会让他们躺在裹着国旗的棺材里的癌症。

我知道。

其中一位士兵是我的儿子博·拜登少校。

我们不确定焚烧坑是否是他脑癌的原因，或者是我们许多部队的疾病。

但我致力于找出我们能知道的一切。

致力于像来自俄亥俄州的丹妮尔·罗宾逊这样的军事家庭。

一等军士希斯·罗宾逊的遗孀。

他生来就是一名士兵。美国国民警卫队。科索沃和伊拉克的战斗医护兵。

驻扎在巴格达附近，距离足球场大小的焚烧坑仅几码远。

希斯的遗孀丹妮尔今晚与我们同在。他们喜欢去俄亥俄州立大学的足球比赛。他喜欢和他们的女儿一起搭建乐高。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.25137997
我正在采取强有力的行动，以确保我们的制裁带来的痛苦针对俄罗斯的经济。我将利用我们所有的工具来保护美国的企业和消费者。

今晚，我可以宣布美国与其他30个国家合作，从全球的储备中释放6000万桶石油。

美国将领导这一努力，从我们自己的战略石油储备中释放3000万桶。如果有必要，我们准备做更多，与我们的盟友团结一致。

这些措施将有助于缓解国内的油价。我知道关于发生的事情的消息可能令人担忧。

但我想让你知道，我们会没事的。

当这个时代的历史被书写时，普京对乌克兰的战争将使俄罗斯变得更弱，而世界其他地方则变得更强。

虽然不应该因为如此可怕的事情而让世界各地的人们看到现在的利害关系，但现在每个人都清楚地看到了这一点。
--------------------------------------------------------------------------------
```

## 使用 vectorstore

上述内容中，我们从头创建了一个 vectorstore。然而，我们经常希望使用现有的 vectorstore。为了做到这一点，我们可以直接初始化它。

```python
store = Lantern(
    collection_name=COLLECTION_NAME,
    connection_string=CONNECTION_STRING,
    embedding_function=embeddings,
)
```

### 添加文档
我们可以将文档添加到现有的 vectorstore 中。

```python
store.add_documents([Document(page_content="foo")])
```

```output
['f8164598-aa28-11ee-a037-acde48001122']
```

```python
docs_with_score = db.similarity_search_with_score("foo")
```

```python
docs_with_score[0]
```

```output
(Document(page_content='foo'), -1.1920929e-07)
```

```python
docs_with_score[1]
```

```output
(Document(page_content='And let’s pass the PRO Act when a majority of workers want to form a union—they shouldn’t be stopped.  \n\nWhen we invest in our workers, when we build the economy from the bottom up and the middle out together, we can do something we haven’t done in a long time: build a better America. \n\nFor more than two years, COVID-19 has impacted every decision in our lives and the life of the nation. \n\nAnd I know you’re tired, frustrated, and exhausted. \n\nBut I also know this. \n\nBecause of the progress we’ve made, because of your resilience and the tools we have, tonight I can say  \nwe are moving forward safely, back to more normal routines.  \n\nWe’ve reached a new moment in the fight against COVID-19, with severe cases down to a level not seen since last July.  \n\nJust a few days ago, the Centers for Disease Control and Prevention—the CDC—issued new mask guidelines. \n\nUnder these new guidelines, most Americans in most of the country can now be mask free.', metadata={'source': '../../how_to/state_of_the_union.txt'}),
 0.24038416)
```

### 覆盖一个向量存储

如果您有一个现有的集合，可以通过执行 `from_documents` 并设置 `pre_delete_collection` = True 来覆盖它。 这将在重新填充集合之前删除该集合。

```python
db = Lantern.from_documents(
    documents=docs,
    embedding=embeddings,
    collection_name=COLLECTION_NAME,
    connection_string=CONNECTION_STRING,
    pre_delete_collection=True,
)
```

```python
docs_with_score = db.similarity_search_with_score("foo")
```

```python
docs_with_score[0]
```

```output
(Document(page_content='And let’s pass the PRO Act when a majority of workers want to form a union—they shouldn’t be stopped.  \n\nWhen we invest in our workers, when we build the economy from the bottom up and the middle out together, we can do something we haven’t done in a long time: build a better America. \n\nFor more than two years, COVID-19 has impacted every decision in our lives and the life of the nation. \n\nAnd I know you’re tired, frustrated, and exhausted. \n\nBut I also know this. \n\nBecause of the progress we’ve made, because of your resilience and the tools we have, tonight I can say  \nwe are moving forward safely, back to more normal routines.  \n\nWe’ve reached a new moment in the fight against COVID-19, with severe cases down to a level not seen since last July.  \n\nJust a few days ago, the Centers for Disease Control and Prevention—the CDC—issued new mask guidelines. \n\nUnder these new guidelines, most Americans in most of the country can now be mask free.', metadata={'source': '../../how_to/state_of_the_union.txt'}),
 0.2403456)
```

### 使用 VectorStore 作为检索器


```python
retriever = store.as_retriever()
```


```python
print(retriever)
```
```output
tags=['Lantern', 'OpenAIEmbeddings'] vectorstore=<langchain_community.vectorstores.lantern.Lantern object at 0x11d02f9d0>
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)